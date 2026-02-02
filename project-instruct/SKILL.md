---
name: generate-project-instructions
description: Analyze a codebase and generate or update AI agent project instructions (e.g., copilot/agent rules) to make AI coding agents productive immediately.
alwaysApply: true
---


When analyzing a codebase to generate AI agent instructions, always:

1. **Discover the Big Picture Architecture**
   - Identify major modules, services, and boundaries
   - Explain how data flows across components
   - Infer the rationale behind the structure (monorepo, microservices, pipelines, etc.)
   - Reference key directories and files that define architecture

2. **Extract Critical Developer Workflows**
   - Document build, test, and debug commands that are not obvious from code
   - Identify scripts, Makefiles, CI configs, Dockerfiles, or package scripts
   - Highlight local dev setup steps and environment assumptions

3. **Capture Project-Specific Conventions**
   - Coding patterns that differ from common frameworks
   - Naming conventions, directory layout rules, custom abstractions
   - Internal frameworks or wrappers around standard libraries

4. **Document Integration Points**
   - External services, APIs, databases, cloud resources
   - IPC, RPC, REST, message queues, model inference endpoints, etc.
   - Cross-language or cross-repo communication patterns

5. **Source Existing Agent Instructions**
   - Search for existing AI/agent rule files:
     ```
     **/{.github/copilot-instructions.md,AGENT.md,AGENTS.md,CLAUDE.md,.cursorrules,.windsurfrules,.clinerules,.cursor/rules/**,.windsurf/rules/**,.clinerules/**,README.md}
     ```
   - Merge intelligently instead of overwriting
   - Preserve valuable content, remove outdated sections

6. **Write Actionable, Project-Specific Guidance**
   - Keep concise (20–50 lines)
   - Use markdown structure with bullet points and headings
   - Include concrete file paths and examples
   - Avoid generic advice (e.g., “write tests”, “handle errors”)

- Produce or update the AI instruction file, create if not exists (default in project root `.cursor/rules/project_instruct.md`)
- Ask for feedback on unclear or missing sections for refinement