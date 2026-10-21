[LG REPORT from local]
id: L-0013
refs: ['0059']
commit: 3a384b98b3aa7985f18d4c7a423cebe96c01dc27
directive_version: 4
evidence:
  - evidence/M2/rl2-session.jsonl
  - evidence/M2/rp1-session.jsonl
  - evidence/M2/rp2-integration.txt
  - evidence/M2/rp5-session.jsonl
negative_tests:
  - RL-2 角色文件不在时 role-file FAIL，read 被拒（ENOENT 原文）
  - RP-5 镜像配置里塞入 http.extraheader 后 mirror-config 暂停，lg_repo_sync 与 read 都被拒
  - RP-2 镜像被指到远端没有的提交后，下一次 sync 撤销回上游
checkpoint: false
---
## 又完成四项：RL-2、RP-1、RP-2、RP-5

### RL-2

暂时移走 `LG-ROLE.md`：`FAIL role-file: cannot read …/LG-ROLE.md: ENOENT`，read 被拒，理由是 `lg-global paused: role-file — cannot read …`。跑完立刻还原。

### RP-1

`lg_repo_log` 返回真实提交、`lg_repo_show` 返回 `AGENTS.md` 原文；同步等值的证据取自 RL-1 那次运行（`mirror 2a93a3795dd4 / remote 2a93a3795dd4 (equal)`）。单独重跑时 `git fetch` 又撞过 60 秒超时（网络抖动），既有证据不受影响。

### RP-2（真实 git + 真实文件系统）

```text
after sync 1:  36af416f… 36af416f… (equal)
after tamper:  66f6539e… (moved)
after sync 2:  36af416f… (reverted to upstream)
```

### RP-5：这一项此前**根本没实现**，本轮补上

SPEC 要求核对镜像自己的 git 配置只含插件写入的键（`core.*`、`remote.origin.*`、可选 `http.proxy`），启动时与每次同步时都查。我加了 `foreignMirrorKeys()` 与 `ALLOWED_MIRROR_KEYS`，接进 `ensure()`：

```text
PASS 之外多了一条：PAUSED: mirror-config
[lg_repo_sync]  Error: lg-global paused: mirror-config — mirror config holds keys the plugin did not write: http.extraheader
[read …]        同上
```

（过程里踩了自己一个坑：第一次补丁的锚点字符串早被之前的 `proxyArgs` 改动改掉了，`str.replace` 静默失败，检查根本没进构建——所以我先怀疑"为什么没暂停"，查完是补丁没生效。）

### 入库与提交

`rl2-session.jsonl`、`rp1-session.jsonl`、`rp2-integration.txt`、`rp5-session.jsonl`（`b04db6ae…`），新增 RP-5 单元测试后 **40 个用例全过**，提交见字段区。密钥扫描 0 命中。

### AC-6 剩余

IN-1（白名单外工具 → 首次调用前暂停）、IN-2（启动发 ISOLATION_REPORT）、IN-3（篡改已存档 global 消息 → 暂停）。IN-2/IN-3 需要把 lg-channel 装进 profile。之后是 P5 索引、dsh-interfaces.md、QA-1/QA-6 重跑与 AGENTS.md 改写。
