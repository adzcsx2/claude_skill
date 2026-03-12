---
name: update-docs
description: Auto-generate Chinese technical documentation for Android projects. Analyzes structure, generates interfaces, navigation, components, notifications, and API docs. Also migrates root md files to docs/ and updates README with quick links.
---

# update-docs Skill

Android 项目文档自动生成工具。分析项目结构，生成中文技术文档，支持增量更新。

## When to Use

- Generating project documentation for Android apps
- Creating interface documentation with control analysis
- Documenting navigation flows and Activity-Fragment relationships
- Listing Android four components (Activity, Service, Receiver, Provider)
- Documenting notification channels and API endpoints
- Migrating root directory md files to docs/ for centralized management
- Updating README with categorized doc quick links
- Tracking document changes with detailed update logs

## Example Prompts

- "/update-docs"
- "Generate documentation for my Android project"
- "Update project docs with --force"
- "Only generate interface documentation"

---

## Command Parameters

| Parameter | Description |
|-----------|-------------|
| No args | Incremental update of all docs |
| `--force` | Force regenerate all docs |
| `--dry-run` | Analyze only, don't generate files |
| `interfaces` | Generate interface docs only |
| `navigation` | Generate navigation docs only |
| `components` | Generate four components docs only |
| `notifications` | Generate notification docs only |
| `api` | Generate API docs only |

---

## Document Structure

```
README.md
├── 最近更新（1条摘要 + 链接到 CHANGELOG）
└── ...

docs/
├── CHANGELOG.md              # 更新列表（每条有链接到详情）
├── PROJECT_OVERVIEW.md       # 项目概览
├── INTERFACES.md             # 界面文档
├── NAVIGATION.md             # 导航文档
├── COMPONENTS.md             # 四大组件
├── NOTIFICATIONS.md          # 通知文档
├── BUILD_VARIANTS.md         # 构建变体
├── DEPENDENCIES.md           # 依赖文档
├── API.md                    # API 文档
├── .doc-metadata.json        # 元数据
└── update-list/              # 详情目录（可重新生成）
    └── update-YYYY-MM-DD.md  # 每次更新的详细内容
```

---

## Execution Flow

### 1. Verify Project Type

Check for these files:
- `settings.gradle` or `settings.gradle.kts`
- `build.gradle` or `build.gradle.kts`
- `app/src/main/AndroidManifest.xml`

Exit if not an Android project.

### 2. Clean Old Update Files (Optional)

If `--force` is used, clean old update files:

```bash
# Remove old update-list directory (can be regenerated)
rm -rf docs/update-list/
rm -f docs/UPDATE_INDEX.md
```

### 3. Load/Create Metadata

Check `docs/.doc-metadata.json`:

```json
{
  "version": "1.3",
  "projectType": "android",
  "lastUpdate": "2026-03-12T10:30:00Z",
  "lastCommit": "abc1234def5678",
  "documents": {
    "PROJECT_OVERVIEW.md": {
      "updatedAt": "2026-03-12T10:30:00Z",
      "sourceFiles": ["build.gradle", "settings.gradle"],
      "lastCommit": "abc1234"
    }
  },
  "updateHistory": [
    {
      "date": "2026-03-12",
      "diffFile": "update-list/update-2026-03-12.md",
      "summary": "新增铸造功能文档，更新 API 接口说明",
      "documentsUpdated": ["INTERFACES.md", "API.md"]
    }
  ]
}
```

### 4. Analyze Git Changes

**Git-based Change Detection:**
1. Read `docs/.doc-metadata.json` to get `lastUpdate` date
2. Run `git log --since="{lastUpdate}" --oneline --no-merges` to get new commits
3. For each commit, get changed files
4. Map changed files to affected documents

**File to Document Mapping:**
| Source File Pattern | Affected Documents |
|---------------------|-------------------|
| `**/*Activity.kt`, `**/*Activity.java` | INTERFACES.md, NAVIGATION.md |
| `**/*Fragment.kt`, `**/*Fragment.java` | INTERFACES.md, NAVIGATION.md |
| `**/res/layout/*.xml` | INTERFACES.md |
| `**/http/*Api.kt`, `**/api/*.kt` | API.md |
| `AndroidManifest.xml` | COMPONENTS.md, NAVIGATION.md |
| `build.gradle`, `build.gradle.kts` | BUILD_VARIANTS.md, DEPENDENCIES.md |
| `**/notification/*`, `*Notification*.kt` | NOTIFICATIONS.md |

