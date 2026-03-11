# Android 开发工具 - Claude Code 插件

[English](./README.md)

一站式 Android 开发工具集。安装一次，拥有全部功能。

## 包含的 Skills

| Skill | 描述 |
|-------|------|
| `gradle-build-performance` | 调试和优化 Gradle 构建性能 |
| `apply-remote-sign` | 自动配置远程 APK 签名 |
| `update-docs` | 生成中文技术文档 |
| `android-i18n` | 审计并生成 4 种语言的国际化资源 |
| `android-fold-adapter` | 诊断和修复折叠屏适配问题 |
| `code-note` | 为 Kotlin/Java 源文件添加中文注释 |
| `android-adb` | 通过 ADB 控制 Android 设备 - 点击、滑动、输入、导航 |
| `update-remote-plugins` | 同步 marketplace 并更新本地插件 |

---

## gradle-build-performance

调试和优化 Android/Gradle 构建性能。

**功能：**
- **新增：** 诊断工作流，提供分级风险方案（零风险/低风险/中等风险）
- **新增：** 常见问题检测（动态版本、版本不一致等）
- **新增：** 推荐 gradle.properties 模板
- 分析 Gradle 构建扫描
- 识别配置与执行阶段瓶颈
- 启用配置缓存、构建缓存、并行编译
- 优化 CI/CD 构建时间
- 调试 kapt/KSP 注解处理
- Groovy DSL 和 Kotlin DSL 示例

**用法：** `/android-dev-tools:gradle-build-performance`

---

## apply-remote-sign

为 Android 项目自动配置远程 APK 签名。

**功能：**
- 支持 Groovy DSL (`build.gradle`) 和 Kotlin DSL (`build.gradle.kts`)
- 创建 `.env.example` 模板
- 更新 `.gitignore` 和 `gradle.properties`
- 集成签名任务到构建脚本
- 内置 AndroidAutoRemoteSignTool 工具

**用法：**
```bash
/android-dev-tools:apply-remote-sign [项目路径] [--modules 模块1,模块2]
```

---

## update-docs

为 Android 项目自动生成中文技术文档。

**功能：**
- 分析项目结构
- 生成界面文档（控件、功能说明）
- 文档化导航流程（Activity-Fragment 关系）
- 列出四大组件（Activity、Service、Receiver、Provider）
- 文档化通知渠道和 API 接口
- 支持增量更新
- **新增：** 将根目录 md 文件迁移到 docs/ 目录
- **新增：** 更新 README 并添加分类文档快捷链接

**用法：**
```bash
/android-dev-tools:update-docs [--force] [--dry-run] [interfaces|navigation|components|notifications|api]
```

---

## code-note

为 Kotlin/Java 源文件添加中文注释。

**功能：**
- 分析代码结构（类、方法、变量）
- 添加 KDoc/JavaDoc 风格文档
- 注释关键逻辑块
- 简短但全面的注释
- 保持原有代码格式

**用法：**
```bash
/android-dev-tools:code-note 文件名
```

**示例：**
- `/android-dev-tools:code-note AlbumActivity`
- `/android-dev-tools:code-note LoginActivity.kt`

---

## update-remote-plugins

同步 marketplace.json 与插件目录，并更新 README 文件。

**功能：**
- 扫描插件目录变更
- 插件修改时自动升级版本号
- 添加/移除插件到 marketplace.json
- 同步中英文 README 文件
- 提交并推送到远程
- 同步更新到本地 Claude Code 插件目录

**用法：** `/android-dev-tools:update-remote-plugins`

---

## android-i18n

审计 Android 项目中的硬编码中文字符串并生成国际化资源。

**功能：**
- 扫描 XML 布局和 Kotlin/Java 代码中的中文字符串
- 在 `strings.xml` 中生成字符串资源
- 自动翻译为 4 种语言 (en/ru/zh/zh-rTW)
- 更新代码使用资源引用

**用法：**
```bash
/android-dev-tools:android-i18n [项目路径]
```

---

## android-fold-adapter

诊断和修复 Android 折叠屏适配问题。

**功能：**
- 诊断折叠/展开时的 Activity 重建问题
- 修复状态丢失问题（UI 可见性、数据字段）
- 解决 Fragment 引用失效（ViewPager2）
- 自动更新 skill 以记录新模式/解决方案
- 归档已知问题供将来参考

**用法：**
```bash
/android-dev-tools:android-fold-adapter "搜索页折叠后内容消失"
```

---

## android-adb

通过 ADB 命令控制 Android 设备 - 点击、滑动、输入、导航应用。

**功能：**
- 感知-动作循环：读取 UI 状态，决定操作
- 多设备支持，自动检测物理设备/模拟器
- 点击、滑动、输入、按键操作
- 启动应用、安装 APK
- 截图用于视觉调试
- 唤醒设备并关闭锁屏

**用法：**
```bash
/android-dev-tools:android-adb 打开 Chrome 并搜索天气
/android-dev-tools:android-adb 截个屏
/android-dev-tools:android-adb 打开设置并启用深色模式
```

**前置条件：**
- ADB 已安装并在 PATH 中
- Android 设备已启用 USB 调试
- 设备已授权调试

---

## 安装

```bash
# 1. 添加 marketplace
/plugin marketplace add github.com/adzcsx2/claude_skill

# 2. 安装（包含所有 skills）
/plugin install android-dev-tools@android-dev-tools
```

---

## 仓库结构

```
claude_skill/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── android-dev-tools/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── AndroidAutoRemoteSignTool/   # 内置工具
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
├── README.md                  # 英文
├── README_CN.md               # 中文
└── .gitignore
```

---

## 环境要求

- Claude Code CLI
- `apply-remote-sign` 需要：Python 3.6+、`requests` 库、JDK 11、Android SDK
- `update-docs` 需要：标准结构的 Android 项目

---

## 许可证

MIT

## 作者

[adzcsx2](https://github.com/adzcsx2)
