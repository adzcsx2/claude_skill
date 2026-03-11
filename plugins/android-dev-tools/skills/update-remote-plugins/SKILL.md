---
name: update-remote-plugins
description: Sync marketplace.json, plugin.json, and README files, then commit and push to remote. Also syncs changes to local Claude Code plugins directory.
---

# Update Remote Plugins

Sync marketplace.json with plugins directory, update both English and Chinese README files, then commit and push to remote. Finally, sync to local Claude Code plugins directory.

## When to Use

- After modifying any skill in `plugins/` directory
- To release a new version of the plugin
- To sync English and Chinese documentation
- After adding new skills or updating existing ones

## Trigger

```
/android-dev-tools:update-remote-plugins
```

---

## Workflow

### 1. Pull Latest Changes

**ALWAYS start by pulling latest changes** to avoid conflicts:

```bash
git fetch origin
git pull --rebase
```

If pull fails due to uncommitted changes:
```bash
git stash
git pull --rebase
# After resolving any conflicts, restore stash if needed
```

### 2. Scan Skills Directory

List all skills in the plugin:
```bash
ls -1 plugins/android-dev-tools/skills/
```

For each skill, read its SKILL.md to get name and description.

### 3. Detect Changes

Check if any file changed since last commit:
```bash
git diff HEAD --name-only -- plugins/
```

If changes detected, determine version bump:
- **Bug fix / minor update** → patch (+0.0.1): 1.0.0 → 1.0.1
- **New skill / feature** → minor (+0.1.0): 1.0.0 → 1.1.0
- **Breaking change** → major (+1.0.0): 1.0.0 → 2.0.0

### 4. Update Configuration Files

**plugin.json** - Update version and description in `plugins/android-dev-tools/.claude-plugin/plugin.json`

**CRITICAL: plugin.json MUST include `skills` field:**
```json
{
  "name": "android-dev-tools",
  "description": "...",
  "version": "2.4.0",
  "author": {...},
  "skills": ["./skills/"]
}
```

Without `skills` field, Claude Code will NOT load any skills from the plugin.

**marketplace.json** - Update version in `.claude-plugin/marketplace.json`

### 5. Sync README Files

**README.md (English)** - Update skills table:
```markdown
## Included Skills

| Skill | Description |
|-------|-------------|
| `skill-name` | Description from SKILL.md |
...
```

**README_CN.md (Chinese)** - Sync with English version:
- Translate any new content
- Keep structure identical
- Update skills table
- Update repository structure section

### 6. Commit and Push (Robust)

Stage and commit changes:
```bash
git add .claude-plugin/marketplace.json README.md README_CN.md plugins/android-dev-tools/
git commit -m "feat: 更新插件至 v{version} - {变更摘要}"
```

**注意：** Commit message 必须使用中文。

Push with retry logic:
```bash
# Try push, if fails due to remote changes, pull and retry
git push || {
  git pull --rebase
  # Resolve conflicts if any
  git rebase --continue  # or --abort if needed
  git push
}
```

### 7. Sync Local Plugins (CRITICAL)

**ALWAYS sync to BOTH cache AND marketplace directories** - Claude Code reads from both locations.

```bash
# Determine target paths
VERSION=$(cat plugins/android-dev-tools/.claude-plugin/plugin.json | grep '"version"' | head -1 | cut -d'"' -f4)

# Cache path (primary)
CACHE_PATH="$HOME/.claude/plugins/cache/android-dev-tools/android-dev-tools/$VERSION"

# Marketplace path (also required!)
MARKETPLACE_PATH="$HOME/.claude/plugins/marketplaces/android-dev-tools/plugins/android-dev-tools"

# === Sync to CACHE directory ===
mkdir -p "$CACHE_PATH/skills" "$CACHE_PATH/.claude-plugin"
cp -r plugins/android-dev-tools/skills/* "$CACHE_PATH/skills/"
cp plugins/android-dev-tools/.claude-plugin/plugin.json "$CACHE_PATH/.claude-plugin/"
cp README.md README_CN.md "$CACHE_PATH/"
echo "✅ Synced to cache: $CACHE_PATH"

# === Sync to MARKETPLACE directory ===
mkdir -p "$MARKETPLACE_PATH/skills" "$MARKETPLACE_PATH/.claude-plugin"
cp -r plugins/android-dev-tools/skills/* "$MARKETPLACE_PATH/skills/"
cp plugins/android-dev-tools/.claude-plugin/plugin.json "$MARKETPLACE_PATH/.claude-plugin/"

# Update marketplace.json version
MARKETPLACE_JSON="$HOME/.claude/plugins/marketplaces/android-dev-tools/.claude-plugin/marketplace.json"
# Use sed or manual edit to update version in marketplace.json
echo "✅ Synced to marketplace: $MARKETPLACE_PATH"

# Verify
echo "=== Cache skills ==="
ls -1 "$CACHE_PATH/skills/"
echo "=== Marketplace skills ==="
ls -1 "$MARKETPLACE_PATH/skills/"
```

