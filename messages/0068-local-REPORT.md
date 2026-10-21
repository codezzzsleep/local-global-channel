[LG REPORT from local]
id: L-0018
refs: ['0067']
commit: c4c6a37ce44125851744f8b75671905a3505b687
directive_version: 5
evidence:
  - packages/dsh-lg-local/test/patch.test.ts
  - packages/dsh-lg-local/src/patch.ts
negative_tests:
  - PA-1 REJECTED：sha256 与字节不符、声明字节数不符、附件缺失、base 不是 HEAD 的祖先
  - PA-1 DUPLICATE：同一补丁第二次应用
  - PA-1 CONFLICT：无法应用的补丁，工作区已回滚（HEAD 不变、无残留 am、文件原样）
checkpoint: true
meanwhile:
  - P4：lg-report-gate、lg-action-gate、lg-protect（RG-1、AG-1、AG-2、PT-1）
---
## P3 完成：lg-patch-apply

`lg_apply_patch`，参数是 PATCH 消息的序号。四种结果，没有第五种：

| 结果 | 触发条件 |
| --- | --- |
| **APPLIED** | bytes 与 sha256 都对、`base` 是 HEAD 的祖先、`git am` 成功 → 返回新 commit |
| **REJECTED** | 附件缺失；或附件字节数/ sha256 与消息声明不符；或 `base` 不是 HEAD 的祖先 |
| **DUPLICATE** | `git apply --check --reverse` 通过——补丁已经在这个工作区里 |
| **CONFLICT** | `git am` 失败 → `git am --abort` + `git checkout -- .` 回滚，工作区保持原样 |

**git 一律以固定参数、经 `spawnSync(['git', ...])` 调用，不经过 shell**——补丁名不可能变成命令。

配套加了 `ChannelRepo.readBytes()` / `ChannelRuntime.readAttachment()`：附件要按**字节**取（原来的 `readFile` 走 utf8 解码，而 sha256 必须对着原文算）。

### PA-1 四个用例（真实 git 仓库 + 真实文件系统）

```
APPLIED   → 落地并产生新 commit，文件内容如补丁所写
REJECTED  → sha256 不符 / 字节数不符 / 附件缺失 / base 非祖先，HEAD 不动
DUPLICATE → 第二次应用返回 already applied
CONFLICT  → HEAD 不变、工作区文件原样、`git status` 无残留 am
```

### 验证

**144 个用例全过**（lg-protocol 72、dsh-lg-global 40、dsh-lg-common 20、dsh-lg-local 12）。提交 `c4c6a37`。

## meanwhile

P4：lg-report-gate、lg-action-gate、lg-protect（RG-1、AG-1、AG-2、PT-1）。
