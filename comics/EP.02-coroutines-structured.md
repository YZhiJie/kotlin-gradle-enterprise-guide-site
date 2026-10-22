# EP.02 协程与结构化并发

> 对应正文：[02-kotlin-advanced.md](../02-kotlin-advanced.md)（2.2 协程基础 / 2.3 结构化并发 / 2.4 Flow）· 示例：`examples/coroutines/CoroutineExceptionDemo.kt`、`examples/coroutines/StateInDemo.kt`

## 剧情回顾

- **第 1 格**：像素酱陷入回调地狱——三层嵌套往右跑再也回不来。协程用 `suspend` 挂起不阻塞线程，用同步的写法写异步逻辑。
- **第 2 格**：码叔画出结构化并发层级：父作用域取消/异常 → 所有子协程跟着取消，`coroutineScope` 一子异常全取消，`supervisorScope` 各子独立。
- **第 3 格**：标签君对比三种构建器：`launch` 即发即忘、`async` 有返回值、`withContext` 只切线程不新建协程；耗时算 Default、IO 用 IO、UI 切 Main。
- **第 4 格**：码叔结案——禁用 `GlobalScope`，协程必须有"爸"管，异常要显式处理。

## 知识点拆解

### 第 1 格 · suspend 与异步写法

- `suspend fun` 可以被挂起（pause）等待异步结果，**不阻塞线程**；
- 挂起函数只能从协程或其他挂起函数调用——这是编译器强制的"传染性"；
- 协程是轻量级线程：成千上万个协程复用少量线程，内存开销远低于线程。

### 第 2 格 · 结构化并发

| 作用域 | 行为 | 场景 |
| ---- | ---- | ---- |
| `coroutineScope` | 一子异常 → 全部取消并抛出 | 并行请求，一损俱损 |
| `supervisorScope` | 子协程独立失败，互不牵连 | 多个独立任务，各管各的 |
| `GlobalScope` | 脱离父级，永不自动取消 | ❌ 生产禁用 |

- 核心保证：**父作用域结束时，所有子协程随之结束**，异常沿层级传播，不会泄漏。

### 第 3 格 · 构建器与调度器

```kotlin
val job = launch { /* 即发即忘，返回 Job */ }          // 不关心返回值
val deferred = async { 42 }                             // 有返回值
val result = deferred.await()                           // 挂起等待结果

withContext(Dispatchers.IO) {                           // 只切线程，不新建协程
    blockingIoCall()
}
```

- `Dispatchers.Default`：CPU 密集（计算、转换）；
- `Dispatchers.IO`：磁盘/网络 IO；
- `Dispatchers.Main`：Android UI 线程；
- ✅ 阻塞调用必须 `withContext` 切到对应 Dispatcher，否则会卡线程。

### 第 4 格 · 异常与取消

- 协程异常不处理会被 `CoroutineExceptionHandler` 捕获或直接崩溃，**不会静默吞掉**（`async` 的异常在 `await` 时抛出）；
- `job.cancel()` 是协作式取消：协程内部必须检查取消状态（`ensureActive()` / `yield()`），死循环不会自动停；
- ❌ `GlobalScope` 没有父级，取消/异常无法传播，协程泄漏无法回收。

## 坑点清单

1. `GlobalScope.launch` 脱离结构化并发，页面销毁了协程还在跑 → 内存泄漏；
2. CPU 密集任务用 `Dispatchers.IO` 会占满 IO 线程池，导致真正的 IO 排队；
3. `async` 不 `await`，异常会被吞到作用域结束才抛出，排查困难。

## 自测 3 题（含答案）

1. `launch` 和 `async` 的区别？——**launch 返回 Job（无返回值，即发即忘），async 返回 Deferred（有返回值，await 取结果）**。
2. 为什么禁用 `GlobalScope`？——**它没有父作用域，取消和异常无法传播，协程生命周期脱离管控，容易泄漏**。
3. `coroutineScope` 和 `supervisorScope` 的区别？——**coroutineScope 中一个子协程异常会取消所有兄弟；supervisorScope 中子协程独立失败，互不影响**。
