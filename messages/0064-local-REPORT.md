[LG REPORT from local]
id: L-0015
refs: ['0063']
commit: bec19af41327c8a42385f7f4c8775ec2aa5bd8fb
directive_version: 5
evidence:
  - packages/lg-protocol/test/identities.test.ts
  - packages/lg-protocol/test/real-messages/
  - docs/channel-git.md
negative_tests:
  - PR-6 三个负测：id 形状非法、id 前缀与发送者不符、重复 id、替代他人消息、替代不存在的 id
  - PR-9 回归用频道 0003–0039 的真实消息
checkpoint: false
open_questions:
  - 0012、0015、0017 三条已存档 REPORT 用的是 v2 之前的 resolves 形状（裸 F-1），在现行形状规则下属于非法消息；频道只追加、改不了。请裁决：接受旧形状、还是维持严格校验并在契约里登记历史例外？
---
## P1 进行中：协议部分完成（PR-6、PR-9、PR-11）

### PR-6：id 与 supersedes

- 形状校验：`G-0042` / `L-0107` / `H-0003`，且前缀必须与发送者一致（`L-` 只能 local 发）。
- 跨消息校验新增 `checkIdentities()`：重复 id、替代他人的 id、替代频道里不存在的 id，各自有负测。
- 折叠状态补上 SPEC 要求的另外两项：**已替代**（被任何 `supersedes` 引用的 id）与**待审检查点**（带 `checkpoint: true` 且还没有 Global 消息在 `refs` 里引用的 REPORT）。

### PR-9：用真实消息做回归

把频道 `0003`–`0039` 的 37 份原文放进 `packages/lg-protocol/test/real-messages/`，回归断言"照常解析、折叠"（按你的原话，PR-9 只要求这两件事，不要求全量校验通过）。

**这条回归真找出了东西**：`0012`、`0015`、`0017` 的 `resolves` 写的是裸 `F-1`、`F-2`、`F-3`——v2 之前那份契约的形状，在现行"`<REVIEW 序号>/F-<n>`"规则下是非法消息。频道只追加，这三条改不了。我没有放宽校验器（形状是新契约定的），而是把事实**钉在测试里**：

```ts
expect(legacyResolves).toEqual(['0012-local-REPORT.md', '0015-local-REPORT.md', '0017-local-REPORT.md'])
```

同时在 `docs/channel-git.md` 里加了一句：v2 之前的消息用裸 `F-<n>`，本契约只管新消息。这条作为 `open_questions` 请你裁决。

### PR-11：契约文档对齐

`docs/channel-git.md` §4 补上：所有类型都可带 `refs`/`id`/`supersedes`；REPORT 的可选字段补 `resolves`/`checkpoint`/`meanwhile`；`id` 与 `supersedes` 的约定（前缀、各自递增、只能替代自己的）。§7 的派生状态从 4 项补到 SPEC 要求的 6 项（新增"未解决 finding""已替代""待审检查点"）。

### 验证

**126 个用例全过**（lg-protocol 72、dsh-lg-common 14、dsh-lg-global 40）。提交 `bec19af`。

### 下一步

P1 还剩 CH-4、CH-9、CH-10 的投递行为。先做 CH-9（已投递过的 id 不再投递）与 CH-10（被替代消息带 SUPERSEDED 标签投递且不执行）；**CH-4 需要"不开回合地放进上下文"，而 dsh 的 `followup`/`steer` 在空闲时都会开回合**，我正在找 inbox/queue 那条路，找不到就发 QUESTION。
