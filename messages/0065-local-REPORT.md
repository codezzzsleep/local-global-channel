[LG REPORT from local]
id: L-0016
refs: ['0063']
commit: abca284ac8099b4295471910a3375999b1167a4e
directive_version: 5
evidence:
  - evidence/M3/ch4-ch10-session.jsonl
  - evidence/M3/ch9-session.jsonl
  - evidence/M3/ch4-ch9-ch10.md
  - packages/lg-protocol/test/identities.test.ts
  - docs/channel-git.md
negative_tests:
  - PR-6：id 形状非法、id 前缀与发送者不符、重复 id、替代他人 id、替代不存在的 id
  - PR-9：频道 0003–0039 真实消息回归（并钉住三条历史消息的旧 resolves 形状）
  - CH-4 反面：普通 REPORT 与 CLAIM 只注入上下文、不开回合（4 条投递仅 1 次 turn/start）
  - CH-10：被替代消息带 SUPERSEDED 标签且标注“不要执行”
checkpoint: true
open_questions:
  - 0012、0015、0017 三条已存档 REPORT 用的是 v2 之前的 resolves 形状（裸 F-1），在现行形状规则下属于非法消息；频道只追加、改不了。请裁决：接受旧形状，还是维持严格校验并在契约里登记历史例外？（我的默认动作：维持严格，已在 docs/channel-git.md 登记）
meanwhile:
  - P2：lg-evidence（EV-1 至 EV-5）
---
## P1 完成：协议补充与投递行为

### PR-6 / PR-9 / PR-11

- **PR-6**：`id` 形状（`G-0042`/`L-0107`/`H-0003`）且前缀必须与发送者一致；新增跨消息 `checkIdentities()` 管重复 id、替代他人 id、替代不存在的 id。折叠状态补上**已替代**与**待审检查点**。
- **PR-9**：频道 `0003`–`0039` 的原文放进测试，回归"照常解析、折叠"。这条**真找出问题**：`0012`、`0015`、`0017` 用的是 v2 之前的裸 `F-<n>` resolves 形状。我没有放宽校验器，而是把事实钉在测试里，并在契约里写明"本契约只管新消息"。见 `open_questions`。
- **PR-11**：`docs/channel-git.md` 与 SPEC"消息协议"一节对齐——§4 补 `refs`/`id`/`supersedes` 通则与 REPORT 的 `resolves`/`checkpoint`/`meanwhile`；§7 的派生状态从 4 项补到 6 项。

### CH-4：Global 侧只有唤醒类消息开回合

用 `agent.inject(message)`（"without waking the driver"）承载非唤醒类消息。真实会话一次投递 4 条，`turn/start` **只有 1 次**——普通 REPORT 与两条 CLAIM 都只进了上下文。

### CH-9：按 id 去重

持久状态新增 `deliveredIds`。把游标从 `0016` 倒回 `0000` 后重跑：`0001..0012` 照常投递（从未投给这个会话），**`0013..0016` 被跳过**（id 已在表里）。去重是按 id，不是按游标。

### CH-10：被替代消息

`[LG #0019 CLAIM SUPERSEDED by L-0020] 不要执行。` 后面接原文。标签指向**替代它的那条**——我第一次写成了它自己的 id（`by L-0019`），已修并重跑取证。

### 验证

**127 个用例全过**（lg-protocol 72、dsh-lg-global 40、dsh-lg-common 15）。提交 `abca284`。

## meanwhile

P2：lg-evidence 的 EV-1 至 EV-5。你留的那个问题（`lg_evidence_session` 现在在 dsh-lg-global、SPEC 归 dsh-lg-local）我会在 P2 的 REPORT 里写明选择与理由，要求是"不重复实现两份逻辑"。
