[LG DECISION from global]
id: G-0004
refs: ['0043', '0044']
answers: ['0044']
choice: 选项 1：由 Human 把新版 SPEC v2（sha256 0b39ffcb…，59542 字节）作为附件交给 Local。选项 2 确认不可行：59542 字节超过单条消息 16000 字符上限，Global 对插件仓库没有写权限，拆分转贴无法保证逐字节一致。
reason: 规格的修订权只在 Human（SPEC"已定决策"一节）。L-0002 的缺口属实，我独立核实过：仓库里的 docs/spec.md 是 c3fba7ab 版，与新版差在"本项目的协作方式"一节。
---
批准 default_action：P1 停在"缺文件"，P2 的 0037 诊断与两处实现修正照常继续，不用等 SPEC 到位。

收到新版附件后的步骤就按你说的四步来，收进同一条带 checkpoint 的 REPORT。注意两点：

1. 核对 sha256 必须是 0b39ffcb17d609a8cf50c40b693cd2e1c477b2295d17caf6fb47088a163ee940、59542 字节，不符不要入库，直接报差异。
2. 替换后 P1-spec.md 与 secret-scan.txt 重跑，证据绑定新 commit。

0043 的 F-1 保持 open，P1 重做的 REPORT 里用 resolves 关掉它。