### 8. Update installed_plugins.json (If Needed)

Verify `~/.claude/plugins/installed_plugins.json` points to correct version:
```bash
cat ~/.claude/plugins/installed_plugins.json | grep -A5 "android-dev-tools"
```

If version mismatch, update the file to point to the new version.

---

## README Sync Rules

When syncing README.md and README_CN.md:

1. **Structure must match** - Same sections in same order
2. **Skills table** - Update both English and Chinese versions
3. **New skills** - Add to both files with translated description
4. **Removed skills** - Remove from both files
5. **Version number** - Update in both files
6. **Repository structure** - Update to show all skill directories

---

## Troubleshooting

### Issue 1: Push Rejected (Remote Has New Commits)

**Symptoms:**
```
! [rejected] main -> main (fetch first)
error: failed to push some refs
```

**Solution:**
```bash
git stash  # Save any uncommitted changes
git pull --rebase
# Resolve conflicts if any
git rebase --continue
git push
git stash pop  # Restore saved changes
```

### Issue 2: Merge Conflicts During Rebase

**Symptoms:**
```
CONFLICT (content): Merge conflict in <file>
```

**Solution:**
1. Read the conflicted file
2. Keep both changes (combine HEAD and incoming)
3. Remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
4. Stage resolved files: `git add <file>`
5. Continue: `git rebase --continue`

### Issue 3: SSH Connection Failed

**Symptoms:**
```
Connection closed by <ip> port 22
fatal: Could not read from remote repository
```

**Solution:**
Simply retry the push:
```bash
git push
```

### Issue 4: New Window Missing Updated Plugin

**Cause:** Local plugins directory not synced after push

**Solution:** Run Step 7 (Sync Local Plugins) manually - sync BOTH cache AND marketplace directories.

### Issue 5: Stash Conflicts After Rebase

**Symptoms:**
```
CONFLICT (content): Merge conflict in <file>
Auto-merging <file>
```

**Solution:**
If stash conflicts are not critical, drop the stash:
```bash
git checkout --theirs <conflicted-file>
git stash drop
```

### Issue 6: Skills Not Loading - Missing `skills` Field

**Symptoms:**
- Plugin loads but skills are not available
- `/plugin-name:skill-name` command not recognized

**Root Cause:** `plugin.json` missing `"skills": ["./skills/"]` field

**Solution:**
Add `skills` field to `plugin.json`:
```json
{
  "name": "android-dev-tools",
  "version": "2.4.0",
  "skills": ["./skills/"]
}
```

Then re-sync to local directories (Step 7).

### Issue 7: Skills Not Loading - Marketplace Not Synced

**Symptoms:**
- New skill works in some windows but not others
- Cache directory has the skill, but skill still not available

**Root Cause:** Claude Code reads from both `cache` and `marketplaces` directories. If marketplace directory is outdated, skills won't load.

**Solution:**
Sync to BOTH directories:
```bash
# Sync to marketplace
MARKETPLACE_PATH="$HOME/.claude/plugins/marketplaces/android-dev-tools/plugins/android-dev-tools"
cp -r plugins/android-dev-tools/skills/* "$MARKETPLACE_PATH/skills/"
cp plugins/android-dev-tools/.claude-plugin/plugin.json "$MARKETPLACE_PATH/.claude-plugin/"
```

### Issue 8: installed_plugins.json Version Mismatch

**Symptoms:**
- New Claude Code window loads old version
- `installed_plugins.json` points to different version than expected

**Solution:**
Update `~/.claude/plugins/installed_plugins.json`:
```json
{
  "plugins": {
    "android-dev-tools@android-dev-tools": [
      {
        "installPath": "/Users/xxx/.claude/plugins/cache/android-dev-tools/android-dev-tools/2.4.0",
        "version": "2.4.0"
      }
    ]
  }
}
```

### Issue 9: Plugin Marked as Orphaned

**Symptoms:**
- `.orphaned_at` file exists in plugin directory
- Plugin not loading despite correct configuration

**Solution:**
Remove the orphaned marker:
```bash
rm -f ~/.claude/plugins/cache/android-dev-tools/android-dev-tools/2.4.0/.orphaned_at
```

---

## Known Issues Archive

### 2026-03: android-adb Skill Not Loading After Sync

