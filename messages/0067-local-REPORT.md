[LG REPORT from local]
id: L-0017
refs: ['0066']
commit: e042ec5b83546346eb4749dfb01aeac677d30a34
directive_version: 5
evidence:
  - evidence/M3/ev-real/manifest.json
  - evidence/M3/ev-real/raw/command-output.txt
  - evidence/M3/ev-leaky/manifest.json
  - evidence/M3/ev4-stale-session.jsonl
  - packages/dsh-lg-common/test/evidence.test.ts
  - packages/dsh-lg-common/test/channel-runtime.test.ts
negative_tests:
  - EV-1 raw 文件不存在即拒绝
  - EV-2 只有模型提供的文件时不能标 real（real 必须含工具亲自执行产生的原始文件）
  - EV-4 digest 变化后清单标 stale，且引用它的 REPORT 被 lg_post 拒绝
  - EV-5 含 token= 的 raw 文件写不进清单，打码后成功
checkpoint: true
meanwhile:
  - P3：lg-patch-apply（PA-1 四个用例）
---
## P2 完成：lg-evidence

### 先回答你的问题：`lg_evidence_session` 怎么摆

**移进 `dsh-lg-common`，两侧共用一份实现。** 理由：

1. 要求是"不重复实现两份逻辑、EV-3 口径一致"。放进 common 是唯一能**结构上保证**这一点的摆法——两侧只是把同一个插件行装进自己的组合，不存在两份代码需要人盯着同步。
2. 它是**两侧都要用的能力**：Global 沙箱要导出自己的会话（M2 的 IN-2 就依赖它），Local 侧也要（EV-3）。
3. 代价是它原来借用 `dsh-lg-global/scope` 的两个小工具（`sessionCwd`、`isInside`）。common 不能依赖 global（方向反了），所以我把这十几行**内联**进 common 的 evidence 里并写明原因——重复的是两个小工具，不是导出逻辑。
4. SPEC 把 lg-evidence 归 dsh-lg-local；我按"能力共用、交付物归属"处理：`dsh-lg-local` 是它的交付文档与测试所在地，实现放在 common。

### EV 逐条

| 编号 | 结果 | 证据 |
| --- | --- | --- |
| EV-1 | 清单绑定当前 commit 与 digest；raw 不存在即拒绝 | `evidence/M3/ev-real/manifest.json` |
| EV-2 | `environment: real` 时**工具亲自执行 command**，输出写成原始文件并记录命令、退出码、起止时间、主机名、dsh 版本、sha256；没有它就不能标 real | 清单里 `producedByTool: true` |
| EV-3 | 导出与会话日志**逐字节一致**（sha256 对比） | `packages/dsh-lg-common/test/evidence.test.ts`：对真实 store 布局（zstd 压缩 + 明文两种）比对 `artifact.jsonl` 与原文 |
| EV-4 | digest 变化 → 清单标 stale；引用它的 REPORT 被 `lg_post` 拒绝 | 真实运行：提交后 digest 变 `be5345a1…`，检查报 `STALE M3/ev-leaky | STALE M3/ev-real`；拒绝对话在 `channel-runtime.test.ts` |
| EV-5 | 含 `token=` 的 raw 写不进清单；打码后成功 | 真实运行两条原文：`Error: … contains token=[A-Za-z0-9_\-]{10,}; redact it…` 与打码后的 manifest |

### 过程中修掉的两个"静默漏掉"

1. **清单只扫一层目录**：`lg_evidence_check` 原来的 `readdirSync(evidenceRoot)` 只到第一层，而 topic 可以是嵌套的（`M3/ev-real`），于是它报 "none"、实际清单就在下面一层。已改成递归。
2. **引用 topic 的提取同样只取第一段**：`path.split('evidence/')[1].split('/')[0]` 对嵌套 topic 得到 `M3` 而不是 `M3/ev-real`，于是 stale 检查**静默匹配不上**。已改成剥掉 `manifest.json` 取整段。

两条都是"看起来在工作、实际什么都没查"的类型，是新写的负测把它们逼出来的。

### 验证

**138 个用例全过**（lg-protocol 72、dsh-lg-global 40、dsh-lg-common 20、dsh-lg-local 6）。提交见字段区。

## meanwhile

P3：lg-patch-apply 的 PA-1 四个用例（APPLIED / REJECTED / DUPLICATE / CONFLICT 且工作区已回滚）。
