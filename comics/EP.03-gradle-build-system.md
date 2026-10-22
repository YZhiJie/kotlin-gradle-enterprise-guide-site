# EP.03 Gradle 构建体系

> 对应正文：[03-gradle-build.md](../03-gradle-build.md)（3.2 Kotlin DSL / 3.4 任务与生命周期 / 3.5 Version Catalog / 3.7 convention plugin）· 示例：`examples/gradle-demo/`

## 剧情回顾

- **第 1 格**：标签君每个模块写死版本号，一升级全工程 grep 还容易冲突——Version Catalog 把版本集中到一个 TOML。
- **第 2 格**：码叔拆解构建三阶段：初始化（读 settings）→ 配置（生成任务图）→ 执行（按依赖跑任务）；任务用 `register` 惰性创建。
- **第 3 格**：像素酱实证 `libs.versions.toml`：`[versions]` 管版本号、`[libraries]` 管坐标，build 脚本里 `libs.ktor.core` 类型安全引用。
- **第 4 格**：码叔结案——把团队约定写成 convention plugin，新模块一行 apply；`implementation` 不泄漏、`api` 才泄漏。

## 知识点拆解

### 第 1 格 · 版本管理的痛点

- 多模块工程里版本号散落在各处，升级靠全局搜索，极易遗漏导致运行时版本冲突；
- Version Catalog（`gradle/libs.versions.toml`）是版本、依赖、插件的**单一事实来源**，IDE 支持自动补全与重构。

### 第 2 格 · 构建生命周期

| 阶段 | 做什么 | 关键文件 |
| ---- | ---- | ---- |
| 初始化 | 确定有哪些项目、哪些构建 | `settings.gradle.kts` |
| 配置 | 执行 build 脚本，生成任务图 | `build.gradle.kts` |
| 执行 | 按依赖顺序执行选中的任务 | Task 图 |

- ✅ 任务用 `tasks.register<T>()` 而非 `tasks.create<T>()`——前者惰性创建，只有真正需要时才配置，显著加快配置阶段；
- ⚠️ 配置阶段不要做 IO 或重计算，否则每次构建（哪怕只是 `gradle tasks`）都要付一遍代价。

### 第 3 格 · Version Catalog 用法

```toml
# gradle/libs.versions.toml
[versions]
ktor = "2.3.0"
kotlin = "2.0.21"

[libraries]
ktor-core = { module = "io.ktor:ktor-core", version.ref = "ktor" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

```kotlin
// build.gradle.kts
plugins { alias(libs.plugins.kotlin.jvm) }
dependencies { implementation(libs.ktor.core) }
```

- `version.ref` 让版本号只定义一次，升级时改 `[versions]` 即可全工程生效。

### 第 4 格 · convention plugin 与依赖配置

- `build-logic/` 子工程里写预编译脚本插件（`kotlin-jvm-convention.gradle.kts`），把 Java 版本、测试框架、编码规范等团队约定封装进去；
- 新模块只需 `plugins { id("shop.kotlin.jvm.convention") }` 一行，告别复制粘贴；
- `implementation`：依赖只在当前模块编译/运行可见，不泄漏给下游；
- `api`：依赖会出现在下游的编译类路径，仅用于公共 API 暴露的类型。

## 坑点清单

1. `api` 滥用会把内部依赖泄漏给所有下游模块，增大编译面与升级风险；
2. 配置阶段执行网络请求或读大文件，导致每次构建都慢；
3. 用 `create` 而非 `register` 创建任务，所有任务都被提前配置，配置阶段变慢。

## 自测 3 题（含答案）

1. Gradle 构建的三个阶段？——**初始化（读 settings）→ 配置（生成任务图）→ 执行（按依赖执行任务）**。
2. `implementation` 和 `api` 的区别？——**implementation 依赖不暴露给下游编译类路径；api 会暴露，仅用于公共 API 的类型**。
3. 为什么推荐 `tasks.register` 而非 `tasks.create`？——**register 是惰性的，任务只在真正需要时才创建和配置，能显著缩短配置阶段时间**。
