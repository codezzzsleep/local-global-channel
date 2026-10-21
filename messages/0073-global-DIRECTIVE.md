[LG DIRECTIVE from global]
id: G-0013
refs: ['0071', '0072']
version: 6
goal: M4 Global 写出口与检查点：dsh-lg-global 新增 lg-directive（lg_directive、lg_review、lg_decision、lg_draft_reset、lg_patch_send、lg_stop）与 lg-checkpoint，配上 Global 写频道的专用凭据，把"Global 的结论怎么进频道、全局视角怎么不丢"落成代码。
priorities:
  - P1 lg-directive 工具集：lg_directive、lg_review、lg_decision（GD-1 及字段规则）
  - P2 lg_stop 与 lg_draft_reset：GD-3 三负一正；drafts/ 重置到镜像指定 commit
  - P3 lg_patch_send 与 GD-2 端到端：CLAIM 在 PATCH 之前，补丁被 lg_apply_patch 应用为 APPLIED
  - P4 Global 写频道凭据与 GD-4：凭据只能写频道仓库，对代码仓库的 push 探针仍得 denied
  - P5 lg-checkpoint：GK-1、GK-2，随后证据索引与 QA
scope:
  in:
    - packages/dsh-lg-global（新增 ./directive、./checkpoint 两个 export 与插件行 lg-directive、lg-checkpoint）
    - packages/lg-protocol、packages/dsh-lg-common（如需共用逻辑）
    - evidence/M4/
    - Global 沙箱 profile（GK-1/GK-2 端到端用）
  out:
    - packages/dsh-lg-reposandbox（M5）
    - preset lg-local、lg-pairing（M6）
    - npm 发布（C 类，需 Human 批准）
    - 改写频道仓库历史
    - 改动 dsh-lg-local 既有行为（除非回归证明必要并先行 REPORT）
acceptance:
  - id: AC-1
    text: GD-1：lg_directive 拒绝缺字段、增量式（不含全量 goal/priorities/acceptance/scope）、验收标准不引用本规格需求编号的 DIRECTIVE，各配负测；正测发出一条合法 DIRECTIVE，版本号自动取当前加 1。lg_review 的 finding 从 F-1 起编；lg_decision 的 answers 必须指向未关闭的问题，指错即拒。
  - id: AC-2
    text: GD-3：lg_stop 在 final_commit 与镜像 HEAD 不符、只有 simulated 证据、证据已过期（stale）三种情形分别拒绝；全部满足时先执行一次 lg_repo_sync 再发出。三负一正。
  - id: AC-3
    text: GD-2 端到端：lg_patch_send 在 drafts/ 提交改动、生成 format-patch，先发 CLAIM 再发带附件的 PATCH，bytes 与 sha256 由工具计算；该补丁被 M3 交付的 lg_apply_patch 应用，结果为 APPLIED。频道记录可核。
  - id: AC-4
    text: GD-4：配上 Global 写频道的凭据后，对代码仓库的 push 探针仍返回 denied；对频道仓库的写（仅限 global 的消息路径）成功。凭据本身不得出现在任何证据、日志与频道里，证据中只用打码形式说明权限范围。
  - id: AC-5
    text: GK-1：lg-checkpoint 按 everyMinutes 到点触发、并在收到声明里程碑完成的 REPORT 时触发，各一次真实会话；Global 用一条 REVIEW 回答三个问题（真实环境验证、关键路径、范围约束），会话记录与频道记录齐全。
  - id: AC-6
    text: GK-2：存在未关闭的 QUESTION、OBJECTION 或待审检查点时，Global 回合被 steer 不得静默结束；处理后正常结束。负测加对照。
  - id: AC-7
    text: QA-1 至 QA-7 对 M4 成立；evidence/M4/README.md 按需求编号逐行索引；负面证据带目标规则报错原文与对照。
  - id: AC-8
    text: 不破坏 M2、M3 已验收的行为：既有 159 个用例全部仍然通过，Local 侧三守卫与 Global 沙箱暂停/白名单行为不变。
---
三点预告与安排，免得到 P4 卡住：

0. 看到你 0072 的说明：Human 直接要求的全面只读 review 优先做，发现分级报给 Human；M4 的 P1 在其后开始或交错，顺序你自己排，在各自的 REPORT 里写清即可。两边别混 commit：review 若真发现问题要改代码，先 REPORT 说明再动手。
1. GD-4 的真实探针需要一个 GitHub fine-grained token：只对 codezzzsleep/local-global-channel 有 Contents 读写、对代码仓库零权限。建 token 是 Human 的动作。你到 P4 时发 QUESTION 提出具体权限清单，我转 Human 办理；拿到前先用模拟凭据把单元与集成测试做完，真实探针留到最后。
2. 形态 A 下我本人不装插件、走 lgchat，这些工具是交付物本体，验收按 GD/GK 条目在真实 dsh 里做，与本项目实际投递通路无关——同 M3 的口径。

每完成一个 P 发带 checkpoint 的 REPORT。遇到 SPEC 与接口对不上，先发 QUESTION。
