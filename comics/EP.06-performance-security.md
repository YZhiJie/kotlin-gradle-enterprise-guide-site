# EP.06 性能与安全

> 对应正文：[06-performance-security.md](../06-performance-security.md)（6.2 GC 调优 / 6.3 Android 启动 / 6.4 R8 / 6.5 密钥与证书固定）

## 剧情回顾

- **第 1 格**：像素酱的 App 启动 5 秒、流量被 Charles 抓包、密钥硬编码——性能没基线、安全没清单，上线裸奔。
- **第 2 格**：码叔讲 GC 选型：G1（≤8G 堆，~200ms 停顿）通用首选，ZGC（>16G，<1ms）大堆低延迟；R8 开启混淆 + 资源收缩，包砍半、反编译只见 `a.b.c()`。
- **第 3 格**：标签君实证证书固定——OkHttp `CertificatePinner` 只信自家证书指纹，中间人装根证书也白搭；密钥放 Android Keystore 不出安全硬件。
- **第 4 格**：码叔结案——性能有基线、安全有清单，上线前逐条对照 OWASP Mobile Top 10 打勾。

## 知识点拆解

### 第 1 格 · 性能与安全的常见坑

- **启动慢**：`Application.onCreate` 里同步初始化所有 SDK → 用 App Startup 延迟初始化 + Baseline Profile；
- **包体积大**：没开 R8、没做资源 shrink → 开启 `isMinifyEnabled` + `isShrinkResources`；
- **被抓包**：没做证书固定 → 中间人可解密 HTTPS；
- **密钥泄露**：硬编码在 `strings.xml`/代码 → 放 Keystore 或 KMS。

### 第 2 格 · GC 选型

| GC | 适用堆 | 停顿目标 | 场景 |
| ---- | ---- | ---- | ---- |
| G1 | ≤ 8G | ~200ms | 通用微服务（JDK 17 默认） |
| ZGC | > 16G | < 1ms | 大堆、低延迟（交易、实时） |
| Shenandoah | 中大堆 | < 10ms | 与 ZGC 类似 |

- ✅ 先用 G1，确实有低延迟需求再上 ZGC；ZGC 在 JDK 17 已生产可用，JDK 21 进一步优化。

### 第 3 格 · 证书固定与密钥管理

```kotlin
// OkHttp 证书固定
val certificatePinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .add("api.example.com", "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBB=")  // 备用指纹
    .build()

// Android Keystore：密钥不出安全硬件
val keyStore = KeyStore.getInstance("AndroidKeyStore").apply { load(null) }
```

- ✅ 至少配置 2 个指纹（当前 + 备用），避免证书更新时 App 全体瘫痪；
- ✅ 密钥用 Keystore（Android）或 KMS（服务端），永不硬编码。

### 第 4 格 · R8 keep 规则

- R8 混淆会重命名类/方法/字段，反射、JNI、序列化、枚举、`@Keep` 注解的类必须加 keep 规则；
- 常见 keep：`@SerializedName` 字段（Gson）、`@Keep` 注解的类、JNI 方法、Parcelable CREATOR；
- ✅ 用 `@Keep` 注解替代手写 `-keep` 规则，语义更清晰。

## 坑点清单

1. R8 过度混淆导致反射/JSON 序列化失败，必须为相关类加 keep 规则；
2. 证书固定只配一个指纹，证书更新时所有用户无法联网；
3. 密钥硬编码在代码或资源文件，反编译即可提取。

## 自测 3 题（含答案）

1. G1 和 ZGC 分别适合什么场景？——**G1 适合 ≤8G 堆、通用微服务（~200ms 停顿）；ZGC 适合 >16G 大堆、要求亚毫秒停顿的低延迟场景**。
2. 什么是证书固定（Certificate Pinning）？——**客户端只信任特定证书的指纹，即使系统安装了恶意根证书也无法进行中间人攻击**。
3. 为什么密钥不能硬编码在代码里？——**APK 可被反编译，硬编码的密钥会直接泄露；应使用 Android Keystore 或服务端 KMS 管理**。