### 5. Analyze Project

#### 5.1 Analyze AndroidManifest.xml
Extract: applicationId, versionCode, versionName, four components list, permissions list

#### 5.2 Analyze build.gradle
Extract: compileSdkVersion, buildTypes, productFlavors, dependencies

#### 5.3 Analyze Activity/Fragment
Use Glob to find: `**/*Activity.java`, `**/*Activity.kt`, `**/*Fragment.java`, `**/*Fragment.kt`

#### 5.4 Analyze Layout Files
Use Glob: `**/res/layout/*.xml`

#### 5.5 Analyze Notification Config
Use Grep: `NotificationChannel`, `NotificationManager`

#### 5.6 Analyze API Interfaces
Use Grep: `@GET`, `@POST`, `@PUT`, `@DELETE`

### 6. Migrate Root MD Files to docs/

Scan root directory for markdown files (excluding README.md) and migrate to appropriate docs/ subdirectories.

### 7. Generate Documents

All docs go in `docs/` directory:

| Document | Content |
|----------|---------|
| PROJECT_OVERVIEW.md | Project overview |
| INTERFACES.md | Interface docs (control analysis, functionality) |
| NAVIGATION.md | Navigation docs (Activity-Fragment relationships) |
| COMPONENTS.md | Four components docs |
| NOTIFICATIONS.md | Notification docs |
| BUILD_VARIANTS.md | Build variants docs |
| DEPENDENCIES.md | Dependencies docs |
| API.md | API interface docs (URL and method) |
| CHANGELOG.md | **Update list with links to details** |
| update-list/*.md | **Detailed update content per update** |

---

## 8. Generate Update Detail Document (CRITICAL)

Generate a detailed update document in `docs/update-list/` for each update:

### 8.1 Filename Convention
- Format: `update-YYYY-MM-DD.md`
- If file exists for today, append number: `update-YYYY-MM-DD-2.md`

### 8.2 Document Content Structure

**MUST include actual document changes, NOT just git commits:**

```markdown
# 更新详情 - YYYY-MM-DD

## 概述

**更新时间**: YYYY-MM-DD HH:MM
**触发方式**: Git 提交分析 / --force 强制更新
**关联提交**: abc1234, def5678

## 文档变更详情

### API.md

**变更类型**: 新增接口

**变更内容**:
- 新增 `POST /mint/nft` NFT 铸造接口
  - 请求参数: `imageHash`, `walletAddress`
  - 返回: `transactionHash`, `status`
- 新增 `GET /user/wallet` 钱包地址查询接口

### INTERFACES.md

**变更类型**: 新增组件

**变更内容**:
- 新增 CastDialog 铸造确认对话框
  - 支持显示铸造进度
  - 支持失败重试
- 更新 AlbumActivity 说明
  - 新增铸造状态显示逻辑

### NAVIGATION.md

**变更类型**: 更新流程

**变更内容**:
- 新增 WalletConnect 连接流程
  - ReviewActivity → WalletConnectResponseActivity
  - 支持返回重连逻辑

## 关联的 Git 提交

| 提交 | 描述 |
|------|------|
| abc1234 | 修复作品页铸造失败重试逻辑与Toast文案 |
| def5678 | 重构完成第一版-铸造流程跑通 |
```

### 8.3 How to Collect Document Changes

**Compare old and new document content:**

1. **Before generating new docs**, read existing docs content
2. **After generating new docs**, compare with old content
3. **Extract changes**:
   - New sections/chapters added
   - Sections modified (describe what changed)
   - Sections removed

**Use git diff for tracked docs:**
```bash
# Get diff of specific doc
git diff HEAD -- docs/API.md

# Get added/removed lines
git diff HEAD --unified=0 -- docs/API.md | grep "^[+-]" | grep -v "^[+-]{3}"
```

---

## 9. Update CHANGELOG.md (Update List)

CHANGELOG.md serves as the update list with clickable links to details:

```markdown
# 文档更新日志

> 本文档记录项目文档的所有更新历史。点击查看详情。

---

## 2026-03-12 - 铸造功能文档更新

**变更概述**: 新增 NFT 铸造相关文档，更新 WalletConnect 集成说明

| 文档 | 变更类型 | 简介 |
|------|----------|------|
| API.md | 新增接口 | 新增 `/mint/nft` 铸造接口、钱包查询接口 |
| INTERFACES.md | 新增组件 | 新增 CastDialog 对话框，更新 AlbumActivity |
| NAVIGATION.md | 更新流程 | 新增 WalletConnect 连接导航流程 |

[查看详情](update-list/update-2026-03-12.md)

---

## 2026-03-09 - 首次文档生成

**变更概述**: 生成完整项目文档

| 文档 | 变更类型 | 简介 |
|------|----------|------|
| PROJECT_OVERVIEW.md | 新增 | 项目概览文档 |
| INTERFACES.md | 新增 | 界面文档 |
| ... | ... | ... |

[查看详情](update-list/update-2026-03-09.md)

---

[← 返回主文档](../README.md)
```

**CHANGELOG Update Rules:**
1. **Newest first**: Insert new updates at the TOP
2. **Summary table**: Show document, change type, and brief description
3. **Detail link**: Each update has a link to `update-list/update-YYYY-MM-DD.md`
4. **No limit**: Keep all history (old update-list files can be regenerated)

---

## 10. Update README.md

README.md shows **ONLY 1** most recent update:

```markdown
## 文档导航

> 快速访问: [文档中心](docs/) | [更新记录](docs/CHANGELOG.md)

### 最近更新

| 日期 | 描述 |
|------|------|
| YYYY-MM-DD | 新增 NFT 铸造相关文档，更新 WalletConnect 集成说明 |

> 查看全部更新: [更新记录](docs/CHANGELOG.md)

---

### 快速开始
| 文档 | 描述 |
|------|------|
| [项目概览](docs/PROJECT_OVERVIEW.md) | 项目简介、版本信息、技术栈 |
| [开发环境](docs/SETUP.md) | 环境配置与开发指南 |
...
```

**README Update Rules:**
1. **Only 1 recent update**: Show just the latest update
2. **Link to CHANGELOG**: Point to `docs/CHANGELOG.md` for full history
3. **Brief description**: Summarize the update in one sentence

---

## 11. Update Metadata

Update `docs/.doc-metadata.json` with:

1. **Update timestamps** for modified documents
2. **Update lastCommit** to current HEAD
3. **Append to updateHistory** array
4. **Update stats** section

---

## Analysis Patterns

### Activity Jump Detection
```
startActivity\(new Intent\(.*?,\s*(\w+Activity)\.class\)\)
ActivityUtil\.next\(.*?,\s*(\w+Activity)\.class\)
(\w+Activity)\.start\(
```

### Fragment Switch Detection
```
beginTransaction\(\)[\s\S]*?replace\((\w+),\s*(\w+Fragment)
viewPager\.setCurrentItem\((\d+)\)
```

### Control Detection
```
findViewById\(R\.id\.(\w+)\)
binding\.(\w+)
android:onClick="(\w+)"
```

### Notification Channel Detection
```
NotificationChannel\(["']([^"']+)["'],\s*["']([^"']+)["']
```

### API Interface Detection
```
@GET\(["']([^"']+)["']\)
@POST\(["']([^"']+)["']\)
["'](https?://[^"']+)["']
["'](\/api\/[^"']+)["']
```

---

## Control Type Mapping

| XML Tag | Type | Category |
|---------|------|----------|
| TextView | TextView | Display |
| EditText | EditText | Input |
| Button | Button | Interactive |
| ImageButton | ImageButton | Interactive |
| ImageView | ImageView | Display |
| RecyclerView | RecyclerView | Container |
| ViewPager2 | ViewPager2 | Container |
| CheckBox | CheckBox | Input |
| Switch | Switch | Input |

---

## Notes

1. All documents are written in **Chinese**
2. Time format uses ISO 8601 standard
3. **CHANGELOG.md**: Serves as update list with links to details
4. **update-list/**: Contains detailed update content (can be regenerated)
5. **README.md**: Shows only 1 most recent update
6. **Document changes**: Record actual document changes, not just git commits
7. **Old update-list files**: Can be deleted and regenerated if needed
8. Root md files are migrated to docs/ and deleted from root
9. Duplicate detection: keep more detailed version when merging
