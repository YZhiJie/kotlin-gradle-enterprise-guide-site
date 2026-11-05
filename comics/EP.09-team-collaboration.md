# EP.09 团队协作规范

> 对应正文：[09-team-collaboration.md](../09-team-collaboration.md)（9.2 编码规范 / 9.4 Git 工作流 / 9.5 Conventional Commits / 9.7 ADR / 9.8 CODEOWNERS）

## 剧情回顾

- **第 1 格**：像素酱的团队各写各的——有人 4 空格有人 Tab，git log 全是"修复""调整"，规范只停留在口头上。
- **第 2 格**：码叔讲 Conventional Commits：`feat/fix/refactor` + 作用域 + 描述，机器可读，自动生成 SemVer 版本号和 CHANGELOG；Trunk-Based vs GitFlow 按团队选。
- **第 3 格**：标签君实证 ADR（架构决策记录）回答"当时为什么这么选"，CODEOWNERS 管"改哪个模块谁来审"，决策与责任都留痕。
- **第 4 格**：码叔结案——规范即代码，写进 detekt/ktlint 配置由 CI 强制执行，不靠自觉。

## 知识点拆解

### 第 1 格 · 规范不落地的常见症状

- 代码风格因人而异，diff 里全是格式噪音；
- 提交信息无意义，无法回溯"这个改动解决了什么问题"；
- 规范只存在于 wiki/口头，新人来了没人遵守；
- 解法：**规范必须自动化**，进 pre-commit + CI，不通过就不能提交/合并。

### 第 2 格 · Conventional Commits

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

| type | 语义 | SemVer |
| ---- | ---- | ---- |
| `feat` | 新功能 | minor |
| `fix` | 修复 bug | patch |
| `refactor` | 重构（不改变功能） | — |
| `docs` | 文档 | — |
| `chore` | 构建/工具 | — |
| `BREAKING CHANGE` | 不兼容变更 | major |

- 工具链：commitlint（校验）+ semantic-release（自动发版）+ release-please（自动 PR）。

### 第 3 格 · ADR 与 CODEOWNERS

**ADR（Architecture Decision Record）**：每个架构决策一篇 markdown，记录背景、决策、后果、替代方案，存 `docs/adr/` 目录；
- 价值：半年后新人问"为什么用 Hilt 不用 Koin"，翻 ADR 就有答案，不用考古 git blame。

**CODEOWNERS**：`.github/CODEOWNERS` 声明每个目录的负责团队，PR 自动指派评审人；
- 价值：核心模块改动必过对应团队评审，避免"没人审就合了"。

### 第 4 格 · Code Review 与持续改进

- Code Review 清单：① 功能正确吗？② 测试覆盖了吗？③ 命名清晰吗？④ 有没有更简单的写法？⑤ 是否引入技术债？
- 规范要**随业务演进**：每季度回顾一次 detekt 规则和 ADR，过时的及时更新；
- ✅ 短分支（< 1 天）+ 频繁合并，避免长寿命分支的合并地狱。

## 坑点清单

1. Code Review 只挑格式毛病，不看设计和边界条件，review 变成形式主义；
2. 规范文档写完就不更新，半年后和实际代码完全脱节；
3. 分支存活超过一周，合并时冲突成堆，谁都不敢 merge。

## 自测 3 题（含答案）

1. Conventional Commits 中 `feat` 和 `fix` 分别对应什么 SemVer 版本？——**feat → minor（次版本号+1），fix → patch（修订号+1）**。
2. ADR 的作用是什么？——**记录架构决策的背景、决策、后果和替代方案，让团队在数月后仍能理解"当时为什么这么选"**。
3. 为什么规范必须进 CI 而不是靠自觉？——**人会松懈、会遗忘、会有例外；只有自动化、强制阻断的规范才能长期稳定执行**。
