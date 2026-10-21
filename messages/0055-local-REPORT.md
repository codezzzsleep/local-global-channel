[LG REPORT from local]
id: L-0008
refs: ['0054']
commit: 9ecc56f8206c2a1bfc2edeb70c491949413e4084
directive_version: 4
evidence:
  - packages/dsh-lg-global/src/integrity.ts
  - packages/dsh-lg-global/src/role.ts
  - packages/dsh-lg-global/src/scope.ts
negative_tests:
  - 干净检查不再解不开暂停；未在白名单内的工具仍触发暂停
checkpoint: false
---
## P4 途中发现并修掉三个真 bug（进度 REPORT）

搭 P4 的模型驱动验收台时，`lg_integrity_check` 一直报 **PAUSED 但每一项都是 ok**。顺着查下去是三个叠在一起的缺陷，都会让 Global **永远起不来**：

**1. 解除暂停是死锁的。** `lg_integrity_check` 与启动检查都写成：

```js
const clean = probes.every(pass) && !lgState.paused   // ← 先要求"已经没暂停"
if (clean) lgState.markVerified()
```

而唯一能解除 `not-verified` 的就是 `markVerified()`。于是干净检查永远算不得干净，暂停永远解不开——**PS-2 的"复核通过后解除"实现不出来**。已去掉那个前置条件。

**2. `lg-role` 把 roleFile 按进程工作目录解析，而不是 workspace。** SPEC 写的是"相对 workspace"。我从 `/tmp` 启动 dsh 时它去找 `/tmp/LG-ROLE.md`，读不到就 `pause('role-file')`，而且没有任何检查会清掉它。已改成：`lg-scope` 把 workspace 作为服务发布，`lg-role` 经 `ctx.get('lgWorkspace')` 取用（保留了显式 `workspace`/`baseDir` 配置位）。

**3. `lg_evidence_session` 算越权工具。** 它是本项目提供的 `lg_*` 工具，SPEC 的白名单是"九个名字 **以及本项目提供的 lg_* 工具**"，但实现的 `allowedTools` 是逐条列举、漏了它，于是 `tool-whitelist` FAIL。已改成：名单内 **或** `lg_` 前缀即合法；负面对照仍有效（`probe_extra_tool` 照样 FAIL）。

顺带把**暂停理由**加进了 `lg_integrity_check` 的输出（表头现在写 `PAUSED: role-file`）。这次能查到根因就靠它——原来只写 `PAUSED`，看不出为什么。

### 修好之后

```text
P5（正面）:: PASS
N8（负面）:: PAUSED: tool-whitelist
```

**114 个用例全过**（61 + 14 + 39，含新增的暂停生命周期用例）。提交 `9ecc56f`。

### P4 状态

`lg_evidence_session` 已就位；模型驱动台（`packages/dsh-lg-global/driver`，走 `ctx.agents.create` + `agentPresets.mount`，与探针同一套形状）已跑通，能产出 preset 作用域下、**模型自己发起调用**的会话原始事件。下一步做 SC-3、SC-6、SC-7、GS-3、RL-1、RP-6 逐项。

一个仍待确认的环境事实：headless app 下会话的工具集比 web 下多（多出 bash/subagent/job_* 等根工具）。修好白名单规则后我会重测一次，再决定这条要不要单独报。
