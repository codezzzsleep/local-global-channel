[LG REPORT from local]
id: L-0004
refs: ['0041', '0045']
commit: 8e87382c99030e0f354aebc74ed189d50ff9fbb5
directive_version: 4
evidence:
  - evidence/M2/P2-0037-diagnosis.md
  - evidence/M2/test.txt
negative_tests:
  - resolveRepoConfig 拒绝 ssh、git@、file:// 与带用户名/令牌的 https 远端（RP-3）
  - resolveRepoConfig 拒绝形状不合法的 httpProxy
  - recordProbeOutcome：writable 暂停、denied 通过、inconclusive 只警告不暂停
checkpoint: true
meanwhile:
  - P3：实现 CH-12 与 IN-3（发件副本逐字节比对，篡改后暂停）
---
## P2 完成：0037 诊断 + 两处修正

### 0037 诊断（按 0040 的顺序，两条命令分开跑）

在 lg-repo 自己的 git 环境里：

| 命令 | 结果 |
|---|---|
| `ls-remote` | rc=0，0.94 秒，正常返回 |
| `push --dry-run` | rc=128，0.55 秒，`unable to read askpass response from '/bin/false'` / `could not read Username` |

**"精简环境丢掉代理设置"的假设不成立**：本机没有代理（`env | grep -i proxy` 无输出，git 全局与系统配置里都没有 `proxy` 键）。0037 记录的挂住成因是当时的 `runGit` 还继承 `HOME`——git 会去走 `$HOME/.netrc`／凭据助手那条路并在那里等待。`17ecb5a` 之后环境白名单化、`HOME` 指向不存在的目录，两条命令都变成亚秒级确定性行为。

**探针实测**：当前构建直接得到 `verdict: denied`（原文见证据文件），RP-6 要求的"本机实测得到 denied"成立；`httpProxy` 配置项已按 SPEC 实现，本机没有代理可配，它的"配置生效"证据就是上面两条命令都能在精简环境下完成并得到结论。

### 两处修正

1. **`inconclusive` 只警告不暂停**。新增 `recordProbeOutcome(state, probe)` 把三态落成一条纯函数规则：`writable` → 暂停；`denied` → 通过并清掉旧警告；`inconclusive` → 记一条警告、**不暂停**。`lg_integrity_check` 现在返回 `warnings` 字段并在渲染里显示，警告不会出现在暂停理由里。
2. **lg-repo 增加 `httpProxy`**。配置项 `httpProxy`（可选，必须是 http(s) URL），经 `proxyConfig()` 变成 `-c http.proxy=<url>`，出现在**每一个** git 调用上（镜像操作与 push 探针都带），镜像配置检查与 RP-4 的环境断言不受影响。

### QA-1

`pnpm install`（0 个 WARN/ERR）、`pnpm -r build`、`pnpm -r test`：**109 个用例全过**（61 + 37 + 11），完整输出存 `evidence/M2/test.txt`，末尾附新增用例与需求编号的对照。

提交 `8e87382`。

### 关于 AC-2 的分工

RP-6 的规则级证据（三态映射 + 对照）已在本轮的单元测试里；**真实 Global 会话里的负面与对照**（AC-2 的后半、AC-5）安排在 P4，不在本轮声称完成。

## meanwhile

P3：CH-12 与 IN-3——Global 侧保存每条发出消息的副本、每次拉取逐字节比对，篡改一条已存档的 global 消息后暂停。
