<div align="center">

# I am not useless _(i-am-not-useless)_

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Agent-Agnostic](https://img.shields.io/badge/Agent-Agnostic-blueviolet)](https://skills.sh)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)
[![Standard-Readme](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=badge)](https://github.com/RichardLitt/standard-readme)

**Treat the user as an active source of context and decisions, not a passive command emitter.**

A portable [Agent Skills](https://agentskills.io/specification) package that stops an agent from guessing when the user is the only available source of reality. It defines when to ask, how to ask, what counts as evidence, and when to stop and let the user decide.

</div>

English | [简体中文](README.zh.md)

## Quick Install

```
npx skills add NaivG/i-am-not-useless
```

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Behavior Contract](#behavior-contract)
- [Maintainers](#maintainers)
- [Thanks](#thanks)
- [Contributing](#contributing)
- [License](#license)

## Background

Agents fail in two opposite ways: they interrogate the user with a full reproducibility form, or they silently invent what the user could have answered in five seconds. Both treat the user as an unreliable input device.

This skill encodes the opposite premise: **users aren't entirely unreliable; they simply don't know how to express professionally.** The agent infers what is needed, accepts whatever evidence the user can actually produce — screenshots, photos, logs, command output, a single sentence, "A or B" — and converges on reality instead of on its own previous narrative.

It also fixes where the agent has no business deciding: cost vs. performance, workaround vs. proper fix, compatibility vs. breaking change. Those are presented as trade-offs and handed back to the user.

Equally, it fixes where the agent has no business asking: the working directory, the file encoding, a reversible format default, a detail that does not change the next step. Every question spends the user's attention, so a question the agent could have answered itself is a defect, not diligence.

Evaluated behavior, in short: one blocking question per round, no question about anything a tool can read, detect, or safely default, every question carrying an exit (`idk` / `skip` / `you decide`), irreversible actions confirmed with object, scope, and rollback.

## Install

The skill package itself contains no code, only instruction text.

### Recommended: `skills` CLI

The standard installer for the open Agent Skills ecosystem, covering 75+ agents. Install it directly using the skills CLI:

```sh
npm i -g skills@latest
npx skills add NaivG/i-am-not-useless
```

Add `-g` to install for every project on this machine (default is current project only), `-a` to target specific agents, `-y` to skip interactive prompts:

```sh
npx skills add NaivG/i-am-not-useless -g -a claude-code -a codex -y
npx skills add NaivG/i-am-not-useless --list   # list skills without installing
```

### Agent Install

Talk directly to your existing agent:

```prompt
Install the [i-am-not-useless](https://github.com/NaivG/i-am-not-useless) skill for the current framework.
```

### Manual Clone

The folder name must stay `i-am-not-useless`, because the Agent Skills spec requires the directory name to match the `name` field in `SKILL.md`:

```sh
# Project scope
mkdir -p .claude/skills && cd .claude/skills
git clone https://github.com/NaivG/i-am-not-useless i-am-not-useless

# Global scope
mkdir -p ~/.claude/skills && cd ~/.claude/skills
git clone https://github.com/NaivG/i-am-not-useless i-am-not-useless
```

For Codex, clone the same way into `${CODEX_HOME:-$HOME/.codex}/skills/`; the `agents/openai.yml` interface file is read from the skill folder.

### Updating

Run `npx skills update i-am-not-useless`, or `git pull` inside a manually cloned folder. The skill carries no state, so an update only replaces instructions.

## Usage

This skill activates **only when you invoke it explicitly**; the agent will not load it on its own. Invoke it by name when a task depends on something only the user can observe or decide:

```text
/i-am-not-useless  The printer stopped responding. Diagnose it.
/i-am-not-useless  Which migration path should we take — downtime or dual-write?
```

Then expect the shape of the interaction to change, not the tone:

- The agent resolves what it can first, then asks the smallest remaining question — and never asks for what a tool can read, detect, or safely default.
- Questions are answerable in plain language, and state what kind of answer is useful.
- Multimodal evidence is first-class: screenshots, photos, logs, terminal output, configs.
- Requests for evidence carry a redaction reminder, scoped narrow enough to be easy to redact.
- Irreversible actions are confirmed before execution, with object, scope, and rollback.
- User corrections are treated as evidence, not as attacks to be argued down.

The loop it runs:

```text
observe → infer → act → verify → enough info? → yes: continue
                                    no: smallest missing fact → ask
                                        → receive evidence → update model → continue
```

## Behavior Contract

The rules the file actually holds the agent to, with their observable consequence:

| Rule | Observable consequence |
| --- | --- |
| Ask late, ask decisively | Reversible work completes before the first question. |
| Don't ask what you can resolve | The directory, the encoding, the tool output, and the safe default are never requested. |
| One blocking question per round | Coupled sub-questions count as one; independent lists do not. |
| Every question has an exit | `idk`, `skip`, `you decide` are all handled, never re-asked. |
| Irreversible actions ask first | Deletion, payment, production, hardware, private data. |
| Evidence over assumptions | Docs, tool output, and user reports weigh the same until they converge. |
| No assumption-fossilizing tests | Invariants are tested; exact legacy output is not promoted to a requirement. |
| No hallucinated documentation | An unreachable document is requested, never invented. |
| Professionalism is not verbosity | A short request that says why, what, how, and what happens next. |

## Maintainers

[NaivG](https://github.com/NaivG).

## Thanks

- Agent Skills format: [agentskills.io](https://agentskills.io/specification)
- Claude Code's skill loading: [Anthropic](https://code.claude.com/docs/en/skills)
- `skills` CLI and its discovery conventions: [Vercel Labs](https://github.com/vercel-labs/skills) / [skills.sh](https://www.skills.sh/docs/cli)
- README spec: [Standard Readme](https://github.com/RichardLitt/standard-readme)

## Contributing

Questions and failure reports are all welcome in [GitHub Issues](https://github.com/NaivG/i-am-not-useless/issues). Pull requests are accepted; small, focused ones are merged fastest.

Before opening a PR: keep `SKILL.md` under 500 lines, keep frontmatter valid for the Agent Skills spec, and, as much as possible, follow the Markdown style already in the file.

## License

MIT © NaivG. See [LICENSE](LICENSE).