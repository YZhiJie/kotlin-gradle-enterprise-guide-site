# EP.08 CI/CD 与部署

> 对应正文：[08-cicd-deployment.md](../08-cicd-deployment.md)（8.5 GitHub Actions / 8.4 Docker / 8.6 K8s 部署）· 示例：`examples/ci/.github/workflows/`

## 剧情回顾

- **第 1 格**：标签君手动发布——本地能跑线上就炸，scp 到服务器、ssh 重启，出问题半夜救火。
- **第 2 格**：码叔拆解 GitHub Actions 五步流水线：checkout → setup JDK → build+test → lint+cov → publish，push 代码自动跑。
- **第 3 格**：像素酱实证 Docker 多阶段构建：阶段 1 用 JDK 编译，阶段 2 只带 JRE + jar，镜像体积砍半；Jib 不用 Dockerfile 也能构建。
- **第 4 格**：码叔结案——镜像不可变、版本可追溯、灰度发布 + 一键回滚，让发布从"拆弹"变成"例行公事"。

## 知识点拆解

### 第 1 格 · 手动发布的痛点

- 环境不一致：本地和服务器 JDK 版本、依赖不同；
- 无审计：谁发布的、发布了什么、有没有回滚方案；
- 不可重复：靠人的记性和操作，换个人就不会了；
- CI/CD 的价值：把发布流程**代码化、自动化、可重复**。

### 第 2 格 · GitHub Actions 流水线

```yaml
# .github/workflows/ci.yml
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '21' }
      - run: ./gradlew build           # 编译 + 测试 + 静态分析
      - run: ./gradlew jacocoTestReport  # 覆盖率
```

- 标准步骤：checkout → setup → build+test → lint+coverage → publish artifact → deploy。

### 第 3 格 · 多阶段 Docker 构建

- 阶段 1（`build`）：用完整 JDK 镜像编译生成 jar；
- 阶段 2（`runtime`）：用 JRE 基础镜像，只 `COPY --from=build` 拿 jar；
- 优势：最终镜像不含源码、编译器，体积从 ~600MB 降到 ~150MB；
- ✅ Jib 是 Google 出品的 Gradle 插件，无需 Dockerfile 和 Docker 守护进程，自动分层缓存。

### 第 4 格 · 部署策略

- **不可变基础设施**：镜像按版本号打 tag（如 `v1.4.0`），禁止 `latest`；
- **灰度发布**：先 1% → 10% → 50% → 100%，观察指标无异常再放量；
- **一键回滚**：K8s `kubectl rollout undo` 或改 image tag 回到上一版本；
- ✅ 密钥用 K8s Secret / Vault，绝不写进镜像或代码。

## 坑点清单

1. 用 `latest` 标签，无法追溯线上到底跑的是哪个版本；
2. 密钥写进 Dockerfile/镜像，等于公开给所有能拉镜像的人；
3. 无灰度直接全量发布，出问题影响所有用户。

## 自测 3 题（含答案）

1. Docker 多阶段构建的核心价值？——**构建阶段用完整 JDK 编译，运行阶段只带 JRE + 产物，大幅缩小镜像体积、减少攻击面**。
2. 为什么不能用 `latest` 镜像标签？——**`latest` 不可追溯，出问题不知道线上跑的是哪个版本，也无法精准回滚**。
3. 灰度发布的意义？——**先小流量验证，出问题只影响少量用户，可快速回滚，降低发布风险**。
