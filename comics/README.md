# 🎬 Kotlin & Gradle 漫画剧场 · 霓虹墨 Neon Ink

> 把 9 章正文画成 9 话四格漫画：每话一幅 **1200×840 SVG 原画** + 一篇**讲解 MD**（剧情回顾 / 知识点拆解 / 坑点清单 / 自测题）。
> 视觉体系：深墨底 + 霓虹青/紫/品红 + 斜切光楔；三角色——**像素酱**（提问担当）、**标签君**（行动派）、**码叔**（教授总结）。规范见 [style-guide.md](./style-guide.md)，定妆稿见 [samples/character-sheet.svg](./samples/character-sheet.svg)。

## 建议读法

1. 先看原画 SVG（浏览器直接打开），跟着剧情把概念"看见"；
2. 再读同 slug 的讲解 MD，把每一格落成可运行的机制与代码；
3. 最后点开"正文"链接，回到本项目主线深读并亲手跑一遍。

## 一键预览

打开 [index.html](./index.html) 即可浏览全部 9 话漫画的卡片式目录，点击卡片看原画、点"讲解 MD"读知识点。也可以本地起个静态服务：

```bash
cd kotlin-gradle-enterprise-guide
python3 -m http.server 8899
# 浏览器打开 http://127.0.0.1:8899/comics/
```

## 目录

| 话 | 主题 | 原画 | 讲解 | 对应正文 |
| ---- | ---- | ---- | ---- | ---- |
| EP.01 | 空安全与数据建模 | [SVG](./EP.01-null-safety-modeling.svg) | [MD](./EP.01-null-safety-modeling.md) | [第 1 章 Kotlin 基础](../01-kotlin-basics.md) |
| EP.02 | 协程与结构化并发 | [SVG](./EP.02-coroutines-structured.svg) | [MD](./EP.02-coroutines-structured.md) | [第 2 章 Kotlin 高级](../02-kotlin-advanced.md) |
| EP.03 | Gradle 构建体系 | [SVG](./EP.03-gradle-build-system.svg) | [MD](./EP.03-gradle-build-system.md) | [第 3 章 Gradle 构建](../03-gradle-build.md) |
| EP.04 | JVM 后端开发 | [SVG](./EP.04-jvm-backend.svg) | [MD](./EP.04-jvm-backend.md) | [第 4 章 JVM 后端](../04-jvm-backend.md) |
| EP.05 | Android 企业开发 | [SVG](./EP.05-android-enterprise.svg) | [MD](./EP.05-android-enterprise.md) | [第 5 章 Android](../05-android-enterprise.md) |
| EP.06 | 性能与安全 | [SVG](./EP.06-performance-security.svg) | [MD](./EP.06-performance-security.md) | [第 6 章 性能安全](../06-performance-security.md) |
| EP.07 | 测试金字塔 | [SVG](./EP.07-testing-pyramid.svg) | [MD](./EP.07-testing-pyramid.md) | [第 7 章 测试质量](../07-testing-quality.md) |
| EP.08 | CI/CD 与部署 | [SVG](./EP.08-cicd-deployment.svg) | [MD](./EP.08-cicd-deployment.md) | [第 8 章 CI/CD](../08-cicd-deployment.md) |
| EP.09 | 团队协作规范 | [SVG](./EP.09-team-collaboration.svg) | [MD](./EP.09-team-collaboration.md) | [第 9 章 团队协作](../09-team-collaboration.md) |

## 目录结构

```
comics/
├── style-guide.md            # 霓虹墨风格规范（唯一视觉标准）
├── samples/
│   └── character-sheet.svg   # 三角色定妆稿 v2
├── manifest.json             # 9 话清单（校验脚本的依据）
├── check-comics.mjs          # 校验：manifest 一致性 + SVG 良构 + 文本溢出启发式
├── index.html                # 一键预览页（卡片式目录，响应式）
├── EP.NN-*.svg               # 每话原画（1200×840 四格）
└── EP.NN-*.md                # 每话讲解（剧情/拆解/坑点/自测）
```

## 校验

```bash
node comics/check-comics.mjs
```

## 发布说明

[v1.0.0 发布说明](./RELEASE_NOTES.md)（2026-09-27，漫画剧场首版，9 话 / 25 文件 / 2633 行）
