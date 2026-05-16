---
name: handoff
description: "Generate an intelligent context handoff document for the next agent. Triggers on: handoff, hand off, pass to next agent, context handoff, next agent, session done, stage done, wrap up for next agent, /handoff"
---

# Handoff — Intelligent Context Handoff for Next Agent

Generate a handoff document that gives the next agent in a fresh conversation the minimum context needed to fully understand the project state and immediately continue work — using the fewest tokens possible.

**This is NOT a template.** Every handoff is different. The structure, length, sections, and language of the handoff doc must be adapted to what actually happened and what comes next. The only thing that's fixed is the investigation process below.

---

## When to Use

- A task or stage is complete and the next step will be done in a new conversation
- The operator says "handoff", "wrap up", "done with this stage", "pass to next agent"
- Context window is getting large and work should continue in a fresh session

---

## The Process

### Phase 1: Trace the Chain (Look Backward)

Before writing anything, investigate:

1. **Find prior handoff docs** — search the project directory for files with "handoff" in the name, or files that look like they served as context transfers between agents. Read them to understand the chain of work.
2. **Find project structure docs** — look for task breakdowns, roadmaps, phase plans, SOPs, CLAUDE.md, PROJECT.md, or any planning documents that define the overall mission and stages.
3. **Find the current task definition** — locate the specific task doc, issue, or plan that was being worked on this session.

If prior handoff docs exist, the new handoff should be written as a continuation — the next agent should be able to read ONLY the new handoff doc (not the old ones) and still have everything it needs.

### Phase 2: Audit This Session (Look at the Present)

Gather concrete facts about what happened:

1. **Git diff / git log** — what files were created, modified, or deleted since the session started (or since the last commit before this session's work). Use `git diff --stat` and `git log --oneline` to get a clear picture.
2. **Key decisions made** — review the conversation for decisions, constraints discovered, trade-offs chosen, things that broke and how they were fixed, architectural choices.
3. **Current state** — does the code build? Are there failing tests? Uncommitted changes? Pending migrations? Anything half-finished?
4. **What was explicitly NOT done** — things that were discussed but deliberately deferred, out-of-scope items, known shortcuts taken.

### Phase 3: Scan Forward (Look at What's Next)

Cross-reference future work:

1. **Identify the next task(s)** — from task breakdowns, roadmaps, or conversation context, determine what the next agent should work on.
2. **Check for ripple effects** — did anything done this session change assumptions, invalidate plans, create new dependencies, or affect future tasks? This is critical — if we changed a database schema, renamed a type, or discovered a constraint, future task docs may now be partially wrong.
3. **Flag stale docs** — if any planning documents are now out of date because of this session's work, explicitly call this out.

### Phase 4: Build the Minimum Reading List

Figure out the smallest set of files the next agent needs to read:

1. **Docs** — only the task definitions, SOPs, or reference docs actually needed for the next step. NOT everything that was read this session.
2. **Code** — only the entry points, types, and modules the next agent will need to touch or understand. Provide absolute paths.
3. **Order them** — put the most important files first. The next agent should be able to stop reading early if context is tight.

### Phase 5: Write the Handoff Doc

Now write the handoff document. Adapt the structure to what's actually needed — don't force sections that have nothing useful to say, and add sections that the specific situation demands.

**General principles:**
- Write for an agent that knows NOTHING about this project — assume zero prior context
- Be precise: file paths must be absolute, type names must be exact, command examples must be copy-pasteable
- Include verification commands the next agent should run before starting (e.g., `git status`, `rg "some_symbol"`, build/lint checks)
- Include constraints and "do NOT touch" rules if applicable
- If the task has sub-steps, specify the exact order
- Use the same language the operator has been using in this session (Chinese, English, or mixed)

**Where to save the file:**
- If there's an existing pattern of handoff docs or task docs in the project, follow that pattern (same directory, similar naming)
- If not, save it in the project root or a planning/docs directory that makes sense
- Name it descriptively — include the stage/task name and "handoff" (e.g., `04-18A-完成-handoff给18B.md`, `handoff-phase2-complete.md`)

### Phase 5.5: Update STATUS.md

Maintain a running progress log at the **project root** as `STATUS.md`. This is the single file a fresh agent reads to orient itself — even without any handoff prompt.

**If `STATUS.md` does not exist**, create it with just a header:

```markdown
# [Project Name] — Status Log
```

**Always append** a new entry at the bottom. Never overwrite or edit previous entries — they are a scrollable history for debugging and review.

```markdown
---

## [YYYY-MM-DD] | [one-line session summary]

**Done this session:**
- [concrete completed item]
- [concrete completed item]

**Current state:**
[Where exactly things stand — what's working, what's half-done, any broken or unstable state]

**Next steps:**
1. [Most immediate action — specific enough to act on]
2. [Following action]
3. [Further if known]

**Decisions / notes:**
- [Non-obvious choices made, constraints discovered, things not to forget]
```

Keep each entry self-contained. Someone reading only the last entry should be able to orient immediately.

### Phase 5.7: Snapshot Commit

After writing the handoff doc and updating STATUS.md, create a snapshot commit so the repo state matches the handoff exactly. This gives the next agent a clean starting point and a restore point if anything goes wrong.

```bash
git add -A && git commit -m "snapshot: before [next task/phase name]"
```

If there are files that should NOT be committed (e.g., `.mcp.json` with API keys, `.env` files), add them to `.gitignore` first. When in doubt, ask the operator before committing.

### Phase 6: Generate the Prompt

After writing the handoff file and updating STATUS.md, output a short prompt block that the operator can copy-paste into a new conversation. This prompt should:

- Point the new agent to the handoff file path (absolute)
- State the core task in 1-2 sentences
- Include any critical constraints in 1-2 sentences
- Tell the agent to read the handoff file first, then follow its instructions
- Be as short as possible — the handoff doc has the details, the prompt is just the entry point

Format it as a fenced code block so the operator can copy it easily.

---

## Fresh Session Orientation

When a new chat opens and the operator asks something like **"帮我看一下这个project进行到什么程度了"** or **"where are we? what's next?"** — without any handoff prompt — do the following:

1. Look for `STATUS.md` in the **project root** (current working directory)
2. Read the **last entry** (scroll to the bottom of the file)
3. Run `git log --oneline -5` to check for any commits since the last STATUS.md entry
4. Present a clear, direct orientation:
   - What was done last session
   - Current state of the project
   - Recommended next steps in priority order
   - Any outstanding blockers or decisions

Do NOT wait for the operator to find the file or point you to it. Finding and reading `STATUS.md` is the default first move when asked about project status.

---

## Anti-Patterns

- **Don't write a conversation transcript** — the next agent doesn't need to know what we discussed, only what was decided and what exists now.
- **Don't include context the next agent can derive from the code** — if something is obvious from reading a file, don't duplicate it in the handoff. Just point to the file.
- **Don't pad with boilerplate sections** — if there are no constraints, don't add an empty "Constraints" section. If there's no schema change, don't add a "Database" section.
- **Don't over-include the reading list** — every extra file costs tokens. Be ruthless about what's truly needed vs. nice-to-have.
- **Don't assume the next agent will read prior handoff docs** — each handoff should be self-contained. If info from a prior handoff is still relevant, carry it forward.
