# Lantern

A Mac app for reading RSS and Atom feeds: subscribe, read, save for later,
dismiss. A local curation layer learns from the reader's signals to rank
items, and Apple's on-device model (Foundation Models framework) writes
summaries and "why this surfaced" explanations. Everything runs on the local
machine.

The primary goal is learning, not product. Two goals, equally weighted:

1. Building *with* agents: the whole project goes through the front-loaded,
   spec-first workflow below.
2. Building an agent *into* the app: a small internal agent with a few tools.

The starting concept is `docs/concept.md`. It is a handoff from a voice
conversation and a starting position, not a set of decisions.

## Status

Nothing is built yet. Spec Kit and EGM are installed and the `/frontload`
workflow is in place, ported from the Z-Touch project on 2026-11-16. It has
not been run here. The constitution at `.specify/memory/constitution.md` is
still the blank template, so the first `/frontload` run starts at Phase 0.
No Xcode project or Swift package exists yet.

## Direction carried over from the concept (not yet decisions)

- Milestone one has no AI: feed fetching, parsing, reading UI, save, dismiss.
- AI hooks are designed in from the start: signals logged as events, full
  article text stored, ranking behind one scoring interface, stable item
  identity across refreshes.
- Plain algorithms do the ranking (embeddings plus a light classifier). The
  on-device model handles summaries, tags, and explanations for the top
  slice only.
- The internal agent stays small.

These become decisions only when a concept brief records them and the user
approves it.

## Development workflow

Spec Kit and EGM are combined into one front-loaded workflow that ends at a
reviewed task list.

| Topic | Decision |
|---|---|
| Record of truth | Spec Kit artifacts under `specs/NNN-name/` (`spec.md`, `plan.md`, `research.md`, `tasks.md`). EGM supplies ideation and review, not a second design doc. |
| Interaction | During ideation: one open question at a time, never guess, argue back. After the concept brief is approved: record assumptions, batch remaining decisions. |
| Research | Done before the spec. Isolated researcher subagents cite sources; a fresh-context reviewer checks the claims before they enter `research.md`. |
| Review depth | Tiered. Lens swarm on the concept brief; comprehension and design critique on the spec; implementation-readiness on plan and tasks. |
| Reviewer lenses | EGM's generic reviewers plus a project-specific Local-first lens. EGM's cloud lenses (quota, on-call, migration) are dropped. |
| Reviewer isolation | A reviewer gets the target file, the constitution, `research.md`, and code the target references. No conversation history, no `CLAUDE.md`, no earlier review reports. Read-only tools. |
| Reviewer authority | A blocker stops the workflow until fixed or explicitly waived by the user; the waiver is recorded in the artifact. Major and nit findings are advisory. |
| Finish line | Tasks that a fresh session can implement with no design decisions left, after `/speckit-analyze`. |

This differs from the process in `docs/concept.md`, which had `/egm-draft`
produce a design doc under `docs/designs/` before Spec Kit. The user chose
the ported workflow on 2026-11-16 so that there is one record, not two.

Rules that follow from this:

- Do not write production code for a feature before its tasks are reviewed.
  Prototypes under `prototypes/` are the exception; see the skill.
- A review run in the main conversation's context is not an independent
  review. Spawn a new subagent each time; never fork or resume one.
- Orchestration and user interviews stay in the main conversation. Only
  research and review are delegated.

### The workflow skill

Start or resume any feature with `/frontload <idea or slug>`
(`.claude/skills/frontload/SKILL.md`). Its parts:

- `.claude/agents/ln-researcher.md`: answers one research question with
  sourced claims.
- `.claude/agents/ln-research-verifier.md`: checks each claim against its
  source before it enters `research.md`.
- `.claude/agents/ln-local-first.md`: the project-specific reviewer lens.
  It checks the local-only promise, what is asked of the small model, and
  agent-versus-plain-code choices.
- `.claude/skills/frontload/standards/spec-standard.md`: the bar for
  `spec.md`. EGM's own feature-spec standard demands architecture, which a
  Spec Kit spec must not contain, so it is applied to `plan.md` instead.

Per-feature state, briefs, raw research, and review reports live in
`docs/egm/<slug>/`.

### Tooling

- **Spec Kit 1.4.0**: skills in `.claude/skills/speckit-*`, templates and
  scripts in `.specify/`.
- **EGM 0.1.1**: plugin `egm@egm`, enabled in `.claude/settings.json`.
  In Z-Touch its skills could not be called through the Skill tool, so the
  workflow reads each `SKILL.md` and follows it. Reviewer agent types are
  `egm:egm-*`.
- **Reviewer isolation has not been checked in this project.** The
  `/frontload` preflight runs the check on first use and records the result
  here.
- **Toolchain** (2026-11-16): macOS 27.2, Xcode 27.2, Swift 6.4.2.

EGM artifact directory: docs/egm
