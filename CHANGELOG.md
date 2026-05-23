# Changelog

All notable changes to the Apparition template will be documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned
- MCP server extraction: each `## STEP` in `onboarding-questions.md` becomes a tool
- Example app: Invoice tracker (complete walkthrough)
- Example app: Simple CRM
- Stripe payment integration prompt
- S3/R2 file storage prompt
- Email queue pattern (Resend + documents table)

---

## [0.1.0] — Initial Release

### Added
- Core stack definition: ASP.NET Core + Razor Pages + PostgreSQL + Dapper
- **Guided onboarding interview** (`prompts/onboarding-questions.md`) — ask one question at a time, synthesize into master prompt; structured for future MCP server extraction
- Master prompt template
- Auth prompt (cookie-based login/logout)
- Feature prompt (generic new feature)
- Deployment prompt (Railway + Render)
- **Git setup guide** covering GitHub, GitLab, and Bitbucket (all supported by Railway and Render)
- Copilot agents: build (with interview), deploy, feature
- Architecture documentation
- Database patterns documentation
- Deployment guide (Railway + Render)
- Customization guide
- Troubleshooting guide
- `.github/copilot-instructions.md` for workspace-level Copilot rules
