---
name: code-note
description: Add Chinese comments to Kotlin/Java source files. Supports classes, methods, member variables, and key logic blocks. Comments are concise but comprehensive.
---

# code-note Skill

为 Android 项目的 Kotlin/Java 源文件添加中文注释。

## When to Use

- Adding comments to existing code without documentation
- Improving code readability for team collaboration
- Documenting legacy code
- Creating inline documentation for complex logic

## Example Prompts

- `/code-note AlbumActivity`
- "/code-note LoginActivity.kt"
- "给 AlbumActivity 添加注释"
- "帮我给这个文件写上注释"

---

## Command Parameters

| Parameter | Description |
|-----------|-------------|
| `文件名` | 要添加注释的文件名（支持 .kt/.java），可带或不带扩展名 |

---

## Execution Flow

### 1. Locate Target File

Use Glob to find the file:
```bash
# Kotlin file
**/*{FileName}.kt

# Java file
**/*{FileName}.java
```

If multiple matches found, list them and ask user to select.

### 2. Read File Content

Read the entire file to understand:
- Class structure and purpose
- Member variables
- Methods and their functions
- Key logic blocks

### 3. Analyze Code Structure

Identify elements that need comments:

#### Class Level
- Class purpose and responsibility
- Key features/capabilities

#### Member Variables
- Purpose of each variable
- Data type significance if non-obvious

#### Methods
- Method purpose (KDoc/JavaDoc style)
- Parameter descriptions
- Return value meaning
- Side effects if any

#### Key Logic Blocks
- Complex conditional logic
- Loops and iterations
- Callback handlers
- Data transformations
- Error handling

### 4. Add Comments

Follow these rules:

**Style Guidelines:**
- Use Chinese comments
- Keep comments concise but informative
- Use KDoc format for method documentation
- Use single-line comments for inline explanations

**Comment Types:**

```kotlin
/**
 * 类/方法说明
 * @param paramName 参数说明
 * @return 返回值说明
 */
```

```kotlin
// 单行注释说明关键逻辑
```

```kotlin
/** 成员变量说明 */
private var variable: Type
```

### 5. Apply Changes

Use Edit tool to add comments without modifying code logic.

---

## Comment Priority

| Priority | Element | Example |
|----------|---------|---------|
| High | Public methods | `fun deleteUser()` |
| High | Complex logic | Nested conditions, algorithms |
| Medium | Class members | `private val adapter` |
| Medium | Private methods | `private fun calculate()` |
| Low | Self-explanatory code | `binding.tvTitle.text = title` |

---

## Comment Style Examples

### Class Comment
```kotlin
/**
 * 相册页面
 * 展示用户拍摄的照片/视频列表，支持查看详情、设为壁纸、重新上链等功能
 */
class AlbumActivity : BaseActivity<ActivityAlbumBinding>() {
```

### Member Variable Comment
```kotlin
/** 照片数据列表 */
val mData: ArrayList<PhotoEntity> = arrayListOf()

/** 相册列表适配器 */
private val mAdapter = object : RecyclerView.Adapter<ViewHolder>() {
```

### Method Comment
```kotlin
/**
 * 重新上链发布
 * @param entity 照片实体
 * @param activity 宿主Activity
 * @param statusChangeListener 状态变化回调
 */
fun repost(entity: PhotoEntity, activity: AppCompatActivity, statusChangeListener: () -> Unit) {
```

### Logic Block Comment
```kotlin
// 根据上链状态显示不同提示
when (status) {
    // 未上链状态
    Status.UNPOST -> { ... }
    // 上链中状态
    Status.POSTING -> { ... }
    // 上链失败状态
    Status.POST_FAIL -> { ... }
}
```

---

## Notes

1. **Do not modify code logic** - Only add comments
2. **Chinese comments** - All comments in Chinese
3. **Concise but comprehensive** - Balance brevity with completeness
4. **Preserve formatting** - Maintain original code style
5. **No redundant comments** - Don't state the obvious
