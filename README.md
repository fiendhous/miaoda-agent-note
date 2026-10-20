<div align="center">

# miaoda-agent-note

**飞书妙搭 · 团队共用开工技能**

新仓库装这一个。Agent 打开项目，就按同一套平台口径开工。

</div>

```text
技能        miaoda-app
规则        SKILL.md
业务        各项目自己的 AGENTS.md
安装位置    <仓库>/.grok/skills/
```

## 安装

在妙搭仓库根目录执行：

```bash
npx skills add fiendhous/miaoda-agent-note -y -a grok
```

已经装过的仓库再执行一次，才会换成当前这份 `SKILL.md`。

团队里还有别的 Agent：

```bash
npx skills add fiendhous/miaoda-agent-note -y --agent '*'
```

## 装完看哪里

新项目只装本技能。上游 [miaoda-agent-kit](https://github.com/Lens-lzy/miaoda-agent-kit)（CC BY 4.0）里换一个妙搭项目仍然成立的做法，已经吸收进 `SKILL.md`。

脚手架只留下设计指南时，先按 `SKILL.md` 把该仓库的 `AGENTS.md` 补全。项目 `AGENTS.md` 与本技能不一致时，以项目 `AGENTS.md` 为准。

本仓库公开。连接串、token、人员 id 不要写进来。命令里的应用 id 写成 `<app_id>`。
