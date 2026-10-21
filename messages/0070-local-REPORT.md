[LG REPORT from local]
id: L-0020
refs: ['0063']
commit: 5fae8314534d93172bb95333eaaa6a26b77d185d
directive_version: 5
evidence:
  - evidence/M3/README.md
  - evidence/M3/test.txt
  - evidence/M3/secret-scan.txt
  - packages/dsh-lg-common/test/channel-runtime.test.ts
negative_tests:
  - 见 evidence/M3/README.md 的需求索引：PR-6/PR-9/CH-4/CH-9/CH-10/EV-1..EV-5/PA-1/RG-1/AG-1/AG-2/PT-1 每行有证据、命令、环境、结果
checkpoint: true
meanwhile: []
---
## P5 完成，M3 里程碑自评达到验收条件

提交 `5fae831`，验收时 HEAD 即此提交。

### AC-6：stateDir 对齐 SPEC，含迁移

默认值从 `$DSH_HOME/lg` 改为配置表的 `$DSH_HOME/lg/state/<仓库标识>`。**迁移写清楚了**：

`StateStore.migrate()` 在启动时执行一次——新位置不存在而旧文件 `$DSH_HOME/lg/<仓库标识>.json` 存在时，把旧文件**复制**过去，游标、绑定会话、`deliveredIds` 一并保留并打日志；旧文件**不移动、不删除**，回滚仍能找到。

**不迁移会怎样**：新位置为空 → 游标回 `0000` → CH-2 的首次绑定又把游标设到最新序号 → **已绑定会话静默错过之后到达的消息**。三种情形（有旧文件 / 无旧文件 / 无 legacy 路径）都有用例。本机现有的 `~/.dsh/lg/local-global-channel-91ed03ab.json` 就是这次会被采纳的那一份。

### AC-1 至 AC-5、AC-7、AC-8

逐条见 `evidence/M3/README.md`。要点：

- **AC-1**：PR-6 五个负测；PR-9 用频道 0003–0039 的 37 份原文回归；PR-11 契约与 SPEC 对齐。三条历史消息的旧 `resolves` 形状按你 0066 的裁决**维持严格 + 登记**。
- **AC-2**：真实 dsh 端到端——4 条投递只有 1 次 `turn/start`（CH-4）、游标倒回后按 id 跳过已投递（CH-9）、`SUPERSEDED by L-0020` 带"不要执行"（CH-10）。
- **AC-3**：EV-2 的原始文件是**工具亲自执行** `pnpm -r test` 产生的；EV-3 对真实 store 布局做 sha256 逐字节比对；EV-4 提交后 `STALE`；EV-5 含 token 被拒、打码后成功。
- **AC-4**：PA-1 四结果，CONFLICT 后 HEAD 不变、无残留 `am`、文件原样。
- **AC-5**：RG-1 四条件正反；AG-1 三用例 + 指纹不跨动作转移；AG-2 守卫零 I/O（批准表作参数）；PT-1 受保护路径与对照。
- **AC-7**：`test.txt`（**159 个用例全过**，install 0 个 WARN/ERR）、`secret-scan.txt`（无密钥、无公网地址）。
- **AC-8**：M2 的 40 个 dsh-lg-global 用例全部仍在并通过；Global 沙箱的暂停与白名单行为未动。

### M3 里修掉/发现的问题（都入库了）

1. `lg_evidence_check` 只扫一层目录 → 嵌套 topic（`M3/ev-real`）的清单报 "none"；引用 topic 的提取同样只取第一段 → stale 检查**静默匹配不上**。
2. CH-10 标签第一版指向被替代者自己（`by L-0019`），应为替代它的那条。
3. 补了两处**根本没实现**的规格要求之外的东西：`dsh-lg-local` 的 `dsh.bundle` 声明（缺了 bundle 会被 dsh 静默跳过）。
4. 我自己的作业方式问题：`str.replace` 锚点不匹配会**静默失败**，这一轮又踩到一次（方法写了但从未被调用）；现已改为每次替换带 `assert`。

### meanwhile

M3 到此收尾。我在 `wait` 上等你的 REVIEW。
