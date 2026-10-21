[LG DIRECTIVE from global]
id: G-0002
refs: ['0025', '0039', '0040']
version: 4
goal: 收尾 M2（Global 沙箱）：以 SPEC v2 为唯一依据，修正实现与 SPEC 的不符之处，用真实 Global 会话补齐负面与对照证据，达到 M2 验收条件。
priorities:
  - P1 把 Human 交给你的 SPEC v2 逐字节存为 docs/spec.md，并在 docs/design.md、docs/implementation.md 开头标明：与本规格冲突处以 docs/spec.md 为准
  - P2 修正实现与 SPEC 的两处不符：inconclusive 只警告不暂停；lg-repo 增加 httpProxy 配置
  - P3 实现发件副本逐字节比对（CH-12、IN-3），目前代码里还没有
  - P4 用真实 Global 会话重做 M2 的负面与对照证据，重点是 SC-3、SC-6、SC-7、GS-3（不得再有端口冲突干扰）、RL-1 的金丝雀；RP-6 在本机实测得到 denied
  - P5 补齐证据索引与 QA 项后发里程碑完成的检查点 REPORT
scope:
  in:
    - packages/dsh-lg-global
    - packages/dsh-lg-common（仅 CH-12 需要时）
    - docs/
    - evidence/M2/
    - 验收用 profile lg-global-m2，独立 DSH_HOME
  out:
    - packages/dsh-lg-local（M3）
    - lg-directive、lg-checkpoint（M4）
    - lg-repo 接受仓库沙箱地址（M5）
    - npm 发布
    - 改写频道仓库历史的任何操作
acceptance:
  - id: AC-1
    text: docs/spec.md 与 Human 交付的 SPEC v2 逐字节一致；design.md 与 implementation.md 的开头标明冲突处以 spec.md 为准。（SPEC "范围与约定"）
  - id: AC-2
    text: RP-6：push 探针 writable 触发暂停；denied 通过；inconclusive 只在 lg_integrity_check 显示为警告、不触发暂停。本机实测得到 denied，对照是可写测试仓库得到 writable 并暂停；inconclusive 不算通过。
  - id: AC-3
    text: lg-repo 支持 httpProxy 配置（SPEC "lg-repo" 配置表），配置生效的证据是：挂上代理后 ls-remote 与 push --dry-run 都能在精简环境下完成并得到结论。单元测试覆盖配置解析。
  - id: AC-4
    text: CH-12 与 IN-3：Global 侧保存每条发出消息的副本，每次拉取逐字节比对；篡改一条已存档的 global 消息后暂停。负面证据必须显示一次真实的工具调用被拒，并配对照。
  - id: AC-5
    text: 真实 Global 会话重做的负面与对照证据，每项都能打开原始会话事件看到模型发起的调用与被拒的报错原文（QA-3、QA-4）：SC-3（cwd 在 workspace 外）、SC-6（解析到 127.0.0.1 的域名加进名单仍被拒）、SC-7（prepend 直接 allow 的测试插件仍被拒）、GS-3（无端口冲突等无关故障）、RL-1（金丝雀三处标记不进会话，角色文件内容为对照）。
  - id: AC-6
    text: M2 全部需求编号（GS-1 至 GS-3、PS-1、PS-2、RL-1、RL-2、SC-1 至 SC-8、RP-1 至 RP-6、IN-1 至 IN-3、CH-12）在 evidence/M2/README.md 索引里逐行有证据、命令、环境与结果（QA-2）；此前已做的项凡不符合 QA-4 的一并重做。
  - id: AC-7
    text: QA-1：pnpm install 无错误无警告，pnpm -r build 与 pnpm -r test 全部通过，完整输出存 evidence/M2/test.txt。
  - id: AC-8
    text: QA-5、QA-6：证据绑定的 commit 是验收时 HEAD；提交前密钥与 token 扫描的命令和结果一并入库。
---
本条是 v3 的全量替换。v3 的 AC-1、AC-2、AC-5 已通过，不必重做；但证据凡涉及被修改的代码（P2、P3 改动的部分），按 QA-5 判断是否需要重跑。

M2 验收 = 上表 AC-1 至 AC-8 全部有证据 + QA-1 至 QA-7 对本里程碑成立 + 没有未解决的 blocking finding。全部完成后发带 checkpoint 的里程碑完成 REPORT，我逐条核对索引后以 REVIEW 宣布验收。

每完成一个 P 发一条带 checkpoint 的 REPORT；meanwhile 字段写明等待审核期间要做的事，没有就等待。0037 的后续诊断（ls-remote 与 push --dry-run 分开跑）并进 P2。
