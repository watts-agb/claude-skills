# Security Validation Report — `watts-agb/claude-skills`

**Generated:** 2026-06-30  
**Scope:** Every tracked file in the repository (full working tree)  
**Files scanned:** 557  
**Reviewer:** Automated + manual static analysis (no code was executed)

---

## Overall Verdict

> ### 🟢 Overall Security Score: **94 / 100** — *Safe to use*
>
> No malicious code, prompt-injection, backdoors, data-exfiltration, or committed secrets were found in any of the 557 files. The repository is a documentation/"skills" pack (≈90% Markdown) for the Claude Code agent. The few point deductions are for *inherently risky but legitimate* documentation patterns (vendor `curl | sh` install one-liners) and skills that legitimately request `Bash` tool access — not for anything malicious.

**Bottom line:** This fork appears to be a clean, untampered copy of the upstream [`Jeffallan/claude-skills`](https://github.com/jeffallan/claude-skills) project. I recommend you may use it, with the minor cautions listed below.

---

## How scoring works

| Score | Meaning |
|------:|---------|
| **100** | Clean. No malicious code, injection, secrets, or exfiltration patterns. |
| **90–99** | Clean, but exercises a sensitive *capability* (e.g. declares `Bash` tool access). |
| **60–89** | Contains a risky-but-legitimate pattern you should review before running (e.g. `curl \| sh`). |
| **0–59** | Suspicious / malicious indicators found. **None in this repo.** |

---

## What was checked

Across **all** files (Markdown, Python, JS/MJS/TS, YAML, JSON, shell, Astro, config):

- **Executable code** — Python scripts, Node/MJS build scripts, the shell test, the Makefile, and the Puppeteer screenshot script.
- **Remote code execution** — `curl|sh`, `wget|bash`, `eval`, `exec`, `Invoke-Expression`, `base64 -d`, `atob`, `fromCharCode`, `/dev/tcp`, reverse shells.
- **Data exfiltration** — env dumps, reads of `~/.ssh`, `~/.aws/credentials`, `.netrc`, `id_rsa`, posting tokens/secrets to remote hosts.
- **Committed secrets** — API keys, tokens, passwords, private keys.
- **Prompt injection** (critical for an AI-skills repo) — "ignore previous instructions", "reveal your system prompt", hidden overrides, instructions to exfiltrate or act without consent.
- **Hidden / obfuscated content** — zero-width characters, bidirectional Unicode (Trojan Source), hidden HTML comments.
- **CI/CD supply chain** — GitHub Actions workflows, pre-commit hooks, third-party action pinning.
- **External network references** — every `http(s)` domain referenced in the repo.

---

## Key findings

### ✅ Clean signals
- **No prompt injection.** No skill or doc attempts to override the agent, hide instructions, leak prompts, or act without user consent. On the contrary, workflow commands and the `security-reviewer` skill explicitly require **user confirmation / written authorization** before acting.
- **No executable malice.** All 4 Python scripts import standard library only (`argparse`, `pathlib`, `re`, `sys`, `json`) — no `subprocess`, `os.system`, `eval`, `requests`, or sockets. They only read/write local documentation files.
- **No committed secrets.** Every key/token match was either a placeholder (`YOUR_API_KEY`, `your-api-key`) or GitHub Actions `${{ secrets.* }}` syntax in example docs.
- **No hidden Unicode / Trojan-Source.** Zero zero-width or bidirectional control characters in any text file (only the PNG asset is binary).
- **Clean CI/CD.** Workflows pin official, reputable actions (`actions/checkout@v6`, `actions/setup-node@v6`, `softprops/action-gh-release@v2`). No curl-pipe-to-shell, no untrusted scripts.
- **Benign network references.** External domains are vendor docs, `registry.npmjs.org`, `github.com`, the upstream author's docs site (`jeffallan.github.io`), and `example.com`/`localhost` placeholders. No suspicious or look-alike domains.
- **Fork integrity.** Internal links, plugin manifests, and the docs site still reference the upstream author (`jeffallan`). The fork shows **no signs of tampering** with these.

### ⚠️ Minor cautions (the only point deductions)

| File | Score | Note |
|------|------:|------|
| `skills/kubernetes-specialist/references/multi-cluster.md` | 70 | Contains a `curl|sh` remote-install one-liner (official vendor command, but inherently risky to run blindly) |
| `skills/kubernetes-specialist/references/service-mesh.md` | 70 | Contains a `curl|sh` remote-install one-liner (official vendor command, but inherently risky to run blindly) |
| `skills/security-reviewer/SKILL.md` | 92 | Declares Bash tool access (legitimate capability for this skill) |
| `skills/spec-miner/SKILL.md` | 92 | Declares Bash tool access (legitimate capability for this skill) |

**Details:**
- **`curl \| sh` install one-liners** (kubernetes-specialist references): these are the *official* install commands published by Istio, Linkerd, and Submariner. They are not malicious and point to the correct vendor domains, but piping a remote script straight into a shell is a pattern you should always run with caution / pin a version. Deducted to **70** to flag the pattern, not because of wrongdoing.
- **`allowed-tools: ... Bash`** (`security-reviewer`, `spec-miner`): these skills legitimately request shell access to run security scanners (semgrep, bandit, gitleaks, trivy) and miner tooling. The capability is appropriate to the skill's purpose and is gated by user/authorization checks. Deducted to **92** purely to surface that they can run shell commands.

### ℹ️ Informational (no deduction)
- The docs site (`site/astro.config.mjs`) loads **Google Analytics (gtag)** and **Mermaid from jsDelivr CDN** — normal for a documentation site, only relevant if you self-host the site.
- `site/scripts/sync-content.mjs` performs `fs.writeFileSync` / `unlinkSync` / `rmdirSync`, but strictly within the generated Astro content directory during docs builds — standard generator behavior, not destructive to your system.

---

## Category scores

| Category | Files | Score | Comment |
|----------|------:|------:|---------|
| Skill docs (`skills/`) | 432 | 100 (min 70) | Markdown skill instructions + references. No injection; 4 minor flags above. |
| Workflow commands (`commands/`) | 26 | 100 (min 100) | YAML+MD command defs. Require explicit user confirmation; no exec. |
| Build/validation scripts (`scripts/`) | 5 | 100 (min 100) | Python/shell. Stdlib only, local file ops, no network/exec. |
| Docs site (`site/`, `.astro/`) | 17 | 100 (min 100) | Astro generator + sync script. Benign; uses analytics/CDN. |
| CI/CD & hooks (`.github/`, pre-commit) | 5 | 100 (min 100) | Pinned official actions; standard linters. |
| Docs/specs/research | 37 | 100 (min 100) | Prose only. Clean. |
| Repo config & meta (`.claude*`, `.serena`, root) | 32 | 100 (min 100) | Manifests/config. Clean, references upstream author. |
| Assets | 3 | 100 (min 100) | HTML/JS/PNG for social preview. Benign. |

---

## Methodology & limitations

- **Static analysis only** — no file in the repo was executed. Scanning used pattern matching (ripgrep) plus manual review of every executable/config file and a representative sample of skill docs.
- The per-file appendix score is a heuristic: **100** unless a sensitive pattern was matched. Files scoring 100 were confirmed free of the malicious-indicator patterns listed under *What was checked*; this is high-but-not-absolute assurance for a 557-file corpus.
- Risk from the *skills themselves* is behavioral: a "skill" instructs the AI agent. None here contain hostile instructions, but you should still review any skill before granting it `Bash` and run agent actions with confirmation enabled.

---

## Appendix — Per-file scores (all 557 files)

Files are grouped by top-level directory. Every tracked file is listed.

<details><summary><b>.astro/</b> — 4 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.astro/content-assets.mjs` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.astro/content-modules.mjs` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.astro/content.d.ts` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.astro/types.d.ts` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>.claude/</b> — 5 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.claude/old-commands/legacy/complete-epic.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.claude/old-commands/legacy/complete-sprint.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.claude/old-commands/legacy/create-epic-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.claude/old-commands/legacy/create-implementation-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.claude/old-commands/legacy/execute-ticket.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>.claude-plugin/</b> — 2 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.claude-plugin/marketplace.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.claude-plugin/plugin.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>.github/</b> — 5 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.github/ISSUE_TEMPLATE/claude-issue.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.github/ISSUE_TEMPLATE/new-skill.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.github/workflows/ci.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.github/workflows/release.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.github/workflows/validate.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>.serena/</b> — 6 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.serena/.gitignore` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.serena/memories/project_overview.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.serena/memories/style_and_conventions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.serena/memories/suggested_commands.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.serena/memories/task_completion_checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.serena/project.yml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>assets/</b> — 3 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `assets/capture-screenshot.js` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `assets/social-preview.html` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `assets/social-preview.png` | 100 | Binary/asset (image) — no executable risk |

</details>

<details><summary><b>commands/</b> — 26 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `commands/common-ground/COMMAND.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/common-ground/common-ground.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/common-ground/references/assumption-classification.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/common-ground/references/file-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/common-ground/references/reasoning-graph.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/intake/capture-behavior.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/intake/create-system-description.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/intake/document-codebase.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/approve-synthesis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/approve.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/create-epic-discovery.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/create.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/synthesize-discovery.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/discovery/synthesize.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/execution/complete-ticket.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/execution/complete-ticket.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/execution/execute-ticket.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/execution/execute-ticket.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/planning/create-epic-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/planning/create-implementation-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/planning/epic-plan.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/planning/impl-plan.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/retrospectives/complete-epic.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/retrospectives/complete-epic.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/project/retrospectives/complete-sprint.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `commands/workflow-manifest.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>docs/</b> — 32 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `docs/ATLASSIAN_MCP_SETUP.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/COMMON_GROUND.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/SUPPORTED_AGENTS.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/WORKFLOW_COMMANDS.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/ideas/audit-primary-docs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/ideas/audit-skill-consistency.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/ideas/audit-workflow-docs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/ideas/documentation-site.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/local_skill_development.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/prompts/discovery-for-feature-forge.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/skill-ideas/tarot.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/skill-ideas/the-fool.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/v0.5.0-narrative.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/v0.5.0-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/common-ground.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/discovery-approve.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/discovery-create.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/discovery-phase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/discovery-synthesize.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/execution-complete-ticket.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/execution-execute-ticket.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/execution-phase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/intake-capture-behavior.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/intake-create-system-description.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/intake-document-codebase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/intake-phase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/planning-epic-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/planning-impl-plan.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/planning-phase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/retrospective-complete-epic.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/retrospective-phase.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `docs/workflow/workflow-definition-schema.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>research/</b> — 4 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `research/GITHUB_ISSUE_TREE.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `research/SKILL_TRIGGER_LOGS.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `research/superpowers.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `research/superpowers_research_findings.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>scripts/</b> — 5 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `scripts/migrate-frontmatter.py` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `scripts/test-makefile.sh` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `scripts/update-docs.py` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `scripts/validate-markdown.py` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `scripts/validate-skills.py` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>site/</b> — 13 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `site/astro.config.mjs` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/package-lock.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/package.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/public/.gitignore` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/public/robots.txt` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/scripts/sync-content.mjs` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/components/Header.astro` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/components/SocialIcons.astro` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/content.config.ts` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/content/docs/index.mdx` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/content/docs/privacy.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/src/styles/custom.css` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `site/tsconfig.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>skills/</b> — 432 files, min score 70</summary>

| File | Score | Comment |
|------|------:|---------|
| `skills/angular-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/angular-architect/references/components.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/angular-architect/references/ngrx.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/angular-architect/references/routing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/angular-architect/references/rxjs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/angular-architect/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/references/error-handling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/references/openapi.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/references/pagination.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/references/rest-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/api-designer/references/versioning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/references/adr-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/references/architecture-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/references/database-selection.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/references/nfr-checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/architecture-designer/references/system-design.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/references/authentication-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/references/common-workflows.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/references/confluence-operations.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/references/jira-queries.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/atlassian-mcp/references/mcp-server-setup.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/references/chaos-tools.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/references/experiment-design.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/references/game-days.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/references/infrastructure-chaos.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/chaos-engineer/references/kubernetes-chaos.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/references/design-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/references/go-cli.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/references/node-cli.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/references/python-cli.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cli-developer/references/ux-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/references/aws.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/references/azure.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/references/cost.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/references/gcp.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cloud-architect/references/multi-cloud.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/api-docs-fastapi-django.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/api-docs-nestjs-express.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/coverage-reports.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/documentation-systems.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/interactive-api-docs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/python-docstrings.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/typescript-jsdoc.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-documenter/references/user-guides-tutorials.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/common-issues.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/feedback-examples.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/receiving-feedback.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/report-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/review-checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/code-reviewer/references/spec-compliance-review.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/references/build-tooling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/references/concurrency.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/references/memory-performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/references/modern-cpp.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/cpp-pro/references/templates.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/references/aspnet-core.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/references/blazor.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/references/entity-framework.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/references/modern-csharp.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/csharp-developer/references/performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/references/index-strategies.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/references/monitoring-analysis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/references/mysql-tuning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/references/postgresql-tuning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/database-optimizer/references/query-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/references/common-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/references/debugging-tools.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/references/quick-fixes.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/references/strategies.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/debugging-wizard/references/systematic-debugging.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/deployment-strategies.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/docker-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/github-actions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/incident-response.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/kubernetes.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/platform-engineering.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/release-automation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/devops-engineer/references/terraform-iac.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/references/authentication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/references/drf-serializers.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/references/models-orm.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/references/testing-django.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/django-expert/references/viewsets-views.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/references/authentication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/references/clean-architecture.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/references/cloud-native.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/references/entity-framework.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/dotnet-core-expert/references/minimal-apis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/references/communication-protocols.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/references/memory-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/references/microcontroller-programming.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/references/power-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/embedded-systems/references/rtos-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/async-sqlalchemy.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/authentication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/endpoints-routing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/migration-from-django.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/pydantic-v2.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fastapi-expert/references/testing-async.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/references/acceptance-criteria.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/references/ears-syntax.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/references/interview-questions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/references/pre-discovery-subagents.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/feature-forge/references/specification-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/references/dataset-preparation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/references/deployment-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/references/evaluation-metrics.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/references/hyperparameter-tuning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fine-tuning-expert/references/lora-peft.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/bloc-state.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/gorouter-navigation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/project-structure.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/riverpod-state.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/flutter-expert/references/widget-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/api-design-standards.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/architecture-decisions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/backend-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/common-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/deliverables-checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/design-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/error-handling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/frontend-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/integration-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/fullstack-guardian/references/security-checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/references/ecs-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/references/multiplayer-networking.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/references/performance-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/references/unity-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/game-developer/references/unreal-cpp.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/references/concurrency.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/references/generics.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/references/interfaces.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/references/project-structure.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/golang-pro/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/federation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/migration-from-rest.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/resolvers.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/schema-design.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/graphql-architect/references/subscriptions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/references/jpa-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/references/reactive-webflux.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/references/spring-boot-setup.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/references/spring-security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/java-architect/references/testing-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/references/async-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/references/browser-apis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/references/modern-syntax.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/references/modules.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/javascript-pro/references/node-essentials.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/references/android-compose.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/references/coroutines-flow.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/references/dsl-idioms.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/references/ktor-server.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kotlin-specialist/references/multiplatform-kmp.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/configuration.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/cost-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/custom-operators.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/gitops.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/helm-charts.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/multi-cluster.md` | 70 | Contains a `curl|sh` remote-install one-liner (official vendor command, but inherently risky to run blindly) |
| `skills/kubernetes-specialist/references/networking.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/service-mesh.md` | 70 | Contains a `curl|sh` remote-install one-liner (official vendor command, but inherently risky to run blindly) |
| `skills/kubernetes-specialist/references/storage.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/troubleshooting.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/kubernetes-specialist/references/workloads.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/references/eloquent.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/references/livewire.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/references/queues.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/references/routing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/laravel-specialist/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/references/legacy-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/references/migration-strategies.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/references/refactoring-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/references/strangler-fig-pattern.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/legacy-modernizer/references/system-assessment.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/references/protocol.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/references/python-sdk.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/references/resources.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/references/tools.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/mcp-developer/references/typescript-sdk.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/references/communication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/references/data.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/references/decomposition.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/references/observability.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/microservices-architect/references/patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/references/experiment-tracking.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/references/feature-engineering.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/references/model-validation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/references/pipeline-orchestration.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/ml-pipeline/references/training-pipelines.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/alerting-rules.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/application-profiling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/capacity-planning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/dashboards.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/opentelemetry.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/performance-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/prometheus-metrics.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/monitoring-expert/references/structured-logging.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/authentication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/controllers-routing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/dtos-validation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/migration-from-express.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/services-di.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nestjs-expert/references/testing-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/references/app-router.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/references/data-fetching.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/references/deployment.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/references/server-actions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/nextjs-developer/references/server-components.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/references/aggregation-groupby.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/references/data-cleaning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/references/dataframe-operations.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/references/merging-joining.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/pandas-pro/references/performance-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/references/async-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/references/laravel-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/references/modern-php-features.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/references/symfony-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/php-pro/references/testing-quality.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/references/api-mocking.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/references/configuration.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/references/debugging-flaky.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/references/page-object-model.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/playwright-expert/references/selectors-locators.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/references/extensions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/references/jsonb.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/references/maintenance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/references/performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/postgres-pro/references/replication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/context-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/evaluation-frameworks.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/prompt-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/prompt-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/structured-outputs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/prompt-engineer/references/system-prompts.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/references/async-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/references/packaging.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/references/standard-library.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/python-pro/references/type-system.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/references/chunking-strategies.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/references/embedding-models.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/references/rag-evaluation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/references/retrieval-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rag-architect/references/vector-databases.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/references/active-record.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/references/api-development.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/references/background-jobs.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/references/hotwire-turbo.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rails-expert/references/rspec-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/hooks-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/migration-class-to-modern.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/react-19-features.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/server-components.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/state-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-expert/references/testing-react.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/references/expo-router.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/references/list-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/references/platform-handling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/references/project-structure.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/react-native-expert/references/storage-hooks.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/references/async.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/references/error-handling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/references/ownership.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/rust-engineer/references/traits.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/references/apex-development.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/references/deployment-devops.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/references/integration-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/references/lightning-web-components.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/salesforce-developer/references/soql-sosl.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/references/authentication.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/references/input-validation.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/references/owasp-prevention.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/references/security-headers.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/secure-code-guardian/references/xss-csrf.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/SKILL.md` | 92 | Declares Bash tool access (legitimate capability for this skill) |
| `skills/security-reviewer/references/infrastructure-security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/references/penetration-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/references/report-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/references/sast-tools.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/references/secret-scanning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/security-reviewer/references/vulnerability-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/references/app-development.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/references/checkout-customization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/references/liquid-templating.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/references/performance-optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/shopify-expert/references/storefront-api.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/references/partitioning-caching.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/references/performance-tuning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/references/rdd-operations.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/references/spark-sql-dataframes.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spark-engineer/references/streaming-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spec-miner/SKILL.md` | 92 | Declares Bash tool access (legitimate capability for this skill) |
| `skills/spec-miner/references/analysis-checklist.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spec-miner/references/analysis-process.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spec-miner/references/ears-format.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spec-miner/references/specification-template.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/references/cloud.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/references/data.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/references/security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/spring-boot-engineer/references/web.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/references/database-design.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/references/dialect-differences.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/references/optimization.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/references/query-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sql-pro/references/window-functions.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/references/automation-toil.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/references/error-budget-policy.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/references/incident-chaos.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/references/monitoring-alerting.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/sre-engineer/references/slo-sli-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/references/async-concurrency.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/references/memory-performance.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/references/protocol-oriented.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/references/swiftui-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/swift-expert/references/testing-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/references/best-practices.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/references/module-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/references/providers.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/references/state-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/terraform-engineer/references/testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/automation-frameworks.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/e2e-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/integration-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/performance-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/qa-methodology.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/security-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/tdd-iron-laws.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/test-reports.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/testing-anti-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/test-master/references/unit-testing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/dialectic-synthesis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/evidence-audit.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/mode-selection-guide.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/pre-mortem-analysis.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/red-team-adversarial.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/the-fool/references/socratic-questioning.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/references/advanced-types.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/references/configuration.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/references/patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/references/type-guards.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/typescript-pro/references/utility-types.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/references/component-architecture.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/references/composables-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/references/jsdoc-typing.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/references/state-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert-js/references/testing-patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/build-tooling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/components.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/composition-api.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/mobile-hybrid.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/nuxt.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/state-management.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/vue-expert/references/typescript.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/references/alternatives.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/references/patterns.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/references/protocol.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/references/scaling.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/websocket-engineer/references/security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/SKILL.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/references/gutenberg-blocks.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/references/hooks-filters.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/references/performance-security.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/references/plugin-architecture.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `skills/wordpress-pro/references/theme-development.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>specs/</b> — 1 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `specs/v050-roadmap-consolidation.spec.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

<details><summary><b>(root files)/</b> — 19 files, min score 100</summary>

| File | Score | Comment |
|------|------:|---------|
| `.editorconfig` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.git-blame-ignore-revs` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.gitignore` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.pre-commit-config.yaml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.prettierignore` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `.prettierrc` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `CHANGELOG.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `CLAUDE.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `CONTRIBUTING.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `LICENSE` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `MODELCLAUDE.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `Makefile` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `QUICKSTART.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `README.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `ROADMAP.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `SKILLS_GUIDE.md` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `pyrightconfig.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `ruff.toml` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |
| `version.json` | 100 | Clean — no malicious code, prompt injection, secrets, or exfiltration patterns |

</details>

---

*Report produced by automated security validation. No third-party services received repository contents; analysis was performed locally.*
