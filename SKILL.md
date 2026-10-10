---
name: miaoda-app
description: >
  飞书妙搭全栈应用的开工步骤。新建妙搭项目，或要改数据库、本地启动、发版、定时任务、日志、飞书接入、文件存储、外部系统时使用。
  触发词：妙搭, miaoda, apaas, AGENTS.md, docs, sprint/default, lark-cli, 发版, db-execute, schema.ts, COMMENT, pg_audit, dev:local, 页面预览, cron, 定时任务, 多维表格同步。
  Use when starting or changing a Feishu Miaoda NestJS and React app.
---

# 妙搭应用

新项目只装本 skill。项目自己的业务写在该仓库的 `AGENTS.md`。和本 skill 冲突时，以那个 `AGENTS.md` 为准。脚手架留下的设计指南不算写完，开工时要把 `AGENTS.md` 补成下面这一节要求的样子。

命令里的 `<app_id>` 用这个应用自己的 id。都加 `--as user`。不要把 lark-cli 自己的应用 id 传给 `apps +*`。

平台共性来自 [miaoda-agent-kit](https://github.com/Lens-lzy/miaoda-agent-kit)（CC BY 4.0，实测于 2026-08 至 2026-09）。本 skill 只保留换一个妙搭项目仍然成立的做法，并补上 2026-10 之后多个项目核实过的发版、定时任务、注释、`pg_audit` 和页面预览。上游「改完就推」和「DDL 一律放 `scripts/sql/`」不要照搬：发版要等这个项目的 `AGENTS.md` 允许，DDL 放生成器碰不到的目录。

## 仓库

仓库根是这个应用自己的目录，不是上一级。只在 `sprint/default` 上改、提交和发布。`main` 由发布成功后平台推进，不要在 `main` 上提交、合并、变基或推送，也不要强推。

线上地址是 `https://<租户域名>/app/<app_id>`。

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
- 界面以哪份主题文件为准。
- 给人看的说明放在 `docs/`，并在这里链到 `docs/README.md`。
- 明确不做的事。

业务规则、表名、审批链、外部地址写在这个 `AGENTS.md` 和 `docs/`，不要写回本 skill。

## 给人看的文档

每个妙搭项目都要有仓库根下的 `docs/`。给人读的 Markdown 都放这里，不要只留在对话里，也不要散落在随机目录。

仓库根的 `AGENTS.md` 继续留在根上，供 Agent 开工时读取，并链到 `docs/README.md`。平台如果要求根上有 `README.md`，那份只保留怎么安装和怎么进入应用。功能说明、操作步骤、方案和变更记录放进 `docs/`。

开工时先建：

- `docs/README.md`：目录。每一份文档一行，写它讲什么、算不算现行口径。
- `docs/使用说明.md`：同事不看代码也能完成的操作。页面、谁能用、做完之后看到什么。
- `docs/work-log/`：每次改完仓库追加一份记录。

改完代码、修缺陷、改配置、做调研或出方案，都要新增或更新对应文档。文档没写，任务不算做完。只回答问题、没有改仓库时，不必新开记录。

工作记录的文件名用 `YYYY-MM-DD-事项.md`，事项是短中文。正文用完整句子，写清：

- 别人会看到什么变化。
- 做了什么、改了哪些地方。
- 怎么验证，结果是什么。
- 还没定的规则和已知限制。没定的规则写成「未定」，不要写成系统已经会自动做。

同一天的连续小改可以补在当天同一份记录里。换一类事情就新开一份。带日期的旧档案按它自己的日期理解，不要改写成当前线上状态，也不要把本次记录写进旧档案。

不要写连接串、token、密码和人员 id。

## 本地

```bash
npm run dev:local
```

后端挂在 Vite 里，自己不听端口。页面和接口走 `http://localhost:8080/app/<app_id>/...`。`npm run dev` 不刷新数据库连接串。

调接口同时带 `Cookie: suda-csrf-token=<X>` 和 `X-Suda-Csrf-Token: <X>`。`/api/*` 不能在浏览器地址栏打开。本地登录的人被 `.env.local` 的 `SUDA_WEBUSER` 钉死，改请求头无效。`.env.local` 不进 git。shell 里已经导出的变量优先于 `.env.local`。验别人的视角要换启动时的身份，换请求头无效。这类断言的第一条必须钉住「不是当前这个人」。

起服务前先停掉旧的 dev server。端口要逐个查，`lsof -ti :3000 :8080` 是非法参数，失败后不能当成端口已空。起来之后先在日志里搜 `already in use`，再打接口。陈旧进程会让请求打到旧代码上。

要人帮忙看的服务端现象，做成日志再用 `+log-list` 捞。不要做成「请打开这个接口」。

`package.json`、`package-lock.json`，以及 `scripts/` 里文件头写着由 sync 维护的启动、构建、lint 脚本，会被平台同步重写。不要手改这些文件，也不要往 `package.json` 加脚本。自己的页面预览放在单独子目录，见下一节。依赖用静态 `import`。部署时按静态 import 裁剪 `node_modules`，运行时 `require()` 的包不会带上。`process.env` 只在客户端入口那一处会被替换。

## 页面预览

改列表、筛选、弹层、日历这类界面时，不要靠 `npm run dev:local` 验收。那条会拉飞书登录并挂上 Nest。另开一个只装 `@vitejs/plugin-react` 的 Vite，直接渲染仓库里的真实页面或组件。

目录放 `scripts/<事项>-preview/`，里面有 `index.html`、`main.tsx`、`vite.config.ts`。不要把启动命令写进 `package.json`。

```bash
npx vite --config scripts/<事项>-preview/vite.config.ts
```

`root` 设成这个预览目录。`server.host` 用 `127.0.0.1`，`strictPort: true`。端口记在该项目的工作记录里，不要占 `8080`。

配置里：

- `@` 指到 `client/src`。页面还引用 `@shared` 时，一并指到仓库的 `shared`。
- `@/inspector.dev.css` 指到 `@lark-apaas/fullstack-vite-preset` 里的 `empty.css`。
- 页面引用的 `@/api`、`@lark-apaas/client-toolkit/logger` 指到预览目录里的 mock。mock 只返回这次要看的数据，不发请求。
- `css.postcss` 用仓库根的 `postcss.config.js`。`server.fs.allow` 包含仓库根。否则真实页面的 Tailwind 类生成不出来，或读不到预览目录外面的文件。

`main.tsx` 引入真实的 `client/src/index.css` 和真实页面。页面用了 react-router 就包一层 `MemoryRouter`，不要依赖 `/app/<app_id>` 这层壳。手机页把容器收在大约 390 宽，再在浏览器的手机视口里点一遍。

假数据的类型跟真实接口的返回类型一致。先放界面上必须出现的那些行，再补筛选、空态和分支会点到的不同取值。只有一种取值时，筛选项点下去是空的，这页不算看过。不要写人员 id、token、连接串。

这条只说明页面在这份假数据下画得出来、点得动。登录、权限、接口和真库都不算验过。改了数据路径，仍用 `dev:local` 或线上再验。

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

SQL 要在真库跑一次，并确认结果不是全 0。全 0 和「条件恒不成立」长得一样。对 SQL 文本做 `toContain` 只锁住字符串，锁不住它能执行。drizzle 拼出来的是表名不是别名，外层再起别名会报 `42P01`。

本地查询脚本有 DML、没有 DDL。`+db-execute --file` 只接受当前目录下的相对路径。

`schema.ts` 表达不了表达式索引和条件索引，生成时还会丢掉 `WHERE`，读起来像全表唯一，库里却是部分唯一。这两类索引用 SQL 文件维护。索引对账放 `tools/db-index-audit.js`，不要放进平台托管的 `scripts/`。

还有这些库结构事实：

- `EXTRACT` 跟随会话时区。写进 CHECK 要锚定 UTC。
- 条件唯一索引会在「先插新行再删旧行」的中间态撞上。
- `ON CONFLICT (id)` 挡不住业务唯一索引。
- `CREATE TABLE IF NOT EXISTS` 改不了已经存在的列默认值。
- `estimated_row_count` 是旧统计，不能用来判断表空不空。
- 清空数据前先查 `NO ACTION` 外键。
- 按一个所有行都相同的列排序，等于没有排序。
- 并存的两条唯一索引口径不同时，新写入会 500。先列出这张表的全部唯一索引。drizzle 把驱动错误包在 `cause` 上。
- 两个环境的 DDL 直接 `diff` 噪声很大，先把空白和标识符归一化。
- 服务端 `Promise.all` 不会让 drizzle 查询重叠。耗时约等于次数乘以往返。要快就减少查询次数。

## 系统自带的 pg_audit

`pg_audit` 是平台自带的变更日志，不是业务表。不要 `CREATE`、不要 `INSERT`、不要 `COMMENT ON`。Nest 运行时也不要建它。

业务表要在数据表的「记录日志」里打开「记录变更日志追溯」，平台才会往 `pg_audit` 写。开关只负责写入。

SQL 编辑器里 `SELECT * FROM pg_audit` 能看到行，应用直查经常是空的，这是行级权限，不是没数据。应用里要看操作日志时，在该应用的 SQL 编辑器手工建 `query_audit_logs`，函数用 `SECURITY DEFINER`，建在本应用自己的 schema 里。不要写成 `public.query_audit_logs`，`public` 会被拒。不要抄另一个应用的 schema 名。应用只调用这个函数，不直接读 `pg_audit`。

这个项目不做操作日志时，不要建函数，也不要在页面上留入口。把「不做」写进 `AGENTS.md`。

平台 RBAC 授予容易、撤销难，不要把它当唯一安全边界。每个写接口自己校验。

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

`npm run build` 会先跑交互式插件初始化，非交互终端里失败不代表代码没编过。分别用 `npm run build:server` 和 `npm run build:client`。`npm run lint` 看退出码，不要用 `tail` 判断。没有 `npm run ts:server` 时，用 `type:check:server` 和 `type:check:client`。检查命令不要加 `--silent`，它会把「脚本不存在」吞掉，退出码仍是 0。管道里的 `$?` 是最后一条命令的状态，要用 `set -o pipefail`。

`package-lock.json` 冲突不要手合，删掉后 `npm install --package-lock-only` 再 `npm ci --dry-run`。rebase 时 `--ours` 和 merge 时的 `--ours` 是反的。fullstack-cli 会把 SQL 文件从 `100644` 改成 `100755`，这种改动先看是不是只有文件模式变了。自动升级只在干净且已经 pull 过的工作区上跑。

发布会把库结构一起带上。发布后再跑 `+db-env-migrate` 得到没有待迁差异，是正常的，不要为此再发一次结构。

## 定时任务

最短间隔 30 分钟。`*/5` 和 `0,15,30,45` 会被拒绝。半小时写成 `0,30 * * * *`。触发器名字里带 `*/` 时，启用接口会 404。名字必须和代码里的键整串一致。

```bash
lark-cli apps +automation-list --as user --app-id <app_id>
lark-cli apps +automation-disable --as user --app-id <app_id> --name '<完整名字>'
```

`can_delete` 为 false 的旧任务删不掉，保持停用。块注释、网段、路径和正则里都不能出现 `*` 紧挨 `/`，报错会飘到几十行外。`+automation-update` 要加 `--yes`，说明最长 50 个字符。界面上不要承诺精确到分钟。

抢占必须写在发消息、发通知和一切副作用之前，条件里带上当前状态，并用受影响行数判断。0 行就返回。排在发送之后的占位只防状态改两次，防不了发两次。没抢到不要记成发送失败。

按表里已有的行去筛人的定时任务，先问这张表的行是谁、在什么时候写进去的。写入方如果在这条链路的下游，任务会永远不启动，日志仍像正常。

排查先 `+automation-list` 看在不在、启没启用，再用线上日志看有没有跑过。库里没有产出，不能推断成平台没配触发器。

## 日志

观测命令只有线上。搜索用 `+log-list`，单条用 `+log-get --log-id`。`timestamp_ns` 是纳秒，除以 `1e9` 是秒。

日志只包括进入请求或触发器之后的代码。中间件和启动阶段的日志不会出现，而且不报错。业务 4xx 会被平台拦截器记成 ERROR。客户端断网（状态码 0）和平台自己的 `sdk_innerapi` 也会进错误日志。接口错误计数是 0，只说明没有 5xx。

线上自查日志最早挂到拦截器。中间件和 `onApplicationBootstrap` 里的日志不会出现。先不带关键字拉一段，看有没有流量、`module` 分布和最新一条的时间，再按关键字搜。观测命令不要传 dev 环境。

几个探针不要共用一份条数额度。共用「记完了」标志时，必须是全部信号都记完，不能是其中任意一个。

## 飞书

应用收不到未登录的入站请求。飞书事件用长连接，不用 webhook。卡片图片认 `image_key`，空的 `img_key` 会让整张卡失败。通知收件人传妙搭 userId。无效 id 保持终态，不要放进重试。写进库不等于收件人在自己的列表里看得见，读路径的过滤要单独验。

卡片图片不接受任意 URL。下载字节，用 `im.image.create` 换成 `image_key` 再放进卡片，需要应用身份权限 `im:resource:upload`。没开通时是 `99991672`，SDK 抛的是 axios 400，不是飞书 `code != 0`。这是租户权限，不用用户重新授权。没有 key 就不要放这个图片元素，空的 `img_key` 会让整张卡变成空消息，日志仍可能是成功。

未登录打 `/api` 会 302 到登录，打 `/openapi` 是 403。登录那一跳会保留原查询串。链接预览要在开发者后台打开能力、登记 URL 规则、用长连接订阅，回调要在 3 秒内返回。只正式版应用支持，域名不能用通配符。每个应用最多 50 条长连接，销毁时要 `close()`。

飞书授权是整页跳转，跳之前把用户正要做的事放进 `sessionStorage`，回来再做。以用户身份发消息，卡在重定向 URL 白名单，和 scope 不是一回事。部门列表的顺序是乱的，按 `order` 排。开放平台文档页是前端渲染的，要抓就抓 `.md`。

字段是 null 时，先把接口实际返回的 key 打出来，再判断是权限没开还是字段名不对。`department_ids` 开权限后会有，`department_path` 补权限也不会有。

多维表格要持续进应用时：streaming 任务写入 `*_sync` 原始表，触发器再投影到正式表。关联主数据对不上就跳过这条。源表删除不要删掉已经产生的业务单。应用里已经进行中或已完成的状态，不要被源表的待办盖回去。项目不需要同步时，不要加这条链路。

## 文件和页面

图片放这个应用自己的文件存储，用 `lark-cli apps +file-upload` 的地址。别的应用的链接不能拿来用，也不要把 base64 写进业务表。页面挂在 `/app/<app_id>` 下，相对路径会请求错位置。`window.location.pathname` 含这个前缀，`useLocation().pathname` 不含。不受前缀影响时用 HashRouter。

输入框用 `text-base md:text-sm`。低于 16px 时，iOS 聚焦会放大整页。高度用 `dvh`。可点区域大约不小于 40×40。宽屏改完要在窄屏再走一遍：放得下、主操作还在第一屏。桌面后台可以不逐页改成手机布局，但不能把页面撑出横向滚动。视觉以这个项目的 `tailwind-theme.css` 为准。任意值类名改完后，在构建出的 CSS 里确认这个类生成了。

`ui/Image` 会给本地图铺一层灰底，透明 SVG 会出现一块矩形。透明图内联成组件。模板若关掉了 `strictNullChecks`，`T | undefined` 的错误 tsc 不会报。`process.env` 只在客户端入口替换，单测在 Node 里是绿的，浏览器里没有 `process`。

## 外部系统

生产环境不能靠临时加自定义环境变量来配地址。地址和凭证放在服务端，页面不直连。飞书服务器访问不到内网和本机。对方必须是公网地址。

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
