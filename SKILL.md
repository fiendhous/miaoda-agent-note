---
name: miaoda-app
description: >
  飞书妙搭全栈应用的开工步骤。新建妙搭项目，或要改数据库、界面、本地启动、发版、定时任务、日志、飞书接入、文件存储、外部系统时使用。先读本文件，做到哪一步再读 references/ 里对应的那一节。
  触发词：妙搭, miaoda, apaas, AGENTS.md, docs, sprint/default, lark-cli, doctor, 授权, 发版, db-execute, schema.ts, COMMENT, pg_audit, dev:local, 页面预览, 界面, cron, 定时任务, 多维表格同步。
  Use when starting or changing a Feishu Miaoda NestJS and React app.
---

# 妙搭应用

新项目只装本 skill。项目自己的业务写在该仓库的 `AGENTS.md`。和本 skill 冲突时，以那个 `AGENTS.md` 为准。脚手架留下的设计指南不算写完，开工时要把 `AGENTS.md` 补成下面这一节要求的样子。

命令怎么写见「本机 lark-cli」。目录名、Git 路径和 lark-cli 自己的应用不是这一个 id，见「认地址」。

平台共性来自 [miaoda-agent-kit](https://github.com/Lens-lzy/miaoda-agent-kit)（CC BY 4.0，实测于 2026-08 至 2026-09）。本 skill 只保留换一个妙搭项目仍然成立的做法，并补上 2026-10 之后多个项目核实过的发版、定时任务、注释、`pg_audit`、页面预览、认地址和本机授权。上游「改完就推」和「DDL 一律放 `scripts/sql/`」不要照搬：发版要等这个项目的 `AGENTS.md` 允许，DDL 放生成器碰不到的目录。

## 做到哪一步读哪一节

本文件只留开工时每次都要守的规则。下面的文件做到那一步再读，读完再改。同一条规则只写在一处。

| 在做 | 先读 |
|---|---|
| 本机把应用跑起来 | `references/local.md` |
| 给人看的文档、工作记录 | `references/docs.md` |
| 数据库、DDL、`schema.ts`、`pg_audit` | `references/database.md` |
| 发版、构建、线上日志 | `references/release.md` |
| 定时任务 | `references/automation.md` |
| 飞书、身份、外部系统 | `references/feishu.md` |
| 页面、预览、文件、首屏体积 | `references/pages.md` |

## 仓库

仓库根是这个应用自己的目录，不是上一级。只在 `sprint/default` 上改、提交和发布。`main` 由发布成功后平台推进，不要在 `main` 上提交、合并、变基或推送，也不要强推。

线上地址是 `https://<租户域名>/app/<app_id>`。

## 认地址

这四个名字经常不一样。命令、日志、数据库和发版只用应用 id。

| 是什么 | 从哪认 | 用来做什么 |
|---|---|---|
| 应用 id | `.spark/meta.json` 的 `app_id`，或浏览器地址里的 `/app/app_xxx` | `apps +*`、接口前缀、线上地址 |
| 线上地址 | 租户域名加 `/app/<app_id>`。发布完成后以 `+release-get` 的 `online_url` 为准 | 给人打开 |
| Git 仓库 | `git remote get-url origin` | clone、push |
| 本机目录 | 文件夹名 | 只是本机路径 |

应用 id 以 `app_` 开头。用户只给了名字时，用 `lark-cli apps +list --keyword "<名字>" --as user` 对一下，多个结果先问人。

Git 远端多在 `miaoda-git.feishu.cn`。路径最后一段常见 `code_...`，有的仓库是 `app_...`。这两段都不要直接拿来当应用 id，先和 `.spark/meta.json` 对上。Git 地址也不能当成线上页面打开。

浏览器里的 `/page/<token>` 是创意模式的 meta token，不是这份全栈仓库的应用 id。

推送认证失败时，用这个应用的 id 执行 `lark-cli apps +git-credential-init --app-id <app_id> --as user`，再重试原来的 git 命令。不要改远端地址。

## 本机 lark-cli

操作妙搭用已经登录的用户身份，不为每个项目再造一个自建应用。lark-cli 自己的飞书应用只负责本机登录，一个本机配置一份，id 以 `cli_` 开头。这个 id 不能传给任何 `apps +*`。

开工先查环境：

```bash
lark-cli doctor
lark-cli whoami
lark-cli --version
```

`doctor` 看配置、登录和连通性。有一项失败就先修这项。`whoami` 里实际生效的身份应是 user。`--as bot` 看不到这个人的应用。

妙搭命令写成 `lark-cli apps +<命令> --as user --app-id <app_id>`。旗标不确定时，先给这条命令加 `--help`。

JSON 成功看 `ok == true` 或退出码 0。成功结果没有顶层 `code`。

本机还没有绑定 CLI 应用时，才跑一次 `lark-cli config init --new`。已经配过就不要再跑，否则会再造一个飞书应用。

未登录，或明确缺少 apps 权限时，才授权一次：

```bash
lark-cli auth login --domain apps --no-wait --json
```

已经登录就不要为了保险再登一次。授权链接原样交给用户，等用户说完成后再继续原来的命令。

看有没有新版本用 `lark-cli update --check`。要升级用 `lark-cli update`，它同时更新 CLI 和它管理的 skills。输出里的 `_notice.update` 先做完手头的事，再告诉用户可以升级。只有命令明确说客户端版本过旧时，升级后用原来的参数重试。

`--file`、`--output`、`--path` 只接受当前目录下的相对路径。绝对路径会报 `unsafe file path`。先切到仓库根再执行。

退出码 10 是确认门，不是失败。`+db-execute`（含 `SELECT`）和 `+automation-update` 的帮助里写了 `--yes`，拟好的命令里就带上，先给人看，同意后再执行。其他命令跑出 10 时，同样先给人看操作和参数，同意后再加到原命令末尾。人还没看过时不要自己补上。

应用运行时代码不要调用 `lark-cli`。它只在开发时查平台。业务请求走仓库里的 SDK。

## 查代码

仓库根有 `.codegraph/` 时，查符号、调用链、某段逻辑在哪、改哪里会波及谁，以及动手改之前，先调 CodeGraph MCP 的 `codegraph_explore`。不要先全库 grep，也不要把文件整篇读进来。一次调用带回相关符号的带行号源码、它们之间的调用路径和影响范围。返回的源码按已经读过处理。

这个 MCP 没有默认项目。`projectPath` 传本仓库根目录的绝对路径。工具不在时，在仓库根执行 `codegraph explore "<符号或问题>"`，输出相同。

图里没有，或确认是生成物、配置时，再用搜索和读取。不要自己跑 `codegraph init` 或 `uninit`。改完后索引通常会跟上；对不上再跑 `codegraph sync`。没有 `.codegraph/` 时不要建索引，用仓库里的搜索和读取。

`.codegraph/` 里的数据库、pid 和套接字留在本机。仓库里只留 `.codegraph/.gitignore`。

## 完善 AGENTS.md

新项目或接着改一个只有设计指南的仓库时，先补 `AGENTS.md`，再写业务代码。文件放在仓库根。设计指南可以留在后面，现行口径写在文件开头。

至少写清这些，没有的项写「不做」：

- 应用 id、线上地址、仓库根、开发分支。`main` 由发布推进。
- 什么时候允许提交、推送、发版。用户没说时不要自行发版。
- 这个应用有没有 dev 库。没有就写明不要建。
- DDL 放在哪个目录。`schema.ts` 是生成物。
- 谁能看、谁能改。平台角色和业务表里的人分别是什么。
- 要不要定时任务、飞书卡片、未登录回调、多维表格同步、外部系统。
- 操作日志用系统自带的 `pg_audit`，还是这个项目不做操作日志。
- 界面以哪份主题文件为准。开头写明少写辅助标签和说明，让布局把信息说清楚，见 `references/pages.md`。
- 给人看的说明放在 `docs/`，并在这里链到 `docs/README.md`。
- 明确不做的事。

业务规则、表名、审批链、外部地址写在这个 `AGENTS.md` 和 `docs/`，不要写回本 skill。

## 做完

先确认请求打到了、身份对、预期条数事先算得出来，再看业务对不对。打出状态码，确认返回类型，0 条不算通过。空列表会让「没有坏数据」永远成立。写入按「写、再读、比对」验，每一步重新读当前状态再拼下一次请求。反向验证要确认关掉之后真的被挡住。

改文件做实验时，锚点在文件里只能有一处，一次只改一处。还原用副本拷回去，不要 `git checkout` 单个文件，那会丢掉这个文件上所有未提交改动。假数据库没有唯一索引和约束，这三样只有真库测得到。全树扫描、索引对账、NUL 字节这类用例不按改动文件筛选，推之前跑全量。冒烟数据不要照着线上那条恰好能过的形状造。

同一条规则写在两处、用例只打了一处，功能可以没上线而测试全绿。服务端要么给人真名，要么给空，不要用 userId 或「用户」这种看起来像名字的兜底。写在代码里的默认文案，运营在后台改不到。失败调用的耗时记 `NULL`，不要记 0。表单把掩码原样提交回来时要报错，不能当成「没改」。

字段拿不到时，先打出实际返回的 key、实际 SQL 或日志里的 module，再下结论。同一条件下的恒定值，不能证明它和条件无关。修了几次还没好，先证明问题仍然存在。

这些写法会静默失败：

| 写法 | 后果 |
|---|---|
| `lsof -ti :3000 :8080` | 非法参数。多端口写成 `-i :3000 -i :8080` |
| zsh 里 `for f in $VAR` | 不做词分割。用 `while read -r` |
| bash 3.2 里 `"$var。中文"` | 全角字符被吞进变量名。写成 `${var}` |
| 块注释里的 `*/` | 注释提前结束，报错在几十行之后 |
| 文件里有 NUL | `grep` 当二进制跳过，`file` 显示 `data`。tsc 和测试仍是绿的 |
| `npm run 不存在的脚本 --silent` | 没有输出，退出码 0 |
| `s[:a] + s[b:]` 且 b < a | 不是删除，是把中间复制一份 |
| 批量替换之后不复查 | 先 `git status`，再 grep 一次 |

批量改完要再搜一遍。一个只能靠「下次不要手改生成文件」维持的约定，要配一个能跑的对账，而不是靠人记得。
