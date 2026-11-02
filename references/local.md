## 本地

先看这个仓库 `package.json` 里的 `dev` 脚本，按有的名字启动。没有 `dev:local` 就不要跑 `npm run dev:local`，那会报 `Missing script`。

当前常见的 `npm run dev` 在本机没有沙箱环境变量时，会转到 `scripts/dev-local.js`，启动前加载 `.env.local`。沙箱里同一条命令走另一条路径。有的仓库把本地入口单独写成 `npm run dev:local`。有的模板单独起 Nest，端口以那个脚本为准，不要套另一个仓库的端口。

后端挂在 Vite 里时自己不听端口。页面和接口常见在 `http://localhost:8080/app/<app_id>/...`。

调接口同时带 `Cookie: suda-csrf-token=<X>` 和 `X-Suda-Csrf-Token: <X>`。`/api/*` 不能在浏览器地址栏打开。本地登录的人被 `.env.local` 的 `SUDA_WEBUSER` 钉死，改请求头无效。`.env.local` 不进 git。shell 里已经导出的变量优先于 `.env.local`。验别人的视角要换启动时的身份，换请求头无效。这类断言的第一条必须钉住「不是当前这个人」。

起服务前先停掉旧的 dev server。端口要逐个查，`lsof -ti :3000 :8080` 是非法参数，失败后不能当成端口已空。起来之后先在日志里搜 `already in use`，再打接口。陈旧进程会让请求打到旧代码上。

要人帮忙看的服务端现象，做成日志再用 `+log-list` 捞。不要做成「请打开这个接口」。

`package.json`、`package-lock.json`、`.githooks/`、`.spark_project`、`client/src/api/gen`，以及 `scripts/` 里文件头写着由 sync 维护的启动、构建、lint 脚本，会被平台同步重写。不要手改这些文件，也不要往 `package.json` 加脚本。自己的页面预览放在 `scripts/<事项>-preview/`。依赖用静态 `import`。部署时按静态 import 裁剪 `node_modules`，运行时 `require()` 的包不会带上。`process.env` 只在客户端入口那一处会被替换。
