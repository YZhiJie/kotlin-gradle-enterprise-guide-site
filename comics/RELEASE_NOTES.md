# 发布说明 · Kotlin & Gradle 漫画剧场 v1.0.0

> **发布日期**：2026-09-27　**版本**：v1.0.0（漫画剧场首版）　**基线教程**：kotlin-gradle-enterprise-guide v1.6+
> **提交**：`b09f37a` docs(comics): 新增 Kotlin & Gradle 漫画剧场 9 话

---

## 一、本次发布概览

为 Kotlin & Gradle 企业级应用开发教程的 **9 章正文**配套创作 **9 话四格漫画**，采用统一的「霓虹墨 Neon Ink」视觉风格，把抽象的工程概念（空安全、协程、Gradle、后端、Android、性能安全、测试、CI/CD、协作）画成"困境 → 揭秘 → 实证 → 结案"四幕故事，让读者先"看见"概念，再读机制与代码。

- **新增文件**：25 个（9 SVG + 9 MD + 7 个工程化文件）
- **新增行数**：2633 行
- **校验结果**：`node comics/check-comics.mjs` → 9 话全绿，**0 错误 0 警告**；xmllint 良构性检查全部通过
- **破坏性变更**：无（纯文档新增，不影响任何代码与示例）

---

## 二、核心亮点

### 1. 图文结合，四幕叙事

每话严格遵循四格结构，符合学习认知规律：

| 格 | 叙事职能 | 内容 |
| ---- | ---- | ---- |
| 第 1 格 · 困境 | 真实开发场景抛出问题 | NPE 崩溃、回调地狱、版本冲突、线程打满、上帝 Activity…… |
| 第 2 格 · 揭秘 | 码叔拆解原理 | 空安全四件套、结构化并发、构建三阶段、分层 UDF、GC 选型…… |
| 第 3 格 · 实证 | 经典代码/对比示例 | data + sealed、launch/async、Version Catalog、MockK、多阶段 Docker…… |
| 第 4 格 · 结案 | 最佳实践 + ⚠ 坑点 + 一句总结 | val 优先、拒绝 GlobalScope、证书固定备用指纹、质量门禁…… |

### 2. 一键打开，零依赖阅读

- 提供 [index.html](./index.html) 卡片式目录页，**浏览器直接打开**即可浏览全部 9 话，无需安装任何插件或工具；
- 每话含「原画 SVG」「讲解 MD」双链接，点击卡片看原画、点链接读知识点；
- SVG 为纯矢量（1200×840 viewBox），任意缩放不失真；MD 为标准 Markdown，所有主流阅读器直接打开。

### 3. 三角色，视觉统一

固定三角色贯穿全系列，降低认知负担：

- **像素酱**（青色高马尾）：提问担当，好奇心强、一针见血；
- **标签君**（橙色刺猬头 + Kotlin 盾徽）：行动派，热情外向、爱动手；
- **码叔**（白发圆框眼镜）：教授，沉稳专业、负责总结。

视觉规范固化在 [style-guide.md](./style-guide.md)，角色定妆稿见 [samples/character-sheet.svg](./samples/character-sheet.svg)。

### 4. 适配主流设备

- 预览页采用 CSS Grid 自适应布局（`repeat(auto-fill, minmax(320px, 1fr))`）+ `clamp()` 流式标题，适配桌面 / 平板 / 手机；
- 原画使用 SVG viewBox 等比缩放，图片 `aspect-ratio: 10/7` 布局稳定，移动端可双击放大阅读代码细节。

### 5. 工程化质量保障

- **manifest.json**：9 话清单为单一事实来源，包含 ep / slug / title / 对应正文章节；
- **check-comics.mjs**：自动校验 manifest 双向一致性、SVG XML 良构性、文本溢出启发式、MD 结构完整性（标题约定 / 正文引用 / 自测题）；
- 每话讲解 MD 固定结构：剧情回顾 → 知识点拆解（四格各一节）→ 坑点清单 → 自测 3 题（含答案），并回链正文与示例。

---

## 三、剧集清单（9 话）

| 话 | 主题 | 对应正文章节 | 核心知识点 |
| ---- | ---- | ---- | ---- |
| [EP.01](./EP.01-null-safety-modeling.md) | 空安全与数据建模 | 第 1 章 Kotlin 基础 | `?.`/`?:`/`as?`/`!!`、data class、sealed class、val 优先 |
| [EP.02](./EP.02-coroutines-structured.md) | 协程与结构化并发 | 第 2 章 Kotlin 高级 | suspend、launch/async/withContext、coroutineScope vs supervisorScope、禁用 GlobalScope |
| [EP.03](./EP.03-gradle-build-system.md) | Gradle 构建体系 | 第 3 章 Gradle 构建 | 三阶段生命周期、register 惰性配置、Version Catalog、convention plugin、api vs implementation |
| [EP.04](./EP.04-jvm-backend.md) | JVM 后端开发 | 第 4 章 JVM 后端 | Spring Boot 3 vs Ktor、WebFlux 协程端点、构造器注入、可观测三件套 |
| [EP.05](./EP.05-android-enterprise.md) | Android 企业开发 | 第 5 章 Android | UI/Domain/Data 分层、单向数据流、Hilt + ViewModel、状态提升、模块化 |
| [EP.06](./EP.06-performance-security.md) | 性能与安全 | 第 6 章 性能安全 | G1 vs ZGC、R8 混淆与 keep、证书固定、Android Keystore、OWASP |
| [EP.07](./EP.07-testing-pyramid.md) | 测试金字塔 | 第 7 章 测试质量 | 70/20/10 分层、MockK + runTest、Turbine、detekt/ktlint 门禁 |
| [EP.08](./EP.08-cicd-deployment.md) | CI/CD 与部署 | 第 8 章 CI/CD | GitHub Actions 五步流水线、多阶段 Docker、Jib、灰度发布、一键回滚 |
| [EP.09](./EP.09-team-collaboration.md) | 团队协作规范 | 第 9 章 团队协作 | Conventional Commits、Trunk-Based、ADR、CODEOWNERS、规范即代码 |

