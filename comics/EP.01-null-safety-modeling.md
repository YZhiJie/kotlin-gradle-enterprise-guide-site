# EP.01 空安全与数据建模

> 对应正文：[01-kotlin-basics.md](../01-kotlin-basics.md)（1.4 空安全 / 1.6 数据类与密封类）· 示例：`examples/standalone/01-null-safety.kt`、`examples/standalone/02-dataclass-sealed.kt`

## 剧情回顾

- **第 1 格**：标签君写 Java 时 `user.getName()` 又炸了 NPE——Kotlin 的解法是把可空性**写进类型系统**，编译器替你守第一道关。
- **第 2 格**：码叔亮出空安全四件套：`?.` 安全调用、`?:` Elvis 兜底、`as?` 安全转换、`!!` 非空断言（生产慎用）。
- **第 3 格**：像素酱用 `data class` 存数据、`sealed class` 建模状态——`when` 不写 `else` 也能穷尽，新增子类全项目编译报错。
- **第 4 格**：码叔结案——`val` 优先、拒绝 `!!`、Java 互操作加注解。

## 知识点拆解

### 第 1 格 · 可空性是类型的一部分

- Kotlin 区分 `String`（非空）与 `String?`（可空），赋值 `null` 给 `String` 直接编译失败；
- 这把 Java 里"运行时才炸"的 NPE 提前到了编译期，是 Kotlin 相对 Java 最大的生产力提升。

### 第 2 格 · 空安全四件套

| 操作符 | 作用 | 示例 |
| ---- | ---- | ---- |
| `?.` | 链上任意一环为 null，整体为 null | `user?.email?.length` |
| `?:` | 为 null 时取默认值 | `user?.email ?: "无"` |
| `as?` | 类型转换失败返回 null | `any as? Int` |
| `!!` | 断言非空，为 null 抛 NPE | `user!!.name`（❌ 慎用） |

- 经验法则：能用 `?.` + `?:` 就别用 `!!`；`!!` 只在你**100% 确定**非空且编译器推断不出来时用。

### 第 3 格 · data class + sealed class

```kotlin
data class User(val name: String, val email: String?)        // 自动 equals/hashCode/copy

sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failure(val message: String) : Result<Nothing>()
}

fun describe(r: Result<String>): String = when (r) {          // 编译器强制穷尽
    is Result.Success -> "成功: ${r.data}"
    is Result.Failure -> "失败: ${r.message}"
}
```

- `data class` 适合值对象（VO/DTO）；`sealed class/interface` 适合状态机、错误类型；
- 对密封类型 `when` 不写 `else` 时编译器做穷尽检查——新增子类 = 免费的安全网。

### 第 4 格 · val 优先与互操作

- ✅ 企业代码 `val` 占比应 > 80%；不可变引用让并发与重构更安全；
- ⚠️ Java 互操作时给 Java 方法加 `@NotNull/@Nullable`（JetBrains 注解），消除平台类型隐患；
- ❌ `data class` 的 equals 会比较所有主构造参数，别把 `MutableList` 放进去。

## 坑点清单

1. `!!` 绕过编译器检查，NPE 照炸不误——生产代码应尽量归零；
2. Java 返回的平台类型（`String!`）Kotlin 无法判断可空性，必须靠注解或显式声明；
3. `data class` 含可变集合字段时，`equals` 结果会随集合内容变化，可能导致 HashMap 行为异常。

## 自测 3 题（含答案）

1. `val list = mutableListOf(1, 2)` 之后能 `list.add(3)` 吗？能重新赋值 `list = mutableListOf()` 吗？——**能 add（val 只锁引用），不能重新赋值（引用不可变）**。
2. 为什么对 `sealed class` 的 `when` 建议不写 `else`？——**不写 else 时编译器做穷尽检查，新增子类会编译报错，强制处理新分支**。
3. `user?.email?.length ?: 0` 当 `user` 为 null 时返回什么？——**返回 0（?. 链短路为 null，?: 兜底取默认值）**。
