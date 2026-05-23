# Contributing to Apparition

Apparition is a template used by non-technical founders. Every contribution should be measured against one question:

> *Does this make it easier for a non-developer to build and ship a real app?*

---

## What We're Looking For

### High value contributions
- New prompt templates for common features (search, payments, email, file upload)
- Better troubleshooting entries based on real errors people hit
- Clearer language in any doc — plain English improvements always welcome
- Example apps that demonstrate the template in action
- Agent improvements that produce better first-try code

### We will not accept
- Stack changes (no React, no EF, no Blazor — see [docs/architecture.md](docs/architecture.md))
- Abstractions that add complexity without a clear benefit for non-technical users
- Features that require external services with complex setup
- Changes that make the codebase harder for AI to reason about

---

## How to Contribute

1. **Fork the repo** and create a branch: `git checkout -b my-improvement`
2. **Make your changes** — keep them focused on one thing
3. **Test your changes** — if you're contributing a prompt, actually run it in Copilot and verify the output
4. **Open a pull request** with a clear description of what you changed and why

---

## Improving a Prompt

When contributing a new or improved prompt:

1. Run it in GitHub Copilot Chat (Agent mode) against a fresh project
2. Verify the generated code compiles: `dotnet build`
3. Verify it deploys to Railway: `dotnet publish -c Release`
4. Note any edge cases in the prompt file itself

---

## Improving Documentation

- Write for someone who has never used a terminal
- Avoid jargon — if you must use a technical term, explain it in parentheses
- Keep steps numbered and short
- Use the existing docs as a style guide

---

## Reporting Issues

Found something broken? Please file an issue with:
- What you were trying to do
- What prompt you used (paste the full prompt)
- What error you got (paste the full error)
- What stack you're on (OS, .NET version, Railway vs Render)

---

## Code of Conduct

Be helpful, be kind, be patient. Many people using this template are learning as they go.
