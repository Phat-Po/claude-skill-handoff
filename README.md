# handoff — Intelligent Context Handoff for Claude Code

> A Claude Code skill that lets you switch sessions without losing context.
> Generates a minimum-token handoff doc for the next agent and maintains a project-root `STATUS.md` log so a fresh session can orient itself in seconds.

---

## What it does

When your Claude Code session ends — either because the context window is full, the task stage is done, or you just want to switch agents — this skill produces:

1. A **handoff document** the next agent reads to fully understand the project state and immediately continue work, with the smallest possible token cost.
2. A running **`STATUS.md`** log at the project root that any fresh agent can read to orient itself — even without a handoff prompt.
3. A **snapshot commit** so the repo state matches the handoff exactly (clean starting point + restore point).
4. A **copy-pasteable prompt** to drop into the next conversation.

It's not a template. The skill investigates what actually happened, figures out the minimum reading list, and writes a doc adapted to the specific situation.

---

## Why

If you've ever:

- Spent the first 10 minutes of a new Claude Code session re-explaining the project
- Run out of context mid-task and didn't know how to hand off
- Lost a debugging conclusion because you closed the chat
- Asked Claude "where were we?" and gotten a hallucinated answer
- Tried to onboard a teammate (or another agent) onto a mid-flight project

…then this skill is for you.

It treats session boundaries as a first-class engineering problem — not a hope-it-works moment.

---

## The killer feature: `STATUS.md` auto-maintenance

The skill maintains a single file at the **project root** called `STATUS.md`. Every handoff appends a new entry (never overwrites — entries are a scrollable history).

When a fresh chat opens and you ask **"where are we?"** or **"帮我看一下这个项目进行到什么程度了"** — the skill's Fresh Session Orientation kicks in:

1. Auto-reads `STATUS.md` from the project root
2. Reads the last entry
3. Runs `git log --oneline -5` to catch any commits since
4. Presents a clean orientation: what was done last session, current state, recommended next steps

No handoff prompt needed. The skill becomes the default first move when asked about project status.

---

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/phat-po/claude-skill-handoff ~/.claude/skills/handoff
```

That's it. The skill is now available.

To verify, in any Claude Code session type `/handoff` or just say "wrap up for next agent" — it should trigger.

---

## Usage

The skill triggers on any of these:

- The slash command `/handoff`
- Phrases like:
  - `handoff`
  - `hand off`
  - `pass to next agent`
  - `context handoff`
  - `next agent`
  - `session done`
  - `stage done`
  - `wrap up for next agent`
- Fresh-session orientation: `"where are we?"` / `"what's next?"` / `"帮我看一下这个项目进行到什么程度了"` automatically reads `STATUS.md`

### Typical flow

```text
You:    "handoff please, context is getting tight"

Claude: [investigates: prior handoffs, git diff, current task, ripple effects]
        [writes handoff doc with minimum reading list]
        [appends entry to STATUS.md]
        [git commit -m "snapshot: before <next task>"]
        [outputs copy-pasteable prompt for new chat]

You:    [open new chat]
        [paste the prompt]

NextClaude: [reads only the handoff doc]
            [orients immediately, picks up where last session ended]
```

---

## Example

See [`examples/handoff-example.md`](./examples/handoff-example.md) for a real (sanitized) handoff document, and [`examples/STATUS-example.md`](./examples/STATUS-example.md) for a sample `STATUS.md` log.

---

## How it works (5-phase investigation)

The skill is deterministic about *process*, not *output*. It runs through:

| Phase | What it does |
|---|---|
| 1. Look Backward | Find prior handoff docs, project structure docs (CLAUDE.md, roadmaps), current task definition |
| 2. Look at Present | `git diff --stat`, `git log --oneline`, decisions made, current state, what was deliberately NOT done |
| 3. Look at Next | Identify next task(s), check for ripple effects from this session, flag stale docs |
| 4. Build Reading List | Smallest set of files the next agent needs — ranked, absolute paths, stop-early friendly |
| 5. Write Handoff Doc | Adapted structure (no boilerplate sections), then update `STATUS.md`, then snapshot commit, then output prompt |

The full SKILL.md is in the repo root — read it for the complete spec.

---

## Anti-patterns it avoids

- Conversation transcripts (the next agent doesn't need to know what you discussed)
- Padding with boilerplate sections (empty "Constraints", "Database", etc.)
- Over-including the reading list (every extra file costs tokens)
- Assuming the next agent reads prior handoffs (each handoff is self-contained)

---

## Compatibility

- **Claude Code** — primary target
- **Codex CLI** — works (it reads skills from the same `~/.claude/skills/` path if symlinked, or you can adapt the SKILL.md frontmatter)
- **Other Claude-compatible agents** — the SKILL.md uses standard skill frontmatter; should work anywhere the skill format is supported

---

## License

MIT — see [LICENSE](./LICENSE).

---

## Background

Built and battle-tested across many vibecoding projects by [Pohan](https://github.com/phat-po).

Open-sourced after the **2026-05-16 vibecoding Hackathon Bottleneck Wall** session, where multiple participants wrote down the same pain point on a sticky note:

> "一个大任务跑到一半 token 就烧到 80% 了，AI 开始变笨或者直接断掉，我也不知道怎么把进度交给下一个 session 继续"
>
> "每次开新对话都要重新跟 AI 解释一遍项目背景，昨天花三小时 debug 出来的结论它今天全忘了"
>
> "The previous context doesn't get carried through the next conversation all the time."

This skill is the answer.
