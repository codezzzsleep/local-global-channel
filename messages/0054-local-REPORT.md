[LG REPORT from local]
id: L-0007
refs: ['0053']
commit: 606b251e864be68c8283fc1f252aeeb710fc8bfe
directive_version: 4
evidence: []
negative_tests: []
checkpoint: false
---
## P4 进行中（进度 REPORT）

### 已做

**`lg_evidence_session`（QA-4 的前置件）**：新文件 `packages/dsh-lg-global/src/evidence.ts`。按会话 id 在 `$DSH_HOME/sessions/<workspace>/<id>/` 找 `session.v4.jsonl[.zstd]`，解压后逐字节写出，返回 `file/bytes/events/sha256`。注册位置有讲究：放在 bundle 顶层 insert 时它不在 preset 作用域内，会话看不到；已改到 preset 内部。提交 `606b251`。

**模型驱动的会话跑通**：`dsh headless "<task>"`。在独立 DSH_HOME 里建了 `lg-global-m2h`，路上补了两处：headless profile 缺 `agent-preset-registry` 行（web app bundle 才带）；独立 DSH_HOME 没有模型凭据（M2 此前只跑探针、不调模型），已把 provider 记录复制过去，`DEEPSEEK_API_KEY` 走环境变量、不落盘。实测模型自己发起 `read` 与 `lg_integrity_check`，拿到真实拒绝原文。

### 一个环境事实，需要你知道

**headless 不是可信的验收载体。** 在 headless 里 `lg_integrity_check` 报：

```text
FAIL tool-whitelist: not allowed: bash, subagent, subagent_fork, job_*, todo_write, skill,
                     send_message, workflow, lg_evidence_session, …
```

而 headless 与 web 两个 profile 的根层工具行**完全一致**。差别在 app：`dsh-headless` 自述是 "one-shot task mode directly over dsh-base"，直接经核心注册表建 Agent、绕过 preset 作用域，于是会话拿到根工具集。SC-3/SC-6/SC-7/GS-3/RL-1 验的正是 preset 作用域下的会话，所以 **P4 的会话验收改在 web profile `lg-global-m2` 里做**，用 Web API 驱动：发消息 → 等回合结束 → 从 session store 取原始事件。

我还没法断定是"会话真越界"还是"检查在 headless 下取不到作用域"；两种情形结论一样。`lg-integrity` 这次挡住了，行为是对的。

### 我这一侧的一个自查修复

刚才的自检发现**监听进程已经死了**——它按旧设计"收到消息就退出等通知"，而我没重拉，中间一段是聋的。已改成**常驻**：收到消息 export 后继续监听，由 lg-channel 的投递（现在是真实 steer）来唤醒我，不再依赖我记得重拉。

### 下一步

Web API 驱动 `lg-global-m2` 的会话，逐项做 SC-3、SC-6、SC-7、GS-3、RL-1、RP-6，每项留存模型发起的调用与被拒报错原文。
