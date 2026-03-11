# Android Dev Tools - Claude Code Plugin

[中文文档](./README_CN.md)

All-in-one Android development toolkit for Claude Code. Install once, get everything.

## Included Skills

| Skill | Description |
|-------|-------------|
| `gradle-build-performance` | Debug and optimize Gradle build performance |
| `apply-remote-sign` | Auto-configure remote APK signing |
| `update-docs` | Generate Chinese technical documentation |
| `android-i18n` | Audit and generate i18n resources for 4 languages |
| `android-fold-adapter` | Diagnose and fix foldable screen adaptation issues |
| `code-note` | Add Chinese comments to Kotlin/Java source files |
| `android-adb` | Control Android devices via ADB - tap, swipe, type, navigate |
| `update-remote-plugins` | Sync marketplace and update local plugins |

---

## gradle-build-performance

Debug and optimize Android/Gradle build performance.

**Features:**
- **NEW:** Diagnostic workflow with risk-level plans (Zero/Low/Medium)
- **NEW:** Common issues detection (dynamic versions, version inconsistencies)
- **NEW:** Recommended gradle.properties template
- Analyze Gradle build scans
- Identify configuration vs execution bottlenecks
- Enable configuration cache, build cache, parallel execution
- Optimize CI/CD build times
- Debug kapt/KSP annotation processing
- Groovy DSL and Kotlin DSL examples

**Usage:** `/android-dev-tools:gradle-build-performance`

---

## apply-remote-sign

Auto-configure remote APK signing for Android projects.

**Features:**
- Supports Groovy DSL (`build.gradle`) and Kotlin DSL (`build.gradle.kts`)
- Creates `.env.example` template
- Updates `.gitignore` and `gradle.properties`
- Integrates signing tasks into build scripts
- Includes AndroidAutoRemoteSignTool (built-in)

**Usage:**
```bash
/android-dev-tools:apply-remote-sign [project_path] [--modules module1,module2]
```

---

## update-docs

Auto-generate Chinese technical documentation for Android projects.

**Features:**
- Analyzes project structure
- Generates interface documentation (controls, functionality)
- Documents navigation flows (Activity-Fragment relationships)
- Lists four components (Activity, Service, Receiver, Provider)
- Documents notification channels and API endpoints
- Supports incremental updates
- **NEW:** Migrates root md files to docs/ directory
- **NEW:** Updates README with categorized doc quick links

**Usage:**
```bash
/android-dev-tools:update-docs [--force] [--dry-run] [interfaces|navigation|components|notifications|api]
```

---

## code-note

Add Chinese comments to Kotlin/Java source files.

**Features:**
- Analyze code structure (classes, methods, variables)
- Add KDoc/JavaDoc style documentation
- Comment key logic blocks
- Concise but comprehensive comments
- Preserve original code formatting

**Usage:**
```bash
/android-dev-tools:code-note 文件名
```

**Examples:**
- `/android-dev-tools:code-note AlbumActivity`
- `/android-dev-tools:code-note LoginActivity.kt`

---

## update-remote-plugins

Sync marketplace.json with plugins directory and update README files.

**Features:**
- Scan plugins directory for changes
- Auto-bump versions on plugin modifications
- Add/remove plugins from marketplace.json
- Sync English and Chinese README files
- Commit and push to remote
- Sync changes to local Claude Code plugins directory

**Usage:** `/android-dev-tools:update-remote-plugins`

---

## android-i18n

Audit Android project for hardcoded Chinese strings and generate i18n resources.

**Features:**
- Scan hardcoded strings in XML layouts and Kotlin/Java code
- Generate string resources in `strings.xml`
- Auto-translate to 4 languages (en/ru/zh/zh-rTW)
- Update code to use resource references

**Usage:**
```bash
/android-dev-tools:android-i18n [project_path]
```

---

## android-fold-adapter

Diagnose and fix Android foldable screen adaptation issues.

**Features:**
- Diagnose Activity recreation issues on fold/unfold
- Fix state loss problems (UI visibility, data fields)
- Resolve fragment reference invalidation (ViewPager2)
- Auto-update skill with new patterns/solutions
- Archive known issues for future reference

**Usage:**
```bash
/android-dev-tools:android-fold-adapter "搜索页折叠后内容消失"
```

---

## android-adb

Control Android devices via ADB commands - tap, swipe, type, navigate apps.

**Features:**
- Perception-action loop: read UI state, decide actions
- Multi-device support with automatic detection
- Tap, swipe, type, key press actions
- Launch apps, install APKs
- Take screenshots for visual debugging
- Wake device and dismiss lock screen

**Usage:**
```bash
/android-dev-tools:android-adb open Chrome and search for weather
/android-dev-tools:android-adb take a screenshot
/android-dev-tools:android-adb open Settings and enable dark mode
```

**Prerequisites:**
- ADB installed and in PATH
- Android device with USB debugging enabled
- Device authorized for debugging

---

## Installation

```bash
# 1. Add marketplace
/plugin marketplace add github.com/adzcsx2/claude_skill

# 2. Install (includes all skills)
/plugin install android-dev-tools@android-dev-tools
```

---

## Repository Structure

```
claude_skill/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── android-dev-tools/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── AndroidAutoRemoteSignTool/   # Built-in tool
│       │   └── remote_sign/
│       │       ├── apply_remote_sign.py
│       │       ├── apply_groovy_sign.py
│       │       ├── apply_kts_sign.py
│       │       └── ...
│       └── skills/
│           ├── gradle-build-performance/
│           │   └── SKILL.md
│           ├── apply-remote-sign/
│           │   └── SKILL.md
│           ├── update-docs/
│           │   └── SKILL.md
│           ├── android-i18n/
│           │   └── SKILL.md
│           ├── android-fold-adapter/
│           │   └── SKILL.md
│           ├── code-note/
│           │   └── SKILL.md
│           ├── android-adb/
│           │   ├── SKILL.md
│           │   ├── scripts/
│           │   └── references/
│           └── update-remote-plugins/
│               └── SKILL.md
├── README.md            # English
├── README_CN.md         # Chinese
└── .gitignore
```

---

## Requirements

- Claude Code CLI
- For `apply-remote-sign`: Python 3.6+, `requests` library, JDK 11, Android SDK
- For `update-docs`: Android project with standard structure

---

## License

MIT

## Author

[adzcsx2](https://github.com/adzcsx2)
