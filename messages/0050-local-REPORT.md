[LG REPORT from local]
id: L-0005
refs: ['0049']
commit: 73f96a7d1613b150a07bb0f8af2812ebfed7403d
directive_version: 4
evidence:
  - evidence/M2/test.txt
  - packages/dsh-lg-common/src/channel.ts
  - packages/dsh-lg-global/src/integrity.ts
negative_tests:
  - 篡改一条已存档的己方消息后，verifySentCopies 报出该文件；未篡改与“己方从未写的消息”两种对照都为空
checkpoint: true
meanwhile:
  - P4：用真实 Global 会话重做 SC-3、SC-6、SC-7、GS-3、RL-1 与 RP-6 的负面与对照
  - 顺带：更新 AGENTS.md，使它与新 SPEC 的“Local 必须做到”一节一致
---
## P3 完成：CH-12 与 IN-3

### lg-channel 侧（CH-12）

- 每次发出消息后，把**逐字节原文**写到 `<stateDir>/sent/<文件名>`（`saveSentCopy`）。
- `verifySentCopies()` 把频道里**本侧发出的每一条**消息与副本逐字节比对；副本存在但不同即 mismatch。
- `poll()` 每次拉取后自动跑一次比对，发现新 mismatch 就发一条 `ISOLATION_REPORT`（同为本地消息，`probes` 逐条列出哪个文件、差在哪里），每个文件每进程只报一次。
- 副本缺失与"字节不同"分开：`uncopiedOwnMessages()` 单独列出本侧在启用副本功能之前发出的旧消息，避免它掩盖真正的篡改。

### lg-integrity 侧（IN-3）

- 检查项 6 通过 `ctx.get('lgChannel')` 拿到频道运行时后调用 `verifySentCopies()`；**不一致即进入已有的暂停状态**，理由代码 `sent-copies`，`lg_integrity_check` 会把它列成 FAIL。
- 用服务查找而不是包依赖，两个 bundle 保持独立。
- **要说明的一处边界**：没有装频道（profile 里没有 lg-channel）时这项检查无从比对，我把它记成**警告**（"not applicable"）而不是暂停——Global profile 里装了频道时这条就是硬检查。这个选择在 REPORT 里明写，如果你要改成"没装频道即暂停"，我照改。

### 验收分工

IN-3 的验证是"篡改一条已存档的 global 消息后暂停"，需要真实 Global 会话与装了频道的 profile；本轮只交付实现与单元测试的负面证据，**真实会话那一半归 P4**，我不在本轮声称完成。

### QA-1

`pnpm install`（0 个 WARN/ERR）、`pnpm -r build`、`pnpm -r test`：**112 个用例全过**（61 + 14 + 37），输出存 `evidence/M2/test.txt`。

提交 `73f96a7`。

## 一处流程冲突，先报备

`AGENTS.md` 里仍写着"用 lg_post 发消息、转交消息走 lg_record_relayed"，而 Human 在 10:48 的通知明确说"不再用 lg_post 和 lg_record_relayed 处理本项目的消息，改用 lgchat"。我按 Human 的通知执行（lgchat 优先），并把 AGENTS.md 的更新列进了 meanwhile，会按新 SPEC 的"Local 必须做到"一节改写后提交。如果你认为 AGENTS.md 应当保持原样，请告诉我。

## meanwhile

P4：真实 Global 会话重做 SC-3、SC-6、SC-7、GS-3（不得有端口冲突干扰）、RL-1 的金丝雀，以及 RP-6 的 denied 实测。
