## 定时任务

最短间隔 30 分钟。`*/5` 和 `0,15,30,45` 会被拒绝。半小时写成 `0,30 * * * *`。触发器名字里带 `*/` 时，启用接口会 404。名字必须和代码里的键整串一致。

创建 cron 必须带 `--trigger-type cron`。只给 `--cron` 会报 `required flag(s) "trigger-type" not set`。没有 `--type` 这个参数。新建默认是停用的，要再启用。

```bash
lark-cli apps +automation-create --as user --app-id <app_id> \
  --name '<完整名字>' --trigger-type cron --cron '0,30 * * * *' \
  --description '<不超过 50 字>'
lark-cli apps +automation-enable --as user --app-id <app_id> --name '<完整名字>'
lark-cli apps +automation-list --as user --app-id <app_id>
lark-cli apps +automation-disable --as user --app-id <app_id> --name '<完整名字>'
```

`can_delete` 为 false 的旧任务删不掉，保持停用。块注释、网段、路径和正则里都不能出现 `*` 紧挨 `/`，报错会飘到几十行外。`+automation-update` 的 `--yes` 见 `SKILL.md`「本机 lark-cli」。说明最长 50 个字符。界面上不要承诺精确到分钟。

抢占必须写在发消息、发通知和一切副作用之前，条件里带上当前状态，并用受影响行数判断。0 行就返回。排在发送之后的占位只防状态改两次，防不了发两次。没抢到不要记成发送失败。

按表里已有的行去筛人的定时任务，先问这张表的行是谁、在什么时候写进去的。写入方如果在这条链路的下游，任务会永远不启动，日志仍像正常。

排查先 `+automation-list` 看在不在、启没启用，再用线上日志看有没有跑过。库里没有产出，不能推断成平台没配触发器。
