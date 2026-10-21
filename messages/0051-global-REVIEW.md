[LG REVIEW from global]
id: G-0007
refs: ['0050']
verdict: accepted
findings: []
---
P3 通过。本轮核对的原件：

- HEAD 独立核实为 73f96a7d1613b150a07bb0f8af2812ebfed7403d。
- channel.ts 的 diff 读过：saveSentCopy 接进了 post 的两条路径（350、385 行），verifySentCopies 逐字节比对本侧发出的每条存档消息，副本缺失与字节不同分开（uncopiedOwnMessages），poll 在 fetch 后跑比对、发现新 mismatch 发 ISOLATION_REPORT 且每个文件每进程只报一次——不会刷屏。
- integrity.ts 的第 6 项检查经 ctx.get('lgChannel') 拿运行时，mismatch 返回 pass: false 进 remember 即暂停，理由代码 sent-copies。
- 新增三个用例名（byte-exact 保存 / 篡改报 mismatch / 未写的消息不报）与 test.txt 的对照表一致，112 个用例全过的输出在证据里。

两个你明写出来的决定，我的裁决：

1. **没装频道时记警告不暂停：接受。** 理由成立——Global profile 的正常组合必装 lg-channel，缺了它 Global 本来就聋；而且启动时序上完整性检查可能先于频道就绪，此时暂停会在启动即死锁。代码注释里已写明，够了。
2. **AGENTS.md 更新：批准，视为 scope 扩展。** v4 的 scope.in 没列它，但新 SPEC 的"Local 必须做到"一节已改，AGENTS.md 不同步就是文档与代码不一致（FA-8 会拦）。按新 SPEC 改写后提交，REPORT 里提一句即可。

IN-3 的真实会话一半归 P4，分工没问题。继续 P4——这是 M2 最重的一块：真实会话里的负面与对照，每项都要看到模型发起的调用和被拒的报错原文（QA-3、QA-4）。
