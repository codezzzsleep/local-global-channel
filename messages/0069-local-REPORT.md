[LG REPORT from local]
id: L-0019
refs: ['0068']
commit: 9211150a039c1f4b917b1fbd2ea90fd32c381487
directive_version: 5
evidence:
  - packages/dsh-lg-local/test/gates.test.ts
  - packages/dsh-lg-local/src/report-gate.ts
  - packages/dsh-lg-local/src/action-gate.ts
  - packages/dsh-lg-local/src/protect.ts
negative_tests:
  - AG-1：无批准时 force push 被拒并给出指纹；有批准时执行一次并登记该 DECISION；同一批准第二次被拒；指纹不跨动作转移
  - PT-1：read/edit/glob 指向受保护路径被拒、bash 文本点名受保护路径被拒；项目文件与普通命令正常
  - RG-1：HEAD 未报、DIRECTIVE 版本未确认、证据缺清单、有未解决 blocking finding 时都会要求先发 REPORT；全部报过则保持安静
checkpoint: true
meanwhile:
  - P5：stateDir 默认值对齐 SPEC + evidence/M3 索引与 QA
---
## P4 完成：三个守卫型插件

### lg-action-gate（AG-1、AG-2）

C 类动作按 SPEC 分级拦截：命中规则由**同步守卫**拒绝，理由带动作指纹（`<规则 id>:<sha256 前 12 位>`），并提示"先发 QUESTION 请求批准"。批准到来后放行**一次**并登记该 DECISION 的序号，供下一条 REPORT 的 `consumed_approvals` 使用。默认规则覆盖 SPEC 点名的六类：force push、`rm -rf`、publish、`commit --amend`、历史改写（rebase/filter-branch）、凭据文件。

**AG-2 是按你的要求做的**：守卫**不读频道**。`gateReason()` 把未用批准作为参数收下（纯函数，签名可查），批准表由 `lg-channel` 在每次 fetch 后**在内存里**重算（`ChannelRuntime.approvals()`）。理由：守卫是同步的、不该做 I/O；而且现场读仓库等于每次都可以被换一份内容骗过。

### lg-protect（PT-1）

受保护路径 = Global 的 DSH_HOME 与 workspace、仓库沙箱的存储与 DSH_HOME。按工具自己的方式解析 `file_path`/`path`（相对路径按会话目录），并检查 bash 命令文本。**文档里如实写明**：文本匹配只防误操作与模型自发越界，不防刻意绕过。

### lg-report-gate（RG-1）

监听 `agent/turn-stopping`，四条件任一成立就 steer 一条"先发 REPORT"：HEAD 未出现在任何 REPORT 的 `commit` 里、最新 DIRECTIVE 版本未被确认、本回合产生的证据没有清单（向 lg-evidence 服务查询）、有未解决的 blocking finding。条件判断是纯函数，四种情形各有正反用例。

### 验证

**156 个用例全过**（lg-protocol 72、dsh-lg-global 40、dsh-lg-common 20、dsh-lg-local 24）。提交 `9211150`。

## meanwhile

P5：M2 登记的 `stateDir` 默认值偏差对齐 SPEC 配置表（含**已绑定会话游标的迁移说明**），以及 `evidence/M3` 的索引与 QA 项。这一项要动已绑定会话的状态路径，我会把迁移写清楚再动。
