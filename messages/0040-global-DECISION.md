[LG DECISION from global]
id: G-0001
refs: ['0037', '0039']
answers: ['0037']
choice: 四个选项都不选，按 SPEC v2 已定方案执行：先用 lg-repo 自己的 git 环境分别跑 ls-remote 和 push --dry-run 定位挂住的原因；若 ls-remote 也挂住，原因就是精简环境丢掉了代理设置，用 lg-repo 新增的 httpProxy 配置解决。inconclusive 改为只警告不暂停；RP-6 验收仍要求在本机实测得到 denied。
reason: SPEC v2 的"当前状态与交接"与 lg-repo、RP-6 两节已定案，本 DECISION 只是把 SPEC 的结论落到 0037 上。
---
0037 关闭。四个选项没有一个与 SPEC v2 一致，逐一说清：

1. 不选。SPEC 已定："push 探针查不出结果时，只警告不暂停"。inconclusive 触发暂停这一条要改掉（repo.ts 目前是 verdict !== 'denied' 就暂停），它属于 DIRECTIVE v4 的修改项。
2. 不选。file:// 的写权限来自文件系统而非凭据，同一用户下探针必然判 writable，此路在 M2 不通，保持 https。
3. 不选。与 SPEC "Global 的 git 不带任何凭据" 冲突。
4. 不选。探针不是辅助，RP-6 要求本机实测得到 denied，配置断言代替不了实测。

执行顺序：

1. 用 lg-repo 自己的 git 环境（白名单环境变量、-c credential.helper= 那一套），分别跑 ls-remote 和 push --dry-run，两条命令的原始输出都留下来。0037 只说"同一环境"挂住，没有分清是精简环境下的行为还是默认环境下的行为，先把这个分清。
2. 如果 ls-remote 也挂住，原因就是精简环境丢掉了代理设置：lg-repo 增加 httpProxy 配置（SPEC 的配置表里已有这一项），配上之后重跑探针。
3. 如果 ls-remote 秒回、只有 push 挂住，先确认挂上 httpProxy 后 POST 能否通过；仍不通就发 QUESTION 并附两条命令的原始输出，不自行降级。
4. 规则改完后的落点：writable 暂停；denied 通过；inconclusive 只在 lg_integrity_check 里显示为警告，不触发暂停。但 RP-6 的验收标准不变：本机实测得到 denied，对照是 writable 触发暂停；inconclusive 不算通过。
