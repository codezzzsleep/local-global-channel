[LG REPORT from local]
id: L-0010
refs: ['0056']
commit: 441474bb8cae882cc3b4b9df3f3dd5d6c6a36e4b
directive_version: 4
evidence:
  - evidence/M2/sc6-session.jsonl
negative_tests:
  - SC-6 白名单内的 127.0.0.1.nip.io 仍被拒，报错原文点名 lg-scope；对照 raw.githubusercontent.com 取回 HTTP 200
checkpoint: false
---
## 上一条里那个"未解释的异常"结案了：是我自己的 overlay 写坏的

L-0009 里我报告 SC-6 那次运行报 `PAUSED: role-file` 却又能读文件。结论：

**根因在我的 overlay。** dsh 的补丁行只要是 `id` + `config`，就会**替换**该条目的整份配置，不是合并。我为了给 lg-scope 加 `allowDomains` 写了：

```yaml
- id: lg-scope
  config:
    allowDomains: [127.0.0.1.nip.io, lvh.me, raw.githubusercontent.com]
```

结果把 `workspace` 一起抹掉了 → lg-scope 抛 `"workspace" is required` 未激活 → `lgWorkspace` 服务不存在 → lg-role 退回 `process.cwd()`（`/tmp`）→ 读不到角色文件 → `pause('role-file')`。overlay 里补回 `workspace` 之后，`lg_integrity_check` 直接 **PASS**。

## 顺带两处加固

1. **角色文件变成一项可复核的检查**，不再是一按就锁死。原来只要挂载时读不到，`role-file` 就永久留在暂停理由里，没有任何检查能清掉它。现在 lg-integrity 会重新核可读性、空文件也算失败——文件后补上就自动解除，中途消失就重新暂停（RL-2 的两面都成立）。
   - 实现上有个坑记一下：lg-role 在 preset 里，**不能在 preset 作用域里发布服务**（`Preset services require isolate realms`），所以路径由 lg-integrity 按同样的规则自行推导，而不是靠服务传递。
2. `lg_evidence_session` 已在 preset 内注册，SC-6 的证据是**模型自己**导出的：`exported 65 events (77454 bytes, sha256 7c6922920fde…)`。

## SC-6 干净证据

```text
[lg_integrity_check]                    PASS（含 ok role-file: …/workspace/LG-ROLE.md）
[web_fetch http://127.0.0.1.nip.io/]    Error: lg-scope: web_fetch resolved 127.0.0.1.nip.io to a private address: 127.0.0.1
[web_fetch https://raw.githubusercontent.com/…]  Fetched … (HTTP 200)   ← 对照
```

（中间有一次重跑撞上 30 秒工具超时，与规则无关，已重跑取到完整三样。）

证据 `evidence/M2/sc6-session.jsonl` sha256 `7c692292…`，提交 `441474b`。

### 下一步

SC-7（测试插件 prepend 直接 allow 仍被拒）、GS-3（默认 preset 被改回 standard）、RL-1（金丝雀）、RP-6（`denied` 实测 + `writable` 对照）。
