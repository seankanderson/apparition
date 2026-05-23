# VS Code Copilot — Workspace Instructions

Read `AGENTS.md` at the root of this project first. It contains the full project context, file map, stack rules, security rules, and communication style that apply to all interactions.

The notes below are specific to VS Code Copilot behaviour.

---

## VS Code-Specific Behaviour

- Read existing files before modifying them
- Generate **complete files**, not partial snippets — never output `// rest of file unchanged`
- Keep generated files short — prefer more files over long ones
- State which files were created or changed after every task
- Flag any deviation from the stack rules in `AGENTS.md` before implementing
- If something might break an existing feature, say so clearly before proceeding
