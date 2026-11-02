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

`+release-get` 还没到 `finished` 或 `failed`，节点是 `PENDING` 时，停下来告诉用户在等审批。当前用户不一定是审批人。不要一直查。

已有发布在进行时再创建，会返回 `400002553`。`--jq` 是参数，不能把查询表达式放在命令末尾。接口 403 不能用来判断这版代码在不在，CSRF 在进路由之前就会拒绝。

推送被拒就 `git pull --rebase origin sprint/default`，然后重跑检查。不要 `--force`。用户没有明确要求时，不要自行提交、推送或发版。

### 平台自动升级

平台会自己在 `sprint/default` 上提交 `chore(upgrade): auto-upgrade by fullstack-cli`，同步模板文件、清理 `package.json`、升级 `@lark-apaas` 依赖，不通知。

推送前先 `git fetch`，看 `git log HEAD..origin/sprint/default`。有升级提交就 rebase。升级改过 `package.json` 时，本地 `node_modules` 可能还是旧版本：`package.json` 写 `^1.0.41`，装着的是 `1.0.40`。这时本地 build 和测试全绿，CI 按 lock 装新版本，差异要到线上才暴露。

rebase 后先 `npm ls <包名>` 对一下；不一致就 `npm ci`。`npm install` 有时在 `node_modules/@esbuild/<平台>` 上报 `ENOTEMPTY`，删掉这个目录再装。不要为此改 lock，也不要把升级的版本手改回去。确实不兼容时，显式钉版本并写原因。

这个应用有正在跑的定时任务时，发版前停用，`finished` 后再打开。停用返回里经常没有状态，用 list 确认。

`npm run build` 会先跑交互式插件初始化，非交互终端里失败不代表代码没编过。分别用 `npm run build:server` 和 `npm run build:client`。`npm run lint` 看退出码，不要用 `tail` 判断。输出里是 `write EPIPE` 时，退出码不能当成代码违规，改跑 `npx eslint <文件>` 分清是管道断了还是代码有问题。没有 `npm run ts:server` 时，用 `type:check:server` 和 `type:check:client`。检查命令不要加 `--silent`，它会把「脚本不存在」吞掉，退出码仍是 0。管道里的 `$?` 是最后一条命令的状态，要用 `set -o pipefail`。

`package-lock.json` 冲突不要手合，删掉后 `npm install --package-lock-only` 再 `npm ci --dry-run`。rebase 时 `--ours` 和 merge 时的 `--ours` 是反的。fullstack-cli 会把 SQL 文件从 `100644` 改成 `100755`，这种改动先看是不是只有文件模式变了。

发布会把库结构一起带上。发布后再跑 `+db-env-migrate` 得到没有待迁差异，是正常的，不要为此再发一次结构。
## 日志

观测命令只有线上。搜索用 `+log-list`，单条用 `+log-get --log-id`。`timestamp_ns` 是纳秒，除以 `1e9` 是秒。

日志只包括进入请求或触发器之后的代码。中间件和启动阶段的日志不会出现，而且不报错。业务 4xx 会被平台拦截器记成 ERROR。客户端断网（状态码 0）和平台自己的 `sdk_innerapi` 也会进错误日志。接口错误计数是 0，只说明没有 5xx。

线上自查日志最早挂到拦截器。中间件和 `onApplicationBootstrap` 里的日志不会出现。先不带关键字拉一段，看有没有流量、`module` 分布和最新一条的时间，再按关键字搜。观测命令不要传 dev 环境。

几个探针不要共用一份条数额度。共用「记完了」标志时，必须是全部信号都记完，不能是其中任意一个。
