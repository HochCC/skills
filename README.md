# Skills

- **动态加载指南**: 执行任务前读取本文件，根据任务需求按需加载对应 skill 的 `SKILL.md`，严禁加载无关 skill。
- 生成Python代码前，须阅读`python-best-practices/SKILL.md`
- 始终用简体中文回答
- walkthrough, implementation_plan, task翻译为中文

---

## 加载规则

1. **按需加载**: 仅当任务明确匹配某 skill 时，读取其 `SKILL.md`
2. **自动应用**: `python-best-practices` 在涉及 `.py` 文件时始终自动加载
3. **禁止贪婪加载**: 不要一次性加载多个无关 skill
4. **组合使用**: 复杂任务可组合多个 skill（如 pptx + theme-factory）
5. **附属资源**: 各 skill 目录下的附属文件（`scripts/`, `reference/` 等）按 `SKILL.md` 指引按需读取


## Skill 索引

### 按场景快速匹配

| 用户意图 | 推荐 Skill | 加载路径 |
|---------|-----------|---------|
| 创建/编辑 `.pptx` 幻灯片 | pptx | `pptx/SKILL.md` |
| 创建/编辑 `.docx` Word文档 | docx | `docx/SKILL.md` |
| 创建/编辑 `.xlsx` / `.csv` 电子表格 | xlsx | `xlsx/SKILL.md` |
| PDF 读取/合并/拆分/填表/OCR | pdf | `pdf/SKILL.md` |
| 创建海报/视觉艺术/设计作品 | canvas-design | `canvas-design/SKILL.md` |
| 构建网页/落地页/React组件/仪表盘 | frontend-design | `frontend-design/SKILL.md` |
| 构建复杂多组件 HTML artifact (shadcn/ui) | web-artifacts-builder | `web-artifacts-builder/SKILL.md` |
| 为 artifact 应用主题配色/字体 | theme-factory | `theme-factory/SKILL.md` |
| 编写文档/提案/技术规范 | doc-coauthoring | `doc-coauthoring/SKILL.md` |
| Git 提交 / conventional commit | git-commit | `git-commit/SKILL.md` |
| 构建 MCP Server (LLM 工具集成) | mcp-builder | `mcp-builder/SKILL.md` |
| 编写 Python 代码 (自动应用) | python-best-practices | `python-best-practices/SKILL.md` |
| Python 性能分析与优化 | python-performance-optimization | `python-performance-optimization/SKILL.md` |
| Python 测试 / pytest / TDD | python-testing-patterns | `python-testing-patterns/SKILL.md` |
| 创建/改进 agent skill | skill-creator | `skill-creator/SKILL.md` |
| 搜索/安装新 skill | find-skills | `find-skills/SKILL.md` |
| 分析代码库生成 AI agent 指令 | project-instruct | `project-instruct/SKILL.md` |

---

## Skill 详细说明

### 📄 文档与办公

#### pptx
- **路径**: `pptx/SKILL.md`
- **触发词**: deck、slides、presentation、幻灯片、.pptx
- **功能**: 创建/读取/编辑/合并 PowerPoint 演示文稿。支持模板编辑、从零创建、缩略图预览、OOXML 底层操作
- **附属资源**: `editing.md`, `pptxgenjs.md`, `html2pptx.md`, `ooxml.md`, `scripts/`

#### docx
- **路径**: `docx/SKILL.md`
- **触发词**: Word doc、word document、.docx、报告、备忘录、信函、模板
- **功能**: 创建/读取/编辑 Word 文档。支持目录、页眉页脚、追踪修订、批注、图片操作、.doc 转换
- **附属资源**: `docx-js.md`, `ooxml.md`, `scripts/`

#### xlsx
- **路径**: `xlsx/SKILL.md`
- **触发词**: 电子表格、.xlsx、.xlsm、.csv、.tsv、数据清洗、财务模型
- **功能**: 创建/读取/编辑电子表格。支持公式、图表、格式化、数据分析、财务模型规范（颜色编码、公式审计）
- **附属资源**: `scripts/`

#### pdf
- **路径**: `pdf/SKILL.md`
- **触发词**: .pdf、PDF、合并PDF、拆分PDF、填写表单、OCR、水印
- **功能**: PDF 全流程处理——读取/提取文本表格、合并/拆分、旋转、水印、加密解密、填写表单、OCR
- **附属资源**: `reference.md`, `forms.md`, `scripts/`

---

### 🎨 设计与前端

#### canvas-design
- **路径**: `canvas-design/SKILL.md`
- **触发词**: 海报、艺术作品、设计、视觉作品、.png、.pdf 设计输出
- **功能**: 创建原创视觉设计作品（.png/.pdf）。先生成设计哲学/美学宣言，再据此进行视觉表达。强调匠心、大胆配色与构图
- **附属资源**: `canvas-fonts/`（27 款开源字体）

