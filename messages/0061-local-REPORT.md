[LG REPORT from local]
id: L-0014
refs: ['0041']
commit: 7f84849e49df983b2c9cc2e6794b6e7518bd37ea
directive_version: 4
evidence:
  - evidence/M2/README.md
  - evidence/M2/dsh-interfaces.md
  - evidence/M2/test.txt
  - evidence/M2/secret-scan.txt
negative_tests:
  - 见 README.md 的需求索引：GS-1..3、PS-1..2、RL-1..2、SC-1..8、RP-1..6、IN-1..3、CH-12 每项都列出报错原文与对照
checkpoint: true
meanwhile: []
---
## P5 完成，M2 里程碑自评达到验收条件

提交 `7f84849`，验收时 HEAD 即此提交。

### 这一轮交出的

| 产物 | 内容 |
| --- | --- |
| `evidence/M2/README.md` | AC-1…AC-8 对照 + **全部需求编号**（GS-1..3、PS-1..2、RL-1..2、SC-1..8、RP-1..6、IN-1..3、CH-12）逐行：证据文件、命令、环境、结果 |
| `evidence/M2/dsh-interfaces.md` | 17 条 dsh 接口的源码路径与行号（QA-7） |
| `evidence/M2/test.txt` | QA-1 重跑：install 0 个 WARN/ERR，**115 个用例全过**（61 + 14 + 40） |
| `evidence/M2/secret-scan.txt` | QA-6 重跑：密钥无命中、`token=` 只剩 `<REDACTED>`、公网部署地址无命中 |
| `AGENTS.md` | 按新 SPEC 的"Local 必须做到"重写，并改为 lgchat 流程（你批准的 scope 扩展） |

### AC 对照（详见 README）

AC-1 PASS（`0b39ffcb…` 逐字节，`cmp` 无差异）；AC-2 PASS（本机实测 `denied`、可写仓库 `writable` → 暂停、三态单元测试）；AC-3 PASS（`httpProxy` → `-c http.proxy=` 进入每个 git 调用 + 解析用例）；AC-4 PASS（`archived bytes differ from the saved copy (copy 866B, archive 880B)`）；AC-5 PASS；AC-6 PASS；AC-7 PASS；AC-8 PASS。

### 两处**此前未实现**、本轮补齐

1. **RP-5**：SPEC 要求核对镜像自己的 git 配置，实现里完全没有 → 补 `foreignMirrorKeys()` / `ALLOWED_MIRROR_KEYS`，接进 `ensure()`。
2. **IN-2**：SPEC 要求启动自检经频道内部接口发 `ISOLATION_REPORT`，而 `ISOLATION_REPORT` 在 lg-global 里一次都没出现过 → 补 `ChannelRuntime.reportIsolation()` 与启动调用。

### 三处必须说清的边界

1. **QA-4 的例外**：暂停中的会话导不出证据（守卫拒绝一切工具，只留 `lg_integrity_check`）。这类负面证据由 driver 从同一个 session store 导出，sha256 可核；**对照与未暂停的会话是模型自己调 `lg_evidence_session` 导出的**。README 里逐条标注，不假装负面也是工具导出。
2. **配置默认值偏差**：SPEC 配置表写 `stateDir` 默认 `$DSH_HOME/lg/state/<仓库标识>`，实现是 `$DSH_HOME/lg`。改默认值会让已绑定会话的游标搬家（要重新 `/lg-bind`），我**没有动它**，在 README 里登记，留给 M3 一起处理。
3. **headless 不是可信载体**：`dsh-headless` 直接经核心注册表建 Agent、绕过 preset 作用域，会话会看到根工具集。SC/GS/IN 这些条款验的就是 preset 作用域下的会话，所以全部验收都在 web profile 里做。

### 过程里修掉的真 bug（都已入库并附证据）

解除暂停的死锁（PS-2 原本实现不出来）、`lg-role` 按进程工作目录解析角色文件、`lg_evidence_session` 的沙箱缺口（写到了 workspace 之外）、`lg_*` 工具的白名单规则、拒绝文本不点名规则。

### meanwhile

M2 到此收尾，我等你逐条核对索引后以 REVIEW 宣布验收。在收到你的消息之前只做 `meanwhile` 里声明的工作——本轮为空，所以我停在 `wait` 上。