**Problem:** After adding `android-adb` skill and running update-remote-plugins, new Claude Code windows could not use the skill. The skill was present in cache directory but not available.

**Root Causes (Multiple):**
1. `plugin.json` missing `skills` field - Claude Code didn't know to load skills
2. Marketplace directory not synced - Claude Code reads from both cache and marketplaces
3. `installed_plugins.json` pointed to old version (2.0.4 instead of 2.4.0)
4. `.orphaned_at` file marked the new version as inactive

**Fixes Applied:**
1. Added `"skills": ["./skills/"]` to `plugin.json`
2. Synced skills to `~/.claude/plugins/marketplaces/android-dev-tools/plugins/android-dev-tools/skills/`
3. Updated `installed_plugins.json` to version 2.4.0
4. Deleted `.orphaned_at` file
5. Cleaned up old versions (2.0.x, 2.1.x, 2.2.x, 2.3.x)

**Prevention:**
- Always include `skills` field in `plugin.json`
- Always sync to BOTH cache AND marketplace directories
- Verify `installed_plugins.json` version after sync
- Clean up old versions to avoid confusion

### 2024-03: Git Rebase Conflicts During Plugin Sync

**Problem:** When adding `android-fold-adapter` skill, remote had new commits (v2.0.4). Local push was rejected, and rebase caused conflicts in marketplace.json, plugin.json, and README files.

**Root Cause:**
- Did not pull before starting work
- Remote and local both modified same files

**Fix Applied:**
1. `git pull --rebase`
2. Manually resolve conflicts by combining changes
3. `git rebase --continue`
4. `git push`

**Prevention:** Always run `git pull --rebase` before starting plugin updates.

---

## Complete Example Execution

```bash
# 1. Pull latest (ALWAYS FIRST)
git pull --rebase

# 2. Check for changes
CHANGES=$(git diff HEAD --name-only -- plugins/)

# 3. If changes exist, update version and files
if [ -n "$CHANGES" ]; then
  # Read current version and bump
  CURRENT=$(cat plugins/android-dev-tools/.claude-plugin/plugin.json | grep '"version"' | head -1 | cut -d'"' -f4)
  # ... bump version logic ...

  # Update README files
  # Sync skills table between README.md and README_CN.md

  # ENSURE plugin.json has skills field!
  # "skills": ["./skills/"]
fi

# 4. Commit and push
git add .claude-plugin/marketplace.json README.md README_CN.md plugins/android-dev-tools/
git commit -m "feat: 更新插件至 v$NEW_VERSION"

# Push with retry
git push || {
  git pull --rebase
  git push
}

# 5. Sync to local (BOTH cache AND marketplace!)
CACHE_PATH="$HOME/.claude/plugins/cache/android-dev-tools/android-dev-tools/$NEW_VERSION"
MARKETPLACE_PATH="$HOME/.claude/plugins/marketplaces/android-dev-tools/plugins/android-dev-tools"

# Sync to cache
mkdir -p "$CACHE_PATH/skills" "$CACHE_PATH/.claude-plugin"
cp -r plugins/android-dev-tools/skills/* "$CACHE_PATH/skills/"
cp plugins/android-dev-tools/.claude-plugin/plugin.json "$CACHE_PATH/.claude-plugin/"
cp README.md README_CN.md "$CACHE_PATH/"

# Sync to marketplace
mkdir -p "$MARKETPLACE_PATH/skills" "$MARKETPLACE_PATH/.claude-plugin"
cp -r plugins/android-dev-tools/skills/* "$MARKETPLACE_PATH/skills/"
cp plugins/android-dev-tools/.claude-plugin/plugin.json "$MARKETPLACE_PATH/.claude-plugin/"

echo "✅ Synced to local plugins"
```

---

## Notes

1. **ALWAYS pull first** - Avoid conflicts by syncing with remote before starting
2. **ALWAYS sync to local** - New Claude Code windows need the updated plugin
3. **Sync BOTH cache AND marketplace** - Claude Code reads from both directories
4. **plugin.json MUST have skills field** - Without it, no skills will load
5. **Commit message 使用中文** - 提交到远程的注释必须使用中文
6. Run from the marketplace root directory
7. Ensure git is configured with push access
8. Keep README.md and README_CN.md synchronized
9. Version format: semver (major.minor.patch)
10. Local paths:
    - Cache: `~/.claude/plugins/cache/android-dev-tools/android-dev-tools/{version}/`
    - Marketplace: `~/.claude/plugins/marketplaces/android-dev-tools/plugins/android-dev-tools/`
    - Installed: `~/.claude/plugins/installed_plugins.json`
11. If push fails, pull and retry before giving up
12. Clean up old versions to avoid version confusion