#### frontend-design
- **路径**: `frontend-design/SKILL.md`
- **触发词**: 网页、落地页、仪表盘、React 组件、HTML/CSS 布局、美化 UI
- **功能**: 创建高品质、有辨识度的前端界面。强调独特排版、大胆配色、动效、空间构图，避免 AI 千篇一律风格
- **设计原则**: 拒绝 Inter/Roboto 等通用字体和紫色渐变；每次设计必须有独特美学方向

#### web-artifacts-builder
- **路径**: `web-artifacts-builder/SKILL.md`
- **触发词**: 复杂 HTML artifact、多组件应用、shadcn/ui、需要状态管理/路由
- **功能**: 使用 React 18 + TypeScript + Vite + Tailwind + shadcn/ui 构建复杂前端 artifact，打包为单一 HTML 文件
- **附属资源**: `scripts/init-artifact.sh`, `scripts/bundle-artifact.sh`

#### theme-factory
- **路径**: `theme-factory/SKILL.md`
- **触发词**: 主题、配色方案、字体搭配、为 slides/文档/网页应用主题
- **功能**: 10 套预设专业主题（配色+字体），可应用于幻灯片、文档、网页等任意 artifact。支持自定义主题生成
- **附属资源**: `themes/`（10 套主题定义文件）

---

### 🐍 Python 开发

#### python-best-practices
- **路径**: `python-best-practices/SKILL.md`
- **触发词**: 自动应用（`globs: ["**/*.py"]`, `alwaysApply: true`）
- **功能**: Python 编码规范——类型注解、快速失败、守卫子句、Google Style Docstring、Ruff/Black/isort。包含 Vibe Coding 控制协议（规划→接口优先→TDD）
- **级别**: 始终生效，编写任何 `.py` 文件时自动加载

#### python-performance-optimization
- **路径**: `python-performance-optimization/SKILL.md`
- **触发词**: Python 性能、瓶颈、慢代码、profiling、cProfile、内存泄漏
- **功能**: CPU/内存/行级 profiling，算法优化、并行化、缓存、Native 扩展等优化策略

#### python-testing-patterns
- **路径**: `python-testing-patterns/SKILL.md`
- **触发词**: pytest、单元测试、TDD、mock、fixture、测试套件
- **功能**: pytest 全面测试指南——fixture、参数化、mock、异步测试、属性测试、CI/CD 集成

---

### 🔧 工程工具

#### git-commit
- **路径**: `git-commit/SKILL.md`
- **触发词**: commit、git commit、提交代码、/commit
- **功能**: 基于 Conventional Commits 规范自动分析 diff，生成标准化提交信息。支持自动检测 type/scope、智能暂存

#### mcp-builder
- **路径**: `mcp-builder/SKILL.md`
- **触发词**: MCP Server、Model Context Protocol、LLM 工具集成
- **功能**: 创建高质量 MCP Server，使 LLM 能与外部服务交互。推荐 TypeScript 栈，覆盖从研究规划到实现测评全流程
- **附属资源**: `reference/`（最佳实践、TS/Python 指南、评测框架）, `scripts/`

#### skill-creator
- **路径**: `skill-creator/SKILL.md`
- **触发词**: 创建 skill、改进 skill、评测 skill、skill 描述优化
- **功能**: 创建/迭代/评测 agent skill 的完整流程——意图捕获→起草→测试→评测→优化触发描述
- **附属资源**: `agents/`, `eval-viewer/`, `scripts/`, `references/`

#### find-skills
- **路径**: `find-skills/SKILL.md`
- **触发词**: 查找 skill、搜索能力、安装 skill、"how do I do X"
- **功能**: 通过 `npx skills find [query]` 搜索开源 skill 生态，发现并安装新能力

#### project-instruct
- **路径**: `project-instruct/SKILL.md`
- **触发词**: 分析代码库、生成 AI 指令、copilot instructions、agent rules
- **功能**: 分析代码库架构、工作流、约定，自动生成/更新 AI agent 项目指令文件

---

### 📝 写作协作

#### doc-coauthoring
- **路径**: `doc-coauthoring/SKILL.md`
- **触发词**: 写文档、起草提案、创建规范、PRD、设计文档、RFC
- **功能**: 结构化文档共创工作流：上下文收集 → 迭代精炼 → 读者测试。确保文档对他人同样可读

---

## Skills IDE 配置

- 始终用简体中文回答，执行任务前先用 `cat ~/.agents/skills/README.md` 读取并遵循其中所有约束和规则。
- 允许自主读取源码及配置, 运行cat,grep,ls,find等所有读取操作无需询问，默认不直接读取 >500KB 的非文本文件，不扫描：`.git/`, `dist/`, `build/`, `__pycache__/`等。


- 本地`~/.agents/skills`，云端`https://github.com/HochCC/skills`
- cursor: Setting中的`Rules, Skills`
- antigravity global rule: `~/.gemini/GEMINI.md`
- antigravity global skill: `~/.gemini/antigravity/global_skills`
- vscode `Chat Instructions`中的`New/User Data`
