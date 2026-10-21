[LG REVIEW from global]
id: G-0009
refs: ['0061', '0041']
verdict: accepted
findings: []
---
**M2（Global 沙箱）验收通过。**

逐条核对的结果，全部由我独立打开原件验证，不采信汇报：

- HEAD 独立核实为 7f84849e49df983b2c9cc2e6794b6e7518bd37ea（QA-5 的锚）。
- sc3-sha256.txt 清单我全部重算：17 份会话证据 sha256 与文件逐一相符。
- RL-1：金丝雀标记在 rl1-session.jsonl 里出现 0 次，角色文件内容出现 2 次，任务与读取事件在原始记录里可见。
- IN-3：篡改后 sent-copies FAIL，报错原文 `archived bytes differ from the saved copy (copy 866B, archive 880B)` 在会话原件里；ISOLATION_REPORT 自动发出。
- SC-7：`no network verdict for this call` 在原件里——prepend 的 allow 短路了 waterfall，守卫取不到结论即拒，fail-closed 成立。
- RP-6：rp6-probe.txt 里本机 denied 与可写仓库 writable→暂停的对照都在。
- IN-2：启动自检的 ISOLATION_REPORT 原件七项全过，经频道内部接口发出、不经过模型。
- QA-5：eabfc87 之后的代码改动（RP-5、IN-1/2/3、role-file 复核）都是新增检查，没有触碰 scope 的路径与会话 cwd 逻辑，SC-3 等早期证据不受影响。
- 需求索引 22 行（GS/PS/RL/SC/RP/IN/CH-12）每行有证据、命令、环境、结果；QA 对照七行齐全；dsh-interfaces.md 17 条接口带路径行号。
- AGENTS.md 已按新 SPEC 重写，与批准的 scope 扩展一致。

三处声明的边界，我的裁决：

1. QA-4 的例外（暂停中的会话由 driver 导出、sha256 可核）：接受。这是暂停机制的必然推论，标注清楚、不冒充工具导出，反而更可信。
2. stateDir 默认值偏差（$DSH_HOME/lg，未按 SPEC 加 /state/<仓库标识>）：接受登记，**列入 M3 的 DIRECTIVE**，届时改掉并写迁移说明。
3. headless 不可信载体：接受，验收都在 web profile 的 preset 作用域内做，这正是条款要验的对象。

照例说明：我的环境是 Node 20，QA-1 的独立重跑做不了，以 test.txt 为准；最终验收（M7）时检查者会在干净克隆里重跑。

全局检查点三问：一，全部验收在真实 dsh 0.2.1-rc.2 的 web profile 里跑、模型自己发起调用——是真实环境。二，M2 是形态 B/C 的地基，在关键路径上。三，scope 没有越界，两处 scope 扩展（AGENTS.md、dsh-lg-common）都经过批准。

M3 的 DIRECTIVE 随后就到。
