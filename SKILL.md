---
name: miaoda-app
description: >
  飞书妙搭全栈应用的开工步骤。新建妙搭项目，或要改数据库、本地启动、发版、定时任务、日志、飞书接入、文件存储、外部系统时使用。
  触发词：妙搭, miaoda, apaas, AGENTS.md, sprint/default, lark-cli, 发版, db-execute, schema.ts, COMMENT, pg_audit, dev:local, cron, 定时任务, 多维表格同步。
  Use when starting or changing a Feishu Miaoda NestJS and React app.
---

# 妙搭应用

新项目只装本 skill。项目自己的业务写在该仓库的 `AGENTS.md`。和本 skill 冲突时，以那个 `AGENTS.md` 为准。脚手架留下的设计指南不算写完，开工时要把 `AGENTS.md` 补成下面这一节要求的样子。

命令里的 `<app_id>` 用这个应用自己的 id。都加 `--as user`。不要把 lark-cli 自己的应用 id 传给 `apps +*`。

## 仓库

仓库根是这个应用自己的目录，不是上一级。只在 `sprint/default` 上改、提交和发布。`main` 由发布成功后平台推进，不要在 `main` 上提交、合并、变基或推送，也不要强推。

线上地址是 `https://<租户域名>/app/<app_id>`。

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
- 界面以哪份主题文件为准。
- 明确不做的事。

业务规则、表名、审批链、外部地址只写在这个 `AGENTS.md`，不要写回本 skill。

## 本地

```bash
npm run dev:local
```

后端挂在 Vite 里，自己不听端口。页面和接口走 `http://localhost:8080/app/<app_id>/...`。`npm run dev` 不刷新数据库连接串。

调接口同时带 `Cookie: suda-csrf-token=<X>` 和 `X-Suda-Csrf-Token: <X>`。`/api/*` 不能在浏览器地址栏打开。本地登录的人被 `.env.local` 的 `SUDA_WEBUSER` 钉死，改请求头无效。`.env.local` 不进 git。

`package.json`、`package-lock.json`、`scripts/` 会被平台同步重写。依赖用静态 `import`。部署时按静态 import 裁剪 `node_modules`，运行时 `require()` 的包不会带上。`process.env` 只在客户端入口那一处会被替换。

## 数据库

`server/database/schema.ts` 由 `npm run gen:db-schema` 生成。手改会在下次生成时消失，也写不出条件索引和表达式索引。真实 DDL 放生成器碰不到的 SQL 文件里。

每张业务表、每一个字段都要有中文注释，含 `_created_at`、`_created_by`、`_updated_at`、`_updated_by`。用 PostgreSQL 的 `COMMENT ON TABLE` 和 `COMMENT ON COLUMN`，和本次 DDL 一起执行。不要用 MySQL 那种写在列定义里的 `COMMENT`。注释写业务含义，不要只重复英文表名或字段名。

```sql
COMMENT ON TABLE example_order IS '订单，一门店一个营业日一行';
COMMENT ON COLUMN example_order.store_name IS '门店名称';
COMMENT ON COLUMN example_order.payload IS '@type { sku: string; qty: number }[] 下单明细';
```

JSONB 字段保留平台要求的 `@type { ... }`，中文说明写在同一条 `COMMENT` 里，不能用中文换掉 `@type`。

SQL 编辑器一次只跑一张表的注释。整文件一次贴进去会超时，注释里的斜杠还可能被当成语句分隔。不要给系统表 `pg_audit` 写注释。

执行顺序：`lark-cli apps +db-execute`，然后 `gen:db-schema`，重新读过 `schema.ts` 再写业务代码。

先看这个应用有没有拆出 dev：

- 有 dev：结构变更打在 dev。online 禁止 DDL。发布会把库结构一起带上。
- 只有一套库：`+db-execute` 不传 `--environment`。不要执行 `+db-env-create`，那一步不可逆。

公共连接禁止 `WITH` 和多语句，DDL 自动提交，不能用 `BEGIN … ROLLBACK` 试探，也不能用 `pg_advisory_lock`。应用进程里已经跑通的 drizzle SQL 不受这条限制。人员字段只比较 `(字段).user_id`，没有 `.name`。不要把 JS 的 `Date` 绑进 SQL。UUID 数组写成 `ARRAY[$1::uuid, ...]::uuid[]`。`SELECT` 也要加 `--yes`。结果大约到 1000 行会截断，按人汇总用 `GROUP BY`。

SQL 要在真库跑一次，并确认结果不是全 0。

## 系统自带的 pg_audit

`pg_audit` 是平台自带的变更日志，不是业务表。不要 `CREATE`、不要 `INSERT`、不要 `COMMENT ON`。Nest 运行时也不要建它。

业务表要在数据表的「记录日志」里打开「记录变更日志追溯」，平台才会往 `pg_audit` 写。开关只负责写入。

SQL 编辑器里 `SELECT * FROM pg_audit` 能看到行，应用直查经常是空的，这是行级权限，不是没数据。应用里要看操作日志时，在该应用的 SQL 编辑器手工建 `query_audit_logs`，函数用 `SECURITY DEFINER`，建在本应用自己的 schema 里。不要写成 `public.query_audit_logs`，`public` 会被拒。不要抄另一个应用的 schema 名。应用只调用这个函数，不直接读 `pg_audit`。

