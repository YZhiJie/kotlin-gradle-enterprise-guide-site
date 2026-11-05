# EP.05 Android 企业开发

> 对应正文：[05-android-enterprise.md](../05-android-enterprise.md)（5.2 分层与 UDF / 5.3 ViewModel / 5.5 Compose / 5.6 Hilt）· 示例：`examples/android-app/`

## 剧情回顾

- **第 1 格**：标签君的"上帝 Activity"三千行，UI/网络/数据库全揉一起——分层架构把关注点拆开，UI 只渲染状态。
- **第 2 格**：码叔画出三层 + 单向数据流：UI → Domain → Data，状态自上而下、事件自下而上，UI = f(state)。
- **第 3 格**：像素酱实证 Hilt 注入 ViewModel：`@HiltViewModel` + `@Inject` 构造器，暴露只读 `StateFlow`，`viewModelScope` 自动管协程。
- **第 4 格**：码叔结案——状态提升（state hoisting）让 Composable 可复用；模块化让大团队并行开发。

## 知识点拆解

### 第 1 格 · 分层架构

- **UI 层**：Composable 界面，只负责渲染状态、转发事件；
- **Domain 层**：UseCase 用例，纯业务逻辑，不依赖 Android 框架，可纯 JVM 单测；
- **Data 层**：Repository 协调网络（Retrofit）与本地（Room），对上层暴露稳定接口；
- 单向数据流（UDF）：状态从 ViewModel 流向 UI，事件从 UI 流向 ViewModel，数据流唯一源头。

### 第 2 格 · StateFlow 与单向数据流

```kotlin
// ViewModel：唯一状态源
private val _uiState = MutableStateFlow(UiState.Loading)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()   // 只读暴露

fun onRefresh() {
    viewModelScope.launch {
        _uiState.value = UiState.Loading
        _uiState.value = runCatching { repo.fetch() }
            .fold(UiState::Success, UiState::Error)
    }
}
```

- UI 层用 `collectAsStateWithLifecycle()` 订阅，生命周期感知、不会泄漏。

### 第 3 格 · Hilt 依赖注入

- `@HiltViewModel` + `@Inject` 构造器：Hilt 编译期生成注入代码，运行时零反射；
- `@Module` + `@Provides`/`@Binds` 声明依赖绑定；
- ✅ Hilt（编译期）适合大型团队；Koin（运行时）适合小团队/原型，启动更快但无编译期检查。

### 第 4 格 · 模块化与状态提升

- **模块化**：按 feature 或 layer 拆 Gradle 模块，编译隔离、多人并行、可单独测试；
- **状态提升**：把状态从 Composable 内部提升到调用方，Composable 变成无状态的纯函数，可复用、可预览；
- ✅ 配置 `@Module(includes = [...])` 聚合模块注入，配合 BuildVariant 做多渠道。

## 坑点清单

1. Composable 里直接 `launch` 协程会导致重组时重复执行——必须用 `LaunchedEffect` 或 `rememberCoroutineScope`；
2. 把 Activity 引用传给单例/静态字段 → Context 泄漏，LeakCanary 会报；
3. `StateFlow` 重复赋相同值也会触发收集，加 `.distinctUntilChanged()` 或用 data class 保证 equals。

## 自测 3 题（含答案）

1. 单向数据流（UDF）的核心思想？——**状态自上而下流动（ViewModel → UI），事件自下而上传递（UI → ViewModel），状态只有一个源头**。
2. Hilt 和 Koin 的主要区别？——**Hilt 是编译期注入（Dagger 封装），有编译期检查但构建慢；Koin 是运行时解析，启动快但无编译期校验**。
3. 为什么要做状态提升（state hoisting）？——**让 Composable 变成无状态纯函数，提升可复用性和可测试性，状态由调用方控制**。
