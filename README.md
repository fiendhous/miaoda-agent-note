# miaoda-agent-note

团队新建飞书妙搭项目时，装这一个 skill。Agent 按 `SKILL.md` 做本地启动、数据库、发版、定时任务、日志和飞书接入。

各项目自己的业务仍写在该仓库的 `AGENTS.md`。两边不一致时，以那个 `AGENTS.md` 为准。

## 安装

在新的妙搭仓库根目录执行：

```bash
npx skills add fiendhous/miaoda-agent-note -y -a grok
```

团队里还有别的 Agent 时，把 `-a grok` 换成 `--agent '*'`。装好后，技能在这个项目的 `.grok/skills/`。

仓库保持公开。不要把连接串、token、人员 id 写进本仓库。命令里的应用 id 用 `<app_id>`。