这个项目不做操作日志时，不要建函数，也不要在页面上留入口。把「不做」写进 `AGENTS.md`。

## 身份

妙搭 userId 和飞书 user_id 是两套号，混用不会报错。转换走 `AuthNPaasService`。通讯录批量接口没有部门路径，要沿上级部门找，并设深度上限。

平台角色留给少数管理身份。店长、伙伴、岗位这一类人写在业务表里，由接口校验。只藏按钮不够，每个写接口都要验。可见范围先保持仅创建者，等人给出名单再放开。

## 发版

```text
改代码 → 在 sprint/default 提交 → git push → 发版
```

```bash
lark-cli apps +release-create --as user --app-id <app_id> --branch sprint/default
lark-cli apps +release-get --as user --app-id <app_id> --release-id <id>
```

发版部署的是远端 `sprint/default` 上已经 push 的提交。没 push 就发，部署的是旧提交，界面仍可能显示成功。创建后先是 `publishing`，前几次查到的 `commit_id` 还是上一版。`status=finished` 且下面的命令退出码为 0，这版才上去：

```bash
git merge-base --is-ancestor <这次提交> <commit_id>
```

已有发布在进行时再创建，会返回 `400002553`。`--jq` 是参数，不能把查询表达式放在命令末尾。接口 403 不能用来判断这版代码在不在，CSRF 在进路由之前就会拒绝。

推送被拒就 `git pull --rebase origin sprint/default`，然后重跑检查。不要 `--force`。用户没有明确要求时，不要自行提交、推送或发版。

这个应用有正在跑的定时任务时，发版前停用，`finished` 后再打开。停用返回里经常没有状态，用 list 确认。

`npm run build` 会先跑交互式插件初始化，非交互终端里失败不代表代码没编过。分别用 `npm run build:server` 和 `npm run build:client`。`npm run lint` 看退出码。没有 `npm run ts:server` 时，用 `type:check:server` 和 `type:check:client`。检查命令不要加 `--silent`。

## 定时任务

最短间隔 30 分钟。`*/5` 和 `0,15,30,45` 会被拒绝。半小时写成 `0,30 * * * *`。触发器名字里带 `*/` 时，启用接口会 404。名字必须和代码里的键整串一致。

```bash
lark-cli apps +automation-list --as user --app-id <app_id>
lark-cli apps +automation-disable --as user --app-id <app_id> --name '<完整名字>'
```

`can_delete` 为 false 的旧任务删不掉，保持停用。块注释里不能出现 `*/`。排查先看任务在不在、启没启用，再用线上日志看它有没有跑过。

## 日志

观测命令只有线上。搜索用 `+log-list`，单条用 `+log-get --log-id`。`timestamp_ns` 是纳秒，除以 `1e9` 是秒。

日志只包括进入请求或触发器之后的代码。中间件和启动阶段的日志不会出现，而且不报错。业务 4xx 会被平台拦截器记成 ERROR。客户端断网（状态码 0）和平台自己的 `sdk_innerapi` 也会进错误日志。接口错误计数是 0，只说明没有 5xx。

## 飞书

应用收不到未登录的入站请求。飞书事件用长连接，不用 webhook。卡片图片认 `image_key`，空的 `img_key` 会让整张卡失败。通知收件人传妙搭 userId。无效 id 保持终态，不要放进重试。

多维表格要持续进应用时：streaming 任务写入 `*_sync` 原始表，触发器再投影到正式表。关联主数据对不上就跳过这条。源表删除不要删掉已经产生的业务单。应用里已经进行中或已完成的状态，不要被源表的待办盖回去。项目不需要同步时，不要加这条链路。

## 文件和页面

图片放这个应用自己的文件存储，用 `lark-cli apps +file-upload` 的地址。别的应用的链接不能拿来用，也不要把 base64 写进业务表。页面挂在 `/app/<app_id>` 下，相对路径会请求错位置。`window.location.pathname` 含这个前缀，`useLocation().pathname` 不含。不受前缀影响时用 HashRouter。

输入框用 `text-base md:text-sm`，高度用 `dvh`，可点区域大约不小于 40×40。宽屏改完要在窄屏再走一遍。视觉以这个项目的 `tailwind-theme.css` 为准。任意值类名改完后，在构建出的 CSS 里确认这个类生成了。

## 外部系统

生产环境不能靠临时加自定义环境变量来配地址。地址和凭证放在服务端，页面不直连。飞书服务器访问不到内网和本机。对方必须是公网地址。

## 做完

先确认请求打到了、身份对、预期条数算得出来，再看业务对不对。空列表会让「没有坏数据」永远成立。写入按「写、再读、比对」验。还原文件用副本拷回去，不要 `git checkout` 单个文件。

`lsof -ti :3000 :8080` 是非法参数，失败后不能当成端口已空。zsh 里 `for f in $VAR` 不做词分割。bash 3.2 里中文旁边的变量写成 `${var}`。
