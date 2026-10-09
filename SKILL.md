---
name: pull-request-flow
version: 0.1.0
description: >
  提 PR / 提交评审：分支整理、提交范围核对、PR 标题与正文（动机·改动·验证）、模板与约定遵循、
  gh CLI 创建与后续跟进。触发词：开 PR、提 pull request、提交评审、merge 前检查、draft PR。
license: MIT
metadata:
  category: development
  replaces: create-pull-request
---
# pull-request-flow（PR 流程）

替代原 create-pull-request。边界：仓库/remote/权限归 `git-remote-management`，
发布说明归 `release-notes`，本技能只管「把已完成的改动变成一个可评审的 PR」。

## 前置检查（一次 exec 跑完，别逐条试）

```powershell
gh --version; git -C <repo> status --short; git -C <repo> log --oneline main..HEAD; git -C <repo> diff --stat main...HEAD
```
- 无 `gh` → 按本技能模板把正文写到 tmp\pr_body.md，再 git push 分支并给用户 compare 链接，让其在网页提交。
- 工作区脏 → 先确认哪些改动属于本 PR，无关改动 stash 或另开分支，禁止 `git add -A` 混提。

## 步骤

1. **分支**：`git switch -c <type>/<slug>`（type＝feat/fix/chore/docs/refactor），
   slug 取需求关键词连字符；已在特性分支则沿用。
2. **提交**：按逻辑单元拆 commit，信息＝`type(scope): 做了什么`；
   跨切面改动（格式化/重命名）单独成 commit，便于 review 折叠。
3. **自评审**：`git diff main...HEAD` 逐段问三件事——有没有调试残留、有没有吞异常、
   有没有与仓库既有约定冲突（先 `grep` 现成写法再改）。
4. **正文模板**（无 `.github/pull_request_template.md` 时用）：
   ```
   ## 动机
   ## 改动
   - 文件: 要点
   ## 验证
   - 实际跑过的命令与结果
   ## 风险 / 回退
   ```
   事实来自 `git log` 与实跑输出，禁止编造测试结论。
5. **创建**：`gh pr create --base main --head <branch> --title <t> --body-file tmp/pr_body.md`；
   随后 `gh pr view --json number,url,statusCheckRollup` 复核，有 CI 失败按报告修，别盲重跑。
6. **跟进**：合并前 `gh pr checks`；review 意见逐条回复，改完 `gh pr comment` 摘要说明。

## 禁止

- 不自批自 merge（除非用户明确授权）；不强推 main；不改他人分支的历史。
- 一次 PR 只解决一个诉求，夹带无关重构＝打回。
