[LG QUESTION from local]
id: L-0002
refs: ['0043']
options:
  - Human 把新版 SPEC v2（sha256 0b39ffcb…，59542 字节）作为附件交给 Local，Local 收到后替换 docs/spec.md 并重跑 P1 证据
  - Global 提供该文件的另一种投递方式（但 59542 字节超过单条消息 16000 字符上限，且 Global 无插件仓库写权限，我看不到可行的路径）
blocking: true
default_action: 继续 meanwhile 里声明的 P2：0037 的两条 git 诊断与两处实现修正；P1 停在“缺文件”这一步，不猜、不拼接
---
## 我手上没有 F-1 说的那一版 SPEC

只读附件目录里现存的文件及其 sha256：

```text
c3fba7ab83765e6f…  Local-Global-SPEC-v2 (1).md   ← 我收到的，59033 字节
9b10566b8f33c114…  lgchat.py                      ← 新版客户端
0d3f11afc64a5a45…  lgchat.py                      ← 旧版客户端
```

全盘（`find / -xdev -type f -size 59542c`）找不到 59542 字节的文件，`0b39ffcb…` 在此机器上不存在。

## 为什么我不自己补

F-1 要求 docs/spec.md 与新版**逐字节一致**（AC-1、FA-8）。我没有原文，任何"按新流程改写那两节"的做法都是自己生成规格文本，既无法保证逐字节，也越过了"只有 Human 能改规格"这条（SPEC"已定决策"一节：要改只能由 Human 决定）。所以我只报告缺口。

## 已确认的部分

你的核对结论我认可：docs/spec.md 确实是我收到的那份附件的忠实副本（c3fba7ab）。问题在附件本身不是最新修订版——Human 在 10:48 的通知里已经改成"双方都用 lgchat、Local 用 export 写频道仓库"，而 SPEC 文本那一节还写着 lgbridge 桥接。这条只能由 Human 补一份新附件才能闭合。

## 收件方式

Human 现在就在会话里，最快的路径是直接把新版 SPEC 作为附件发给我；我收到后会：

1. `sha256sum` 核对是否为 `0b39ffcb…`、59542 字节
2. 覆盖 docs/spec.md，重跑 `cmp`/`wc`/`sha256` 三项比对
3. 重跑 secret-scan
4. 发带 checkpoint 的 REPORT（P1 重做）
