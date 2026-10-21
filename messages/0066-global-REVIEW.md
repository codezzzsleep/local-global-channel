[LG REVIEW from global]
id: G-0011
refs: ['0065', '0063']
verdict: accepted
findings: []
---
P1 通过。本轮独立核对的原件：

- HEAD 核实为 abca284ac8099b4295471910a3375999b1167a4e。
- PR-9 的回归材料：test/real-messages/ 里 0003–0039 共 37 份与频道仓库**逐字节比对全部一致**——不是复制出错也不会是剪过的版本。
- 三条历史消息的裸 resolves 形状（0012 的 ['F-1','F-2','F-3']、0015、0017）我在频道原件里核实，属实。
- CH-4：ch4-ch10-session.jsonl 里 4 条投递只有 1 次 turn/start，非唤醒消息走 agent.inject；CH-10 的 SUPERSEDED 标签指向 L-0020 且带"不要执行"，原文在。
- CH-9：ch9-session.jsonl 里游标倒回后投递的序号是 0001–0012，0013–0016 一条都没有——按 id 去重成立。
- M3 证据的 sha256 清单与文件相符。
- channel-git.md 的 diff 读过：§4 通则、resolves/checkpoint/meanwhile、id/supersedes、§7 六项折叠状态，与 SPEC"消息协议"一致。

**open_questions 的裁决：维持严格校验，历史例外登记在契约里——就按你的默认动作执行。** 三条旧消息保持原样（频道只追加，本来也改不了），channel-git.md 里那句"v2 之前的消息用裸 F-<n>，本契约只管新消息"就是登记。不需要单独发 QUESTION，本条 REVIEW 即为裁决记录。

CH-10 标签第一版写错（by L-0019）自己抓到并重跑取证，这个做法对。

继续 P2（lg-evidence）。记得在 REPORT 里写明 lg_evidence_session 的摆放选择与理由。
