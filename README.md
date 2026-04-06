# code-tour

[![✅ Merged — Everything Claude Code](https://img.shields.io/badge/%E2%9C%85%20Merged-Everything%20Claude%20Code-green?style=flat-square)](https://github.com/affaan-m/everything-claude-code/blob/main/skills/code-tour/SKILL.md)
[![✅ Merged — alirezarezvani/claude-skills](https://img.shields.io/badge/%E2%9C%85%20Merged-alirezarezvani%2Fclaude--skills-green?style=flat-square)](https://github.com/alirezarezvani/claude-skills)

AI-generated [CodeTour](https://github.com/microsoft/codetour) walkthroughs for any repository.

A skill that reads your codebase and produces persona-targeted `.tour` files — step-by-step, file-linked guides that open directly in VS Code.

## Install

### Claude Code
```bash
npx skills add vaddisrinivas/code-tour
mkdir -p ~/.claude/skills && ln -sf ~/.agents/skills/code-tour ~/.claude/skills/code-tour
```

The second line is needed because `npx skills add` installs to `~/.agents/skills/` and Claude Code reads from `~/.claude/skills/`. The symlink links them permanently — any future updates via `npx skills add` are picked up automatically.

Then restart Claude Code and run `/skills` to confirm it appears.

### Cursor / Cline / Copilot / Windsurf / any skills.sh-compatible agent
```bash
npx skills add vaddisrinivas/code-tour
```
skills.sh supports 40+ AI agents. Same command, works across all of them.

### Claude.ai (web)
1. Open [Claude.ai](https://claude.ai) → **Projects** → your project
2. Click **Project instructions**
3. Copy the contents of [`skills/code-tour/SKILL.md`](./skills/code-tour/SKILL.md) and paste it in

### ChatGPT / CustomGPT
1. Go to [chat.openai.com](https://chat.openai.com) → **Explore GPTs** → **Create**
2. Paste the contents of `SKILL.md` into the **Instructions** field
3. Save as a private GPT named "CodeTour Generator"

### Any AI with a system prompt
Copy `skills/code-tour/SKILL.md` into the system prompt. Works with any model that can read files — the skill uses no tool-specific APIs, just reads files and writes JSON.

---

## What it does

Most documentation fails because it's written for everyone, which means it works for no one. A new joiner needs orientation. A bug fixer needs a causality chain. A PR reviewer needs the change story. An architect needs module boundaries.

This skill generates the tour that the right person would wish existed when they first opened a repo. You tell it who you're writing for — it reads the code and writes the walkthrough.

---

## Usage

Once installed, just describe what you want:

```
create a new joiner tour for this repo
tour for PR #456
quick vibe check of the codebase
why did the login flow break — bug tour
architecture deep dive
security review tour focusing on auth
```

One message is enough. The skill infers persona, depth, and focus from what you say.

---

## 20 personas

| Persona | What the tour covers |
|---------|----------------------|
| **New joiner** | Directories, setup, business context, team terms |
| **Vibecoder** | Entry point, main flow, what to ignore — max 8 steps |
| **Bug fixer** | User action → trigger → fault points → tests |
| **RCA investigator** | Causality chain, state transitions, observability anchors |
| **PR reviewer** | Change story + unchanged files that must be verified |
| **Architect / tech lead** | Module boundaries, design tradeoffs, future evolution |
| **Security reviewer** | Auth flow, input validation, sensitive sinks |
| **Reboarding engineer** | What changed since you last worked on this |
| **Feature explainer** | One feature: UI → API → backend → storage |
| **Concept learner** | Pattern → implementation → rationale → tradeoffs |
| **Maintainer** | Architectural intent, extension points, invariants |
| **Refactorer** | Seams, hidden deps, safe extraction order |
| **Performance optimizer** | Hot paths, N+1, I/O, caches, benchmarks |
| **Test writer** | Behavior contracts, mocking seams, coverage gaps |
| **API consumer** | Public surface, auth, error semantics, stable vs internal |
| **Platform engineer** | Boot sequence, config, infra deps, graceful shutdown |
| **Data engineer** | Event emission, schemas, source-to-sink lineage |
| **AI agent operator** | Stable anchors, allowed edit zones, validation steps |
| **Product engineer** | Business rules, domain language, feature toggles |
| **External contributor** | Safe areas, conventions, landmines to avoid |

---

## Depth levels

| Level | Steps | Use for |
|-------|-------|---------|
| `quick` | 5–8 | Vibecoder, fast orientation, external contributors |
| `standard` | 10–15 | Most personas (default) |
| `deep` | 15–25 | Architects, concept learners, full onboarding series |

---

## Tour types supported

All [CodeTour](https://github.com/microsoft/codetour) step types:

- `file + line` — point to a specific line
- `selection` — highlight a block of code
- `directory` — orient to a module
- `pattern` — match by regex (resilient to line number shifts)
- `uri` — link to a PR, issue, RFC, ADR, or external doc
- `view` — auto-focus a VS Code panel (terminal, explorer, scm)
- `commands` — execute VS Code commands on arrival

Tour-level fields: `ref` (pin to a git commit/branch/tag), `isPrimary`, `nextTour` (chain tours into a series), `when` (conditional display), `stepMarker`.

---

## Customization

| You say | What happens |
|---------|-------------|
| `cover src/auth.ts and config/db.yml` | Those files are required stops |
| `pin to v2.3.0` | Sets `"ref": "v2.3.0"` |
| `link to PR #456` | Adds a `uri` step at the right narrative moment |
| `lead into the security tour` | Sets `"nextTour": "Security Review"` |
| `make this the main onboarding tour` | Sets `"isPrimary": true` |
| `open terminal at this step` | Adds `"commands": ["workbench.action.terminal.focus"]` |

---

## Tour gallery

Pre-generated tours for popular repos — browse the files on GitHub:

| Repo | Tour | File |
|------|------|------|
| [expressjs/express](https://github.com/expressjs/express) | New Joiner — 17 steps | [view](./tours/expressjs--express/new-joiner.tour) |
| [tiangolo/fastapi](https://github.com/tiangolo/fastapi) | Vibecoder — 7 steps | [view](./tours/tiangolo--fastapi/vibecoder.tour) |
| [trpc/trpc](https://github.com/trpc/trpc) | Architect — 15 steps | [view](./tours/trpc--trpc/architect.tour) |

To use a tour: copy the `.tour` file into the `.tours/` directory of a local clone of that repo, then open it in VS Code with the CodeTour extension.

---

## Bundled tools

### Validator

Checks a `.tour` file before you share it:

```bash
python ~/.agents/skills/code-tour/scripts/validate_tour.py .tours/my-tour.tour --repo-root .
```

Validates: JSON, file paths, line numbers, pattern matches, directory existence, `nextTour` cross-references, narrative arc.

### Skeleton generator

When you want to start from the repo's documentation:

```bash
python ~/.agents/skills/code-tour/scripts/generate_from_docs.py \
  --persona new-joiner \
  --output .tours/skeleton.tour
```

Extracts file/directory references from README and docs, generates a `[TODO: ...]` skeleton, then the skill fills it in by reading the actual files.

---

## Sharing tours

For any public repo with `.tours/` files committed, anyone can open the tour in a browser — no install needed:

```
https://vscode.dev/github.com/<owner>/<repo>
```

VS Code Web detects the `.tours/` directory and offers to start the tour automatically.

---

## References bundled with the skill

- `references/codetour-schema.json` — the official CodeTour JSON schema
- `references/examples.md` — 8 annotated real-world tours with technique notes
- GitHub search for all public `.tour` files: https://github.com/search?q=path%3A**%2F*.tour+&type=code
