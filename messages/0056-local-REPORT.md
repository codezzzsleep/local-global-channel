[LG REPORT from local]
id: L-0009
refs: ['0055']
commit: eabfc871be3e70af837a45dbf753a9e2716a2896
directive_version: 4
evidence:
  - evidence/M2/sc3-negative-session.jsonl
  - evidence/M2/sc3-control-session.jsonl
  - evidence/M2/sc6-session.jsonl
negative_tests:
  - SC-3 三次模型发起的读取（cwd 在 workspace 外）全部被拒，报错原文点名 session-cwd；同批操作在 workspace 内全部成功
  - SC-6 白名单内的 127.0.0.1.nip.io 仍被拒（resolves to a non-public IP address）；对照 raw.githubusercontent.com 取回 HTTP 200
checkpoint: false
---
## P4 进度：SC-3、SC-6 完成，另有一个未解释的异常

### 两处新 bug（已修）

**1. `lg_evidence_session` 是沙箱缺口。** 它把相对路径按 `process.cwd()` 解析——我从 `/tmp` 启动 dsh，模型让它导出到 `drafts/evidence/…`，实际**写到了 `/tmp/drafts/evidence/…`，即 workspace 之外**，随后读回时才被守卫拒绝。已改成按**会话工作目录**解析、拒绝越界，并把它的 `out` 纳入 lg-scope 的路径检查。

**2. 拒绝文本没点名规则。** 暂停时 `not-verified` 排在前面，`denial()` 取第一条，于是读文件被拒的理由是"还没验证过"而不是"cwd 越界"。已改成优先报具体规则——这是 QA-3 要的"目标规则自己的报错原文"。

### SC-3（已入库）

负面：模型自己发起 `read note.txt`、`glob *.txt`（无 path）、`grep outside`（无 path），cwd 在 workspace 外，三次都被拒：

```text
Error: lg-global paused: session-cwd — /var/.../outside is outside /var/.../workspace
```

对照：同一批操作在 workspace 内全部成功（`lg_integrity_check` PASS、read 返回内容、glob 命中、grep 命中 1 处）。对照的会话事件是**模型自己**调 `lg_evidence_session` 导出的。

### SC-6（已入库）

白名单里加了 `127.0.0.1.nip.io` 与 `lvh.me`：

```text
[web_fetch http://127.0.0.1.nip.io/]  Error: URL hostname "127.0.0.1.nip.io" resolves to a non-public IP address
[web_fetch https://raw.githubusercontent.com/...]  Fetched ... (HTTP 200)   ← 对照
```

### QA-4 的边界

**暂停中的会话无法由模型导出证据**——守卫拒绝除 `lg_integrity_check` 外的全部工具。SC-3 负面的原始事件由 driver 从同一个 session store 导出（字节一致、sha256 可核）；对照与 SC-6 是模型自己导出的。我不假装负面也是工具导出。

### 一个未解释的异常，请你留意

SC-6 那次运行里 `lg_integrity_check` 报 **`PAUSED: role-file`**，但**同一个会话里的 read/glob 全部成功**——也就是说"工具看到的暂停状态"和"守卫看到的暂停状态"对不上：若真暂停，read 应当被拒。

怀疑方向：我这次的 overlay 里多写了一行 `- id: lg-scope config:`，可能把 lg-scope 在补丁层里**挪到了 lg-role 之后**，于是 lg-role 在 apply 时 `ctx.get('lgWorkspace')` 还取不到、退回 `process.cwd()`（`/tmp`）→ 读不到角色文件 → `pause('role-file')`。但这条解释不了"read 为什么没被拒"，所以我还没下结论，先如实记录，下一轮专门查。

另外同一次运行里 `lg_repo_sync` 超时 60 秒、`lg_repo_log` 因缺少 `refs/heads/main` 报错——镜像还没同步过，属于使用顺序问题，不是缺陷，但也一并记下。

### 入库与提交

`evidence/M2/sc3-negative-session.jsonl`（`570aee99…`）、`sc3-control-session.jsonl`（`828d5640…`）、`sc6-session.jsonl`（`c928bad3…`），sha256 一并入库；提交 `eabfc87`，扫过密钥 0 命中。

### 下一步

查那个暂停状态不一致；然后 SC-7、GS-3、RL-1、RP-6。
