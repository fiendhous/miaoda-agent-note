# miaoda-agent-note

团队新建飞书妙搭项目时，装这一个 skill。`SKILL.md` 已经吸收 [miaoda-agent-kit](https://github.com/Lens-lzy/miaoda-agent-kit)（CC BY 4.0）里换项目仍然成立的做法，并补上发版、定时任务、中文注释和 `pg_audit`。不用再同时装上游 kit。

各项目自己的业务写在该仓库的 `AGENTS.md`。脚手架只留设计指南时，要先按 `SKILL.md` 把 `AGENTS.md` 补全。两边不一致时，以那个 `AGENTS.md` 为准。

## 安装

在新的妙搭仓库根目录执行：

```bash
npx skills add fiendhous/miaoda-agent-note -y -a grok
```

团队里还有别的 Agent 时，把 `-a grok` 换成 `--agent '*'`。装好后，技能在这个项目的 `.grok/skills/`。

仓库保持公开。不要把连接串、token、人员 id 写进本仓库。命令里的应用 id 用 `<app_id>`。
