# EP.04 JVM 后端开发

> 对应正文：[04-jvm-backend.md](../04-jvm-backend.md)（4.2 Spring Boot 骨架 / 4.3 WebFlux 协程 / 4.4 Ktor / 4.7 可观测性）

## 剧情回顾

- **第 1 格**：标签君的 Spring MVC 一个慢 SQL 卡住线程，其他请求全排队——WebFlux + 协程挂起不占线程，少量线程扛高并发。
- **第 2 格**：码叔对比两条路线：Spring Boot 3 生态最全（WebFlux 支持协程端点），Ktor 3 协程原生、启动毫秒级，适合微服务/网关。
- **第 3 格**：像素酱实证构造器注入——`val` 参数天然不可变，单构造器无需 `@Autowired`，告别字段注入。
- **第 4 格**：码叔结案——生产环境必须有可观测三件套：日志（traceId）、指标（Micrometer）、链路（Sleuth/Micrometer Tracing）。

## 知识点拆解

### 第 1 格 · 阻塞式 vs 响应式

- Spring MVC（Servlet 模型）：一个请求占一个线程，慢调用会耗尽线程池；
- Spring WebFlux + 协程：`suspend fun` 挂起期间释放线程，少量线程处理海量并发；
- 数据层配合 R2DBC（响应式 JDBC）才能全链路不阻塞，JPA 仍是阻塞的。

### 第 2 格 · Spring Boot vs Ktor

| 维度 | Spring Boot 3 | Ktor 3 |
| ---- | ---- | ---- |
| 生态 | 最全，企业标配 | 轻量，按需装插件 |
| 协程 | WebFlux 支持 | 原生协程 |
| 启动 | 秒级 | 毫秒级 |
| 适合 | 大型企业后端 | 微服务、网关、BFF |

- 团队已有 Spring 经验 → 选 Spring Boot；追求轻量与启动速度 → 选 Ktor。

### 第 3 格 · 构造器注入

```kotlin
@RestController
class UserController(
    private val repo: UserRepository,   // val，不可变
    private val mapper: UserMapper
) {
    suspend fun list() = repo.findAll()
}
```

- ✅ Kotlin 推荐构造器注入：`val` 不可变、无 `@Autowired` 注解（单构造器自动注入）、易于测试；
- ⚠️ Kotlin 类默认 `final`，Spring 需要 `kotlin("plugin.spring")` 插件为 `@Component` 等自动 `open`。

### 第 4 格 · 可观测性

- **日志**：结构化日志（JSON）+ MDC 注入 `traceId`，方便 ELK/Loki 聚合；
- **指标**：Micrometer 暴露 `http.server.requests` 等，接 Prometheus + Grafana；
- **链路**：Micrometer Tracing 接 OpenTelemetry/Zipkin，跨服务追踪一次请求；
- ✅ 三者通过 `traceId` 串联，才能从"指标发现异常"一路追到"日志定位根因"。

## 坑点清单

1. WebFlux 里调用阻塞库（JPA、`Thread.sleep`）不切 `withContext(Dispatchers.IO)`，会卡事件循环；
2. 日志不打 `traceId`，微服务间排查像大海捞针；
3. 没有熔断（Resilience4j），下游慢/挂了会拖垮自己，形成雪崩。

## 自测 3 题（含答案）

1. Spring MVC 和 WebFlux 的核心区别？——**MVC 一个请求占一个线程（阻塞），WebFlux 用少量线程通过异步/协程处理海量并发**。
2. Kotlin 为什么推荐构造器注入？——**val 不可变、单构造器无需 @Autowired、易于单元测试，避免字段注入的可变与空安全问题**。
3. 可观测性三件套是什么？——**日志（含 traceId）、指标（Micrometer）、链路追踪（OpenTelemetry）**。
