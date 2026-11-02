## 文件和页面

页面上的固定图放这个应用自己的文件存储，用 `lark-cli apps +file-upload` 的地址。别的应用的链接不能拿来用。用户上传的附件只存 `bucketId` 和 `filePath`，打开时按当前登录态 `createSignedUrl`。不要把 `download_url`、`blob:` 或 base64 写进业务表。签名链接谁拿到谁就能打开，权限要在发链接之前验。页面挂在 `/app/<app_id>` 下，相对路径会请求错位置。`window.location.pathname` 含这个前缀，`useLocation().pathname` 不含。不受前缀影响时用 HashRouter。

做界面时少写辅助标签和说明。字段名、按钮、空状态已经说清的，不要再配一句「这里是什么」或操作说明。靠位置、层级和留白把结构说清楚。Avoid unnecessary secondary labels and descriptions. Let the layout do the explaining.

输入框用 `text-base md:text-sm`。低于 16px 时，iOS 聚焦会放大整页。高度用 `dvh`。可点区域大约不小于 40×40。

固定底栏在软键盘弹起时会浮在键盘上方，盖住正在编辑的输入框。用 `visualViewport` 的高度变化判断键盘，弹起时收起底栏；不要用 `:has(:focus)`，WebView 里支持不一致。底栏补 `env(safe-area-inset-bottom)`。

宽屏改完要在窄屏再走一遍：放得下、主操作还在第一屏。桌面后台可以不逐页改成手机布局，但不能把页面撑出横向滚动。视觉以这个项目的 `tailwind-theme.css` 为准。任意值类名，以及用字符串拼出来的类名，改完后在构建出的 CSS 里确认这个类生成了。扫描不到的类不会报错。

`ui/Image` 会给本地图铺一层灰底，透明 SVG 会出现一块矩形。透明图内联成组件。模板若关掉了 `strictNullChecks`，`T | undefined` 的错误 tsc 不会报。`process.env` 只在客户端入口替换，单测在 Node 里是绿的，浏览器里没有 `process`。

## 页面预览

改列表、筛选、弹层、日历这类界面时，不要靠本地 dev 服务验收。那条会拉飞书登录并挂上 Nest。另开一个只装 `@vitejs/plugin-react` 的 Vite，直接渲染仓库里的真实页面或组件。

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

这条只说明页面在这份假数据下画得出来、点得动。登录、权限、接口和真库都不算验过。改了数据路径，仍用这个仓库的 dev 脚本或线上再验。
## 首屏体积

路由用 `React.lazy` 加 `Suspense`，管理端页面不要全部静态 import。Vite 的 `manualChunks` 把 react、antd、平台工具包拆成独立 vendor，发版时只失效业务包。

echarts 从 `echarts/core` 按需注册，xlsx 在导出函数里 `await import`。列表里不要逐行挂 `<video>`，点开再 `createSignedUrl`，否则一屏几十个签名请求。

先量再改：主 chunk 超过 1.5MB（gzip）时，看它混进了哪些页面和库。
