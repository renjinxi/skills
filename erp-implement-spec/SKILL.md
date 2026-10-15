---
name: erp-implement-spec
description: 在 erp-main 多仓 workspace 里按 /implement-spec 把一份 spec 及其票全部实现，自带本仓的分支、worktree、推送、收尾与测试约定。
disable-model-invocation: true
argument-hint: "<spec issue 号，如 82>"
---

用 `/implement-spec` 的流程，把 GitLab `ai-team/erp-main` 上的一份 spec 和它的票全部实现。本 skill 只补 erp-main 多仓 workspace 与 skill 默认做法不同的地方；流程本身以 implement-spec 为准。

## 1. 摸清现场

1. **确认在 workspace 里**：当前目录（或其上层）有 `workspace.yaml` 和 `repos/`，且**不是**主工作区 `~/work/erp-main`。在主工作区就停下，让用户先用 `/erp-ws-feat` / `/erp-ws-fix` 起 workspace 再开新会话。
2. **定集成分支**：对 `repos/*` 逐个 `git -C <仓> rev-parse --abbrev-ref HEAD`，取它们共同的需求分支（如 `feat/pet0945-card-goods-limit`）。这就是集成分支，**不另建**。
3. **读 spec 和票**：`glab issue view <spec号> -R ai-team/erp-main --comments`；票 = 正文里写 `Parent` 指向该 spec 的 issue（`glab issue list -R ai-team/erp-main --search "#<spec号>" --all` 后逐个读正文核对）。依赖以每张票正文的「Blocked by」为准（本 GitLab 不支持原生 blocks 链接）。从票里列出每张涉及的子仓。
4. **定笔记目录**：从分支名 / spec 取 PET 编号，在 `~/work/erp-main/notes/tasks/` 下找对应目录（找不到就新建 `PETxxxx-<slug>/`）。读其中的 README 拿背景和决策。

完成标准：能写出「集成分支名 / 票号 + 依赖 + 各票涉及子仓 / 笔记目录」四项，并简短告知用户后继续。

## 2. 执行

Read `~/.claude/skills/implement-spec/SKILL.md`，按它的步骤执行（它是 user-only skill，不能用 Skill 工具调用），并用下面的约定覆盖它的默认做法。给每个 subagent 的指令里都带上这一节和第 3 节的指针（本文件路径），不要复述大段内容。

### 覆盖约定

- **glab 每条命令都带 `-R ai-team/erp-main`。**
- **集成分支**：用第 1 步定下的已有分支，不建新分支，不开 draft PR / MR。
- **每票隔离用子仓自己的 git worktree**：`git -C repos/<仓> worktree add <路径> -b <集成分支>-t<票号> <集成分支>`。**不要**用 Agent 的 `isolation: worktree`——它只隔离 host 仓。两张并行票改同一子仓时必须各开各的 worktree。合并回集成分支由 merger subagent 做。
- **推送**：每票合进集成分支后 `git push origin HEAD:refs/heads/<集成分支>`（workspace 的 feat 分支 upstream 指向 baseline，裸 `git push` 会被拒）。不开 MR、不用 `glab mr`；全部完成后由用户自己跑 `/erp-mr`。
- **票的收尾**：在 issue 上评论本票 commit 与验证结果，然后关闭该 issue。**不改 spec issue**，**不动 Peter 卡状态**。
- **笔记**：过程笔记、探索笔记一律写到第 1 步的 `~/work/erp-main/notes/tasks/<目录>/`，不写进 worktree（回收会删）。
- **范围**：只改票上写的端；没改动的端不扩成测试工程，要扩先问用户。
- **最后的 code-review** 照 implement-spec 做，修复在一个 implementer subagent 里完成后同样推送。
- 收尾清理只删本次为票开的子仓 worktree 和 `-t<票号>` 分支，不碰 workspace 本身。

## 3. 测试与仓库坑（subagent 必读）

**erp-lt-vv**

- 不要 `git add -A`；编译用 `go build -o /tmp/...`（裸 `go build ./main.go` 会在根生成 109MB 的 `main`）。
- 单测缺 `conf/local_config.yaml` 会静默跳过（显示 ok 但零执行）：先 `cp conf/config.yaml conf/local_config.yaml`，用 `-v` 看到 `=== RUN` 再信。impl 包要 `-vet=off`。本地库 `ERP_TEST_DB_PORT=3306 ERP_TEST_DB_NAME=autoapp_local`。
- 生产不跑 AutoMigrate：加列 / 建表要另给迁移 SQL（照现有 migrations 写法），并在票评论里注明「迁移先于代码上线」。
- 测试失败先在 `origin/<baseline>` 上跑同一套对比，基线本来就红的不认领、自己引入的不放过。

**E2E**

- 先加载 `$erp-localdev-docker`，再在 workspace 根目录：`bash tools/work/work.sh install` → `./localdev e2e setup` → `./localdev e2e run <端> <带目录前缀的文件>`（掌柜端 `tests/`、运营后台 `e2e/`；裸文件名会回落成全量）。
- 账号固定 `15735404577:000000`，不得因缺 `.env` 换账号。
