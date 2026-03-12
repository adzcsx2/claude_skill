---
name: apply-remote-sign
description: 为 Android 项目自动配置远程签名，支持 Groovy DSL (build.gradle) 和 Kotlin DSL (build.gradle.kts)。调用 AndroidAutoRemoteSignTool 工具完成配置。
---

> **中文环境要求**
>
> 本技能运行在中文环境下，请遵循以下约定：
> - 面向用户的回复、注释、提示信息必须使用中文
> - AI 内部处理过程可以使用英文
> - 所有生成的文件必须使用 UTF-8 编码
>
> ---

# Android Remote Sign Tool

为 Android 项目自动配置远程签名，支持 Groovy DSL (build.gradle) 和 Kotlin DSL (build.gradle.kts)。

## 功能概述

此 skill 调用 AndroidAutoRemoteSignTool 工具，自动完成以下配置：

1. 创建 `.env.example` 环境变量模板
2. 更新 `.gitignore` 文件
3. 更新 `gradle.properties` 添加 AGP 配置
4. 修改模块的 `build.gradle` 或 `build.gradle.kts` 集成远程签名任务
5. 复制运行时脚本到项目的 `scripts/` 目录

## 使用方法

### 基本用法

```bash
# 为当前目录的 Android 项目配置远程签名
/apply-remote-sign

# 为指定目录的 Android 项目配置远程签名
/apply-remote-sign /path/to/android/project

# 配置额外模块（如 app_d, app_link）
/apply-remote-sign /path/to/project --modules app_d,app_link
```

### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `project_path` | Android 项目根目录路径（可选，默认使用当前目录） | `/Users/xxx/MyApp` |
| `--modules` | 额外需要配置的模块名称，多个用逗号分隔 | `app_d,app_link` |

## 配置签名 Token

配置完成后，需要设置 SIGN_TOKEN（三选一）：

### 方式一：项目根目录 .env 文件（推荐）
```bash
cp .env.example .env
# 编辑 .env 文件，填入实际 Token
# SIGN_TOKEN=your_actual_token_here
```

### 方式二：上级目录 .env 文件（多项目共享）
```bash
echo "SIGN_TOKEN=your_actual_token_here" > ../.env
```

### 方式三：系统环境变量（适合 CI/CD）
```bash
export SIGN_TOKEN=your_actual_token_here
```

## 构建签名 APK

配置完成后，使用以下命令构建：

```bash
# 使用 Python 构建脚本（推荐，自动检测 JDK 11）
python scripts/build.py assembleDebug

# 或直接使用 Gradle
./gradlew assembleDebug
```

## 签名原理

- **V1 签名**：本地计算摘要（32字节）→ 远程获取签名 → 本地注入 META-INF 签名文件
- **V2 签名**：基于 V1 签名后的 APK 计算摘要 → 远程获取签名 → 本地组装 APK Signing Block

签名服务只传输摘要数据，不传输整个 APK，安全高效。

## 依赖要求

- Python 3.6+
- `requests` 库（`pip install requests`）
- JDK 11
- Android SDK

## 执行命令

当用户调用此 skill 时，执行以下 Python 脚本：

```bash
# 查找插件缓存目录中的工具路径
PLUGIN_BASE="$HOME/.claude/plugins/cache/android-dev-tools/android-dev-tools"
TOOL_PATH=$(find "$PLUGIN_BASE" -name "AndroidAutoRemoteSignTool" -type d 2>/dev/null | head -1)

if [ -z "$TOOL_PATH" ]; then
  echo "Error: AndroidAutoRemoteSignTool not found in plugin cache"
  exit 1
fi

# 基本命令
python "${TOOL_PATH}/remote_sign/apply_remote_sign.py" --project-path "{project_path}"

# 带额外模块的命令
python "${TOOL_PATH}/remote_sign/apply_remote_sign.py" --project-path "{project_path}" --modules "{modules}"
```

## 注意事项

1. 配置前请确保项目已提交到 Git，以便在需要时回滚
2. 配置会修改 `build.gradle` 或 `build.gradle.kts` 文件，请检查修改内容
3. 签名 Token 需要单独配置，不要将 Token 提交到版本控制
4. 首次使用前请确保已安装 Python requests 库
