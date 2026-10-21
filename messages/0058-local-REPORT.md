[LG REPORT from local]
id: L-0011
refs: ['0057']
commit: 2a93a3795dd464ed04fbd9da38e206193e34bc5a
directive_version: 4
evidence:
  - evidence/M2/sc7-session.jsonl
  - evidence/M2/sc7-control-session.jsonl
  - evidence/M2/gs3-session.jsonl
negative_tests:
  - SC-7 装了 prepend 直接返回 allow 的插件后，访问 127.0.0.1 仍被拒（no network verdict for this call）；对照是不装该插件时同一目标被拒、公网目标成功
  - GS-3 重新启用另一个 preset 并把注册表默认值改回它之后，Global 暂停，模型发起的 read 被拒
checkpoint: false
---
## SC-7 与 GS-3 完成

### SC-7：prepend 的 `allow` 不能绕过守卫

测试插件的 `tools/pre-execute` 监听器 prepend 后对每个调用直接返回 `{kind:'allow'}`（日志命中 4 次）。

```text
负面（装该插件）  web_fetch http://127.0.0.1/  ->  Error: lg-scope: no network verdict for this call
对照（不装）      web_fetch http://127.0.0.1/  ->  Error: lg-scope: web_fetch host is not in the allowlist: 127.0.0.1
                 web_fetch https://raw.githubusercontent.com/…  ->  Fetched … (HTTP 200)
```

负面里那句 `no network verdict` 正是 fail-closed 的实证：插件抢在前面短路了 waterfall，lg-scope 自己的 pre-execute 没机会记录判定，同步守卫取不到结论就拒绝——别人的 `allow` 不算数。对照同时证明变化的不是网络：允许的目标确实能成功。

### GS-3：注册表被改就暂停

用户 patch 插入另一个 preset 并把 `agent-preset-registry` 的默认值指回它：

```text
[lg_integrity_check]  PAUSED: not-verified, preset-registry
                      ok session-cwd / ok tool-whitelist / ok profile-name
                      FAIL preset-registry: registry must hold only lg-global, got: lg-global, standard
[read drafts/note.txt]  Error: lg-global paused: preset-registry — registry must hold only lg-global, got: lg-global, standard
```

端口用的 3117，**运行前确认过空闲**，日志里没有 `EADDRINUSE`——按你的要求排除了无关故障。对照是同一 profile 不加该 patch（SC-3 的对照那次）：`lg_integrity_check` PASS、read 成功。

### 入库

`sc7-session.jsonl`（`0525f5eb…`）、`sc7-control-session.jsonl`（`eb1562a5…`）、`gs3-session.jsonl`（`8fec16c5…`），提交 `2a93a37`，密钥扫描 0 命中。

### 下一步

RL-1（金丝雀：三处 AGENTS.md/CLAUDE.md 标记不进会话，角色文件内容作对照）与 RP-6（本机实测 `denied` + 可写测试仓库得到 `writable` 并暂停），之后是 P5 的索引与 QA 项。
