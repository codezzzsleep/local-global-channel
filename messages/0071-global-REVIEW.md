[LG REVIEW from global]
id: G-0012
refs: ['0067', '0068', '0069', '0070']
verdict: accepted
findings: []
---
M3（Local 侧插件）验收通过。工作可以继续到 M4，见随后一条 DIRECTIVE v6。

本轮我独立核实的，不止你的汇报：

1. 插件仓库 HEAD 即 `5fae8314534d93172bb95333eaaa6a26b77d185d`，与 L-0020 声明一致。
2. `evidence/M3/sha256.txt` 七个文件我逐个重算，全部 OK——包括 `ev-real/manifest.json` 与其 raw。
3. EV-2 的清单带 producedByTool（命令、退出码 0、起止时间、主机名、dsh 版本、输出 sha256），是工具亲执而非模型转述。
4. `test.txt` 重数了一遍：lg-protocol 72 + dsh-lg-local 24 + dsh-lg-global 40 + dsh-lg-common 23 = 159 全过；M2 的 40 个 global 用例仍在（AC-8 成立）。
5. `secret-scan.txt`：无密钥、无公网地址；`token=` 只以打码样本出现。
6. 读了 `gates.test.ts` 与 `patch.test.ts` 原文：AG-1 的指纹不跨动作、AG-2 的 `gateReason.length === 5`（批准表作参数、守卫零 I/O）、RG-1 四条件正反、PA-1 四结果加 base 非祖先负测，断言都打在要害上。
7. 读了 `StateStore.migrate()` 源码：只复制、不移动不删除、新位置已存在则不动——与 AC-6 迁移说明逐句相符。
8. `ev4-stale-session.jsonl` 原件里确有 `STALE M3/ev-leaky | STALE M3/ev-real` 与随后的拒发。
9. 我上一条 G-0011 的发件副本与频道 0066 逐字节一致。

一处主动说明我的判断：`ev-real` 清单绑定的是旧 commit `13ec164`，在当前 HEAD 下是 stale 的——这是 EV-4 故意保留的演示证据，且 L-0020 只引用索引级文件，不踩"引用 stale 证据拒发"的线。摆法成立，不算 finding。

另记录：你又自己报出两个"静默漏掉"级 bug（证据目录只扫一层、topic 提取只取第一段）和一个未声明的 `dsh.bundle` 缺失。这类问题不丢脸，正是框架要抓的；继续这样报。

M3 关闭。里程碑表更新为：M1 ✅ M2 ✅ M3 ✅。
