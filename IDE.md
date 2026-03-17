# Skills IDE 配置

## 全局rules

- 始终用简体中文回答，执行任务前先用 cat ~/.agents/skills/README.md 读取并遵循其中所有约束和规则。
- 默认不直接读取 >1MB 的非文本文件，不扫描：`.git/`, `dist/`, `build/`, `__pycache__/`, `.gitignore`等路径。
- 错误总结：将重要的、重复的、有教训的错误精简记录到 `~/.claude/memory/MEMORY.md`，避免再犯

---

- 本地`~/.agents/skills`，云端`https://github.com/HochCC/skills`
- cursor: Setting中的`Rules, Skills`
- antigravity global rule: `~/.gemini/GEMINI.md`
- antigravity global skill: `~/.gemini/antigravity/global_skills`
- vscode `Chat Instructions`中的`New/User Data`
- claude `~/.claude/CLAUDE.md`
- codex `~/.codex/AGENTS.md`

## claude-code-router 配置

```sh
conda create -n claude_env nodejs=20 -c conda-forge -y
npm install -g @anthropic-ai/claude-code
npm install -g @musistudio/claude-code-router
ccr_code () {
    conda activate claude_env
    ccr code --permission-mode dontAsk "$@" # acceptEdits, dontAsk
}
```

`~/.claude/settings.json`
```json
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash",
      "WebSearch",
      "WebFetch",
      "NotebookEdit",
      "mcp__ide__executeCode",
      "mcp__ide__getDiagnostics"
    ]
  }
}

```