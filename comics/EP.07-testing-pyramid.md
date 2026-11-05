# EP.07 测试金字塔

> 对应正文：[07-testing-quality.md](../07-testing-quality.md)（7.2 测试金字塔 / 7.3 JUnit + MockK / 7.4 协程测试 / 7.7 质量门禁）

## 剧情回顾

- **第 1 格**：标签君的团队全靠 UI 测试兜底，跑一次半小时还老 flaky——这是"冰淇淋蛋筒"反模式。
- **第 2 格**：码叔画出测试金字塔 70/20/10：单元测试 70%（快、隔离）、集成/契约 20%、UI/E2E 10%（关键路径），把逻辑压在底座。
- **第 3 格**：像素酱实证 MockK + `runTest`：`coEvery` mock 挂起函数，Turbine 断言 Flow，虚拟时间让 `delay` 瞬间跳过。
- **第 4 格**：码叔结案——覆盖率 + detekt/ktlint + 质量门禁钉在 CI 上，不达标即阻断合并。

## 知识点拆解

### 第 1 格 · 冰淇淋蛋筒反模式

- 反模式：UI 测试占大头（>50%），单元测试很少；
- 后果：跑一次慢（分钟级）、flaky（环境相关）、贵（维护成本高）、bug 定位难；
- 正确做法：把业务逻辑尽量下沉到 Domain 层用单元测试覆盖，UI 层只测渲染和关键交互。

### 第 2 格 · 测试金字塔比例

| 层级 | 占比 | 速度 | 工具 |
| ---- | ---- | ---- | ---- |
| 单元测试 | ~70% | 毫秒级 | JUnit 5 + MockK + Kotest |
| 集成/契约 | ~20% | 秒级 | Testcontainers + Pact |
| UI/E2E | ~10% | 分钟级 | Espresso / Compose Test |

- 单元测试是底座：快、隔离、确定性高，应覆盖 Domain 层所有业务逻辑。

### 第 3 格 · 协程测试

```kotlin
class UserVMTest {
    private val repo = mockk<UserRepository>()
    private val vm = UserVM(repo)

    fun `load emits success`() = runTest {
        coEvery { repo.fetch() } returns User("x")    // mock 挂起函数
        vm.load()
        vm.ui.test {                                    // Turbine 断言 Flow
            assertEquals(UiState.Success, awaitItem())
        }
    }
}
```

- `runTest` 使用虚拟时间调度器，`delay(1000)` 瞬间跳过，测试不真等；
- `coEvery`/`coVerify` 专门处理挂起函数的 mock；
- Turbine 的 `test { }` 提供 `awaitItem()`/`expectNoEvents()` 等断言。

### 第 4 格 · 质量门禁

- **覆盖率**：JaCoCo/Kover 统计，建议行覆盖率 ≥ 70%、核心 Domain 层 ≥ 90%；
- **静态分析**：detekt（代码气味）+ ktlint（格式），不通过即 fail；
- **门禁**：CI 中 `./gradlew check` 聚合测试 + detekt + ktlint + 覆盖率，PR 不绿不能合。

## 坑点清单

1. 覆盖率只追数字，为了凑覆盖率写无意义测试，真正的边界条件没覆盖；
2. detekt/ktlint 配了但不进 CI，等于没有规范；
3. UI 测试不隔离数据（依赖真实后端/固定账号），换环境就挂。

## 自测 3 题（含答案）

1. 测试金字塔的理想比例？——**单元测试 ~70%、集成/契约 ~20%、UI/E2E ~10%**。
2. `runTest` 的作用？——**使用虚拟时间调度器，让协程中的 delay/挂起瞬间完成，测试不需要真实等待**。
3. 为什么要把质量门禁放在 CI 上？——**只有自动化、强制阻断的规范才是真规范；人工约定会随时间松懈**。
