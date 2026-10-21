[LG DIRECTIVE from global]
id: G-0010
refs: ['0061', '0062']
version: 5
goal: M3 协议补充与 Local 侧插件：lg-protocol 补齐 id/supersedes 等 v2 规则，docs/channel-git.md 与 SPEC 对齐；新建 dsh-lg-local（lg-evidence、lg-patch-apply、lg-report-gate、lg-action-gate、lg-protect），把"证据要原始、先批后做、不碰另外两个区"落成代码。
priorities:
  - P1 协议补充：PR-6、PR-9、PR-11，以及 CH-4、CH-9、CH-10 的投递行为
  - P2 lg-evidence：EV-1 至 EV-5
  - P3 lg-patch-apply：PA-1
  - P4 lg-report-gate、lg-action-gate、lg-protect：RG-1、AG-1、AG-2、PT-1
  - P5 stateDir 默认值对齐 SPEC 配置表（M2 登记的偏差），证据索引与 QA
scope:
  in:
    - packages/lg-protocol
    - packages/dsh-lg-common
    - packages/dsh-lg-local（新建）
    - docs/channel-git.md
    - evidence/M3/
    - Local 的日常 profile 与 Global 沙箱 profile（CH-4/CH-9/CH-10 端到端用）
  out:
    - packages/dsh-lg-reposandbox（M5）
    - lg-directive、lg-checkpoint（M4）
    - npm 发布（C 类，需 Human 批准）
    - 改写频道仓库历史
acceptance:
  - id: AC-1
    text: PR-6（id 与 supersedes 校验，三个负测）；PR-9 用本频道 0003 至 0039 的真实消息做回归，没有 id 的旧消息照常解析折叠；PR-11 的 docs/channel-git.md 与 SPEC"消息协议"一节一致，我审 diff。
  - id: AC-2
    text: CH-4（Global 侧只有唤醒类消息开新回合，正反各一）、CH-9（已投递过的 id 不再投递）、CH-10（被替代消息带 SUPERSEDED 标签投递且不执行），都在真实 dsh 端到端验证，证据是会话原始事件。
  - id: AC-3
    text: EV-1 至 EV-5：清单绑定当前 commit 与 digest；real 证据至少一份由工具亲自执行产生；lg_evidence_session 导出与会话日志逐字节一致（sha256 对比）；digest 变化后清单标 stale 且 REPORT 引用被拒；含 token 或密钥的 raw 文件写不进清单、打码后能写。
  - id: AC-4
    text: PA-1 四个用例：APPLIED、REJECTED（哈希不符）、DUPLICATE、CONFLICT 且工作区已回滚。
  - id: AC-5
    text: RG-1（有未报 commit 时回合被 steer 继续，已报时正常结束）；AG-1 三个用例（无批准拒、有批准执行一次、同一批准第二次被拒）；AG-2 守卫用内存批准表不现场读频道；PT-1 指向受保护路径被拒、项目文件正常。
  - id: AC-6
    text: stateDir 默认值对齐 SPEC 配置表（$DSH_HOME/lg/state/<仓库标识>），改动附迁移说明：已绑定会话的游标怎么处理写清楚。
  - id: AC-7
    text: QA-1 至 QA-7 对 M3 成立；evidence/M3/README.md 按需求编号逐行索引；负面证据带目标规则报错原文与对照。
  - id: AC-8
    text: 不破坏 M2 已验收的行为：既有 115 个用例全部仍然通过，Global 沙箱侧的暂停与白名单行为不变。
---
本条是 v4 的全量替换，M2 已按 0062 的 REVIEW 验收。

两点说明：

1. lg_evidence_session 现在在 dsh-lg-global 里，SPEC 把 lg-evidence 放在 dsh-lg-local。M3 实现时自己决定怎么摆（移进 common 共用、或两侧各留），要求只有一条：不重复实现两份逻辑，EV-3 的导出在两边的口径一致。在 REPORT 里写明你的选择和理由。
2. 本项目你的消息通路是 lgchat + export，但 dsh-lg-local 的插件是交付物本体，验收按 SPEC 的需求条目做，与本项目实际用不用它们投递无关。

每完成一个 P 发带 checkpoint 的 REPORT。遇到 SPEC 与接口对不上，先发 QUESTION。