---

## 四、文件结构

```
comics/
├── README.md                       # 漫画剧场导航表与读法说明
├── RELEASE_NOTES.md                # 本发布说明
├── style-guide.md                  # 霓虹墨风格规范（唯一视觉标准）
├── manifest.json                   # 9 话清单（校验脚本依据）
├── check-comics.mjs                # 校验脚本（node comics/check-comics.mjs）
├── index.html                      # 一键预览页（响应式卡片目录）
├── samples/
│   └── character-sheet.svg         # 三角色定妆稿 v2
├── EP.01-null-safety-modeling.svg  # EP.01 原画
├── EP.01-null-safety-modeling.md   # EP.01 讲解
├── EP.02-coroutines-structured.svg / .md
├── EP.03-gradle-build-system.svg / .md
├── EP.04-jvm-backend.svg / .md
├── EP.05-android-enterprise.svg / .md
├── EP.06-performance-security.svg / .md
├── EP.07-testing-pyramid.svg / .md
├── EP.08-cicd-deployment.svg / .md
└── EP.09-team-collaboration.svg / .md
```

同时更新了项目根目录 [README.md](../README.md) 的「配套资源」表，新增漫画剧场入口。

---

## 五、快速开始

### 方式一：直接打开（推荐，零配置）

双击或用浏览器打开 `comics/index.html`，即可看到全部 9 话卡片。

### 方式二：本地静态服务

```bash
cd kotlin-gradle-enterprise-guide
python3 -m http.server 8901
# 浏览器访问 http://127.0.0.1:8901/comics/
```

### 建议读法

1. 先看原画 SVG，跟着四格剧情把概念"看见"；
2. 再读同 slug 的讲解 MD，把每一格落成可运行的机制与代码；
3. 最后通过 MD 中的"对应正文"链接回到主线深读，并跑一遍 `examples/` 配套示例。

### 质量校验

```bash
node comics/check-comics.mjs
# 预期输出：结果：✓ 9 话，0 错误，0 警告
```

---

## 六、技术规范摘要

| 维度 | 规范 |
| ---- | ---- |
| 画布 | 1200×840 viewBox，深墨底渐变（#0b1220 → #101a33） |
| 分格 | 2×2 四格，面板 572×361，圆角 14 |
| 色板 | 60% 深墨底 / 30% 结构线 / 10% 霓虹强调（青/紫/品红/橙/绿/黄） |
| 字体 | PingFang SC / Hiragino Sans GB / Microsoft YaHei；代码 SF Mono / Consolas |
| 对比度 | 浅底气泡配深字、深底正文用 #e2e8f0，≥ 4.5:1 |
| 元素约束 | 纯 SVG 矢量，禁止 image / script / foreignObject |
| 中文编码 | 全部简体中文，代码注释与标签中文友好 |

---

## 七、兼容性与升级影响

- **阅读端兼容**：Chrome / Edge / Firefox / Safari 现代版本均可直接打开 SVG 与 HTML；MD 支持 Typora / VS Code / Obsidian / GitHub 渲染；
- **对现有内容影响**：纯新增，不修改任何正文章节与示例代码，零回归风险；
- **Node 版本**：校验脚本为纯 ESM（.mjs），需 Node.js ≥ 14（仅校验时需要，阅读无需 Node）。

---

## 八、已知限制与后续计划

本版为首版（v1.0.0），当前范围与规划：

- 每章一话，覆盖 9 大主题的**核心机制**；章节内的细分高级主题（如 KSP、泛型星投影、gRPC 等）暂未单独成话；
- 后续可按需追加扩展话（编号 EP.10+），需同步更新 `manifest.json`、`README.md`、`index.html` 三处；
- 暂未提供 PDF 合订本导出；如需离线分发，可直接用浏览器"打印 → 另存为 PDF"。

---

## 九、致谢与反馈

- 视觉风格体系「霓虹墨 Neon Ink」借鉴自团队既有前端漫画剧场实践，本版针对 Kotlin & Gradle 主题调整了角色徽章（Kotlin 盾徽）与代码示例；
- 如发现原画文案、代码片段或讲解有误，欢迎提交 Issue 或 PR，修改后请运行 `node comics/check-comics.mjs` 确保全绿。

---

**学习路径**：[README 总览](../README.md) → [漫画剧场导航](./README.md) → [EP.01 空安全与数据建模](./EP.01-null-safety-modeling.md)
