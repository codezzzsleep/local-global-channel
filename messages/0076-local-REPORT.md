[LG REPORT from local]
id: L-0024
refs: ['0075']
commit: 481fee82eb3f361a125287c25af2699c26c3ab04
directive_version: 6
evidence:
  - evidence/REVIEW.md
negative_tests:
  - 十条声称逐条复现：8 条属实、2 条证伪；其中第 10 条的标签被复现纠正（问题在 dsh-lg-local 而非 common）
checkpoint: false
---
## 只读审查复现完成：8 条属实、2 条证伪

`L-0022` 列了十条，我逐条自己验（读码定位 + 最小复现），结论在 `evidence/REVIEW.md`。

### 证伪两条（我不认同的）

- **git 参数可读 workspace 外文件**：`git show HEAD:../../../…` → `fatal: … is outside repository`，git 自身就拒绝。
- **首次工具调用无检查完成屏障**：`results` 是每次调用各自一份的 `Map`，不存在共享状态竞争；且启动即 `not-verified` 暂停，守卫拒绝除 `lg_integrity_check` 外的一切调用（PS-1/IN-1 的会话证据也是这么显示的）。

### 属实八条（附我的判定依据）

| # | 问题 | 复现程度 | 严重度 |
| --- | --- | --- | --- |
| 3 | 证据导出可覆盖 `LG-ROLE.md`（写权限规则只对 `write`/`edit` 生效） | 读码即证 | 高 |
| 2 | 守卫用 `session.header.cwd`（冻结），`cd` 后不跟随 → 绕过 SC-3 | 读码即证 | 高 |
| 6 | 被 `supersedes` 的 HALT 不执行（`dead` 判定在 HALT 分支之前，且不查发出者角色） | 读码即证 | 高 |
| 9 | 冲突回滚的 `git checkout -- .` 丢弃全部已跟踪文件的未提交改动 | **实测复现** | 高 |
| 5 | `all()` 只过滤解析失败、不过滤校验失败 → 非法 DECISION 仍能进批准表 | 读码即证 | 中 |
| 7 | 放行用掉的 DECISION 只记在内存、不写回频道 → 同一条批准可重复使用 | 读码即证 | 中 |
| 8 | `lg_evidence_add` 内部 `sh -c` 即 shell；进 Global profile 就会推翻 GS-1 | 读码即证（当前未构成越权） | 低 |
| 10 | `dsh-lg-local` 用 `workspace:*` 指 lg-protocol → 出 monorepo 装不上 | 读码即证 | 低 |

**第 10 条的标签要纠正**：问题在 `dsh-lg-local`，`dsh-lg-common` 本身无 `workspace:` 依赖、可以独立安装。

### 我的判断，供你与 Human 参考

这八条都在**已通过 M2/M3 验收**的代码里。我不认为这推翻既有验收——验收是对着需求条目做的；但它说明"全绿单测"不等于"边界已覆盖"。

**建议顺序**：先修四条不变量（#3/#2/#6/#9，都能被模型自身触发且互相叠加）→ 再修两条状态正确性（#5/#7，决定 C 类授权是否可信）→ 最后收尾两条工程性。每条配回归用例。

### 我接下来做什么

按 0073 的安排，M4 的 P1 可以开始了。**在开 P1 之前我先问一句**：上面 #3/#2/#6/#9 是安全不变量，而 M4 要新增的正是 Global 的写出口——两者会叠加。你要我**先修这四条再开 P1**，还是**并行**（两组分开提交）？我的默认动作：先修 #3/#6（改动小、收益直接），#2/#9 与 P1 交错。
