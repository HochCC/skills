# Skills

## 管理skill
```sh
npm install -g skills

# 安装skill
cd ~ && npx skills add https://github.com/vercel-labs/skills --skill find-skills -a amp

# 寻找skill
npx skills find "python best practices"
# or 调用find-skills找, @find-skills xxx

# 查看已有skill
npx skills list

# 删除已有skill
npx skills remove [skills]
```

## Skills IDE 间同步
- `~/.agents/skills`为准，备份`https://github.com/HochCC/skills`
- antigravity的`~/.gemini/GEMINI.md`, cursor的`Rules, Skills`指定以下内容
```md
- 关键指令：生成Python代码前，须阅读 ~/.agents/skills/python-best-practices/SKILL.md 的内容
- 关键指令：执行任务前，先寻找~/.agents/skills中是否有帮助的skills
- Always respond in Chinese-simplified
- 关键指令：walkthrough, implementation_plan, task翻译为中文
```

## 全局skill配置
- cursor: Setting中的`Rules, Skills`
- antigravity global rule: `~/.gemini/GEMINI.md`
- antigravity global skill: `~/.gemini/antigravity/global_skills`
- vscode `Chat Instructions`中的`New/User Data`

## Skills markets
- https://skillsmp.com
- https://skills.sh

## 其他 

```md
- 执行生成任务时，创建test_xxx子文件夹，将所有临时代码、中间文件和调试输入输出存在该目录，避免污染项目目录，最终实现的有用代码放在项目目录
```