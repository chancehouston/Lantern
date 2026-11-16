---
name: frontload
description: Lantern's front-loaded feature workflow. Takes a raw idea through ideation, verified research, a concept brief, and Spec Kit spec, plan, and tasks, with isolated-context reviewers gating each artifact. Use to start or resume any new feature before code is written.
argument-hint: "A feature idea, or the slug of a feature to resume"
---

# Frontload: ideation → research → specification, independently reviewed

This workflow combines GitHub Spec Kit and Google's Elephant-Goldfish Model
(EGM). Spec Kit owns the documents of record. EGM supplies the ideation
discipline and the fresh-context reviewers. It ends when a task list has been
reviewed and a fresh session could implement it without making a design
decision. It never writes production code.

You, the main conversation, are the orchestrator. You hold the context, talk
to the user, and write the documents. Research and review are done by
subagents that do not share your context. Do not run this workflow from
inside a subagent: it has to interview the user.

## Ground rules

1. **No production code.** Illustrative snippets inside a document are fine.
2. **A review in your own context is not a review.** If you cannot spawn a
   fresh subagent, stop and tell the user.
3. **Documents go to files, not chat.** In chat, give the path and at most
   three points the user most needs to see.
4. **Do not flatter and do not agree by default.** Lead with what is weak or
   missing. No apologies; state the corrected reasoning.
5. **Contradictions are noted and discussed.** If a new instruction, or the
   work, goes against the constitution, the concept brief, or an earlier
   decision, name it to the user and discuss which side wins before writing
   the change. A subagent or unattended run that notices one notes it at the
   top of its report and keeps going. Any rule the constitution marks
   as a hard rule is the exception: refuse.
6. **The concept document is not binding.** `docs/concept.md` is the handoff
   that started this project. It is a starting position, not a set of
   decisions. Anything taken from it enters a brief or a spec as a new
   decision the user has confirmed, not as settled fact. Its technical claims
   (about Apple's frameworks, the on-device model, feed formats) are research
   questions until verified.
7. **Plain English** in every document. No promotional adjectives.

## Where things live

`<slug>` is a short kebab-case feature name. `<feature>` is the directory
Spec Kit creates, `specs/NNN-<slug>/`.

| Path | Holds |
|---|---|
| `docs/egm/<slug>/STATE.md` | Phase, decisions, open blockers, waivers. Update at every phase change. |
| `docs/egm/<slug>/concepts_brief.md` | The approved concept. Input to the spec; not shown to spec reviewers. |
| `docs/egm/<slug>/research/` | One raw file per researcher, plus verifier reports. |
| `docs/egm/<slug>/reviews/` | One report per review round. Never shown to any reviewer. |
| `<feature>/research.md` | Verified research only. |
| `<feature>/spec.md`, `plan.md`, `tasks.md` | The record. |

To resume, read `STATE.md` and continue from the recorded phase. Never infer
the phase from which files exist.

## Preflight (every run)

1. **EGM root.** Run `ls -d ~/.claude/plugins/cache/egm/egm/*/ | sort -V | tail -1`
   and use that absolute path as `EGM` below. If it is missing, stop: EGM is
   not installed.
2. **Constitution.** Read `.specify/memory/constitution.md`. If it still
   contains bracketed placeholders such as `[PRINCIPLE_1_NAME]`, there is no
   constitution. Run Phase 0 before anything else.
3. **Isolation.** If `STATE.md` or `CLAUDE.md` does not record a passed
   isolation check for the current Claude Code version, run the check in
   "Isolation check" below once and record the result in `CLAUDE.md`.

## How to interact with the user

Two modes. `STATE.md` records which one is active.

**Ideation mode** (Phases 0 to 2, until the concept brief is approved):

- Exactly one question per turn, open-ended, in plain chat. No multiple-choice
  options and no structured question tool while exploring.
- Never fill a gap with a guess. An unanswered question stays open.
- Challenge the answer you get. Ask what would make it wrong.
- Structured single-select gates are used only for "ready to proceed?"
  decisions.

**Specification mode** (Phases 3 to 5):

- Where a reasonable default exists, take it, and write it under Assumptions.
- Collect open decisions and put them to the user in one batch with the
  structured question tool, up to four at a time.
- Before each review, show the user the Assumptions list for that artifact as
  one batch. An assumption the user has not seen is not accepted.

## How to dispatch a reviewer

Every reviewer is a **new** subagent, spawned with the Agent tool by its
registered type. Never fork, never resume a previous reviewer, never reuse one
across rounds.

Give each reviewer exactly this and nothing more:

```
Mode: <BRAINSTORM | REVIEW>
Target: <absolute path> (full file)
Standard: <absolute path, or "none">
Reading list (the only other context you may use):
- <absolute path to .specify/memory/constitution.md>
- <absolute path to <feature>/research.md, when it exists>
- <absolute paths of source files the target references>
Do not read: anything under docs/egm/, any CLAUDE.md or AGENTS.md, any
STATE.md. Findings listed under "Review Waivers" in the target are settled;
do not raise them again.
Return findings in your persona's output format. End with the list of files
you read.
```

Do not add your own view of the document, what you are worried about, what
the user said, or what earlier reviewers found. That is the context the
reviewer exists to be free of.

Exception: `egm:egm-comprehension` gets the target and the constitution only,
no research and no source files. Its job is to test whether the document
stands on its own.

Reviewers of one stage run in parallel, in a single message. Stages run in
order. A reviewer that fails, times out, or returns without listing the
target among the files it read **did not review**: retry once, then record it
as "not run". A stage with a reviewer not run is not clean.

## How findings are handled (the gate)

After each stage:

1. Write every finding, verbatim, to
   `docs/egm/<slug>/reviews/<artifact>-<stage>-round<N>.md`, with a stable id
   `<reviewer>-<round>-<n>` and each reviewer's status.
2. Classify each finding yourself as **propose fix** (with the exact change)
   or **contest** (with the reason the reviewer is wrong). You may contest. A
   reviewer lacks context and can be mistaken.
3. **Blockers** stop the workflow. Each one ends in exactly one of:
   - **Fixed**: the artifact is changed.
   - **Waived**: only the user can waive. Put each blocker you want to
     contest, and any the user may prefer to accept, to the user. Record a
     waiver in the artifact itself:

     ```
     ## Review Waivers
     - <finding id> (<reviewer>, <date>): <the finding, one sentence>.
       Waived by user: <their reason>.
     ```
   You never waive a blocker yourself, and contesting one is a recommendation
   to the user, not a resolution.
4. **Major and Nit** findings are advisory. Fix the ones that are right, note
   the ones you decline in the review report with one line of reasoning, and
   show the user the declined Majors in one batch. Do not ask about Nits.
5. **Re-audit.** If any blocker was fixed, run the stage once more with new
   reviewers. One re-audit per stage. If blockers remain after it, do not loop:
   give the user the choice to fix by hand, waive, or return to an earlier
   phase.
6. Record the outcome in `STATE.md` and move on only when no blocker is open.

EGM tells its users to expect a lot of feedback and to ignore much of it.
Keep the volume away from the user: they see blockers, declined Majors, and
the path to the full report.

---

## Phase 0: Constitution (once per project)

Ideation mode. The constitution is read by every reviewer on every pass, so
errors in it are copied into everything.

1. Interview the user, one question at a time, about what this project must
   always and never do. Draft from the interview, not from another
   project's constitution. The primary goal here is learning, so ask what
   the process itself must and must not do, not only the app.
2. Draft the principles and invoke the `speckit-constitution` skill with them.
3. Review it: dispatch `egm:egm-zoom-out` (standard
   `EGM/resources/document_standards/zoom-out-standard.md`) and
   `egm:egm-comprehension` on the constitution, with an empty reading list.
4. Apply the gate. The user ratifies the result explicitly.

Only the user amends the constitution. Draft the amendment and wait for
approval. Record it with the question it answered, its reason, and the
documents written under the old wording. Those documents are not re-opened;
they are reviewed at the next retrospective.

## Phase 1: Ideation

Ideation mode.

1. Create `docs/egm/<slug>/STATE.md`.
2. Drill into the idea, one question per turn, until all of these are
   answered in the user's own words: what problem and for whom; what the
   reader (the person using the app) can do afterwards that they cannot now;
   hard constraints; what is out of scope; how it is handled today and why
   that fails; how success will be measured.
3. As you go, keep a list of **research questions**: every point where the
   answer depends on a fact about feed formats, Apple's frameworks (SwiftUI,
   storage, Foundation Models), what the on-device model can and cannot do,
   or what other feed readers have already tried. Do not answer these from
   memory: the Apple frameworks in particular change every release.
4. State the working hypothesis in two or three sentences and present one
   single-select gate: proceed to research, or keep clarifying. Do not spawn
   anything before the user chooses to proceed.

## Phase 2: Research, then concept brief

Still ideation mode.

**Research**

1. Show the user the research questions; let them add, drop, or reword.
2. Dispatch one `ln-researcher` per question, in parallel. Each task contains
   the question, any paths or topics that are off limits, and any local
   paths worth reading. It contains no hypothesis and no preferred
   answer: a researcher handed a conclusion tends to return it.
3. Save each result verbatim to `docs/egm/<slug>/research/q<N>.md`.
4. Dispatch a new `ln-research-verifier` per result file, in parallel, with
   the file's absolute path, the same off-limits list, and a note of which claims
   the hypothesis leans on hardest. Save each report beside the result.
5. Write `docs/egm/<slug>/research/verified.md`:
   - **Confirmed** claims, using the verifier's corrected wording where given.
   - **Unverified** claims (not supported, could not check), kept separate and
     labelled.
   - **Contradicted** claims, with both sources.
   Only confirmed claims may be relied on in any later document. If the
   hypothesis rested on something not confirmed, tell the user before going
   on; that is a finding about the idea, not a detail.

**Concept brief**

6. Write `docs/egm/<slug>/concepts_brief.md` with EGM's structure: Overview;
   Analysis; Goals vs. Non-Goals; Alternatives Weighed (each with the concrete
   reason it was rejected); Key Decisions. Cite confirmed claims by id.
7. Lens swarm, `Mode: BRAINSTORM`, in parallel, target the brief, reading
   list the constitution and `verified.md`:
   - `egm:egm-zoom-out`: does this need to exist; what already solves it.
   - `egm:egm-structural`: coupling and long-term maintenance.
   - `egm:egm-style`: vague framing and loose scope.
   - `egm:egm-inventive-catalyst`: trade-offs accepted that need not be.
   - `ln-local-first`: the reader whose data stays on their Mac and whose
     only model is a small on-device one.

   Do not use `egm:egm-design`, `egm:egm-codebase-details`, or
   `egm:egm-implementation-readiness` in BRAINSTORM mode. In that mode their
   personas ask about cloud quotas, on-call telemetry, and data migration,
   which do not apply here. They are used later in REVIEW mode.
8. Apply the gate. Discuss objections with the user one at a time. If the
   concept changes materially, rewrite the brief and re-run the swarm once.
9. The user approves the brief explicitly. Record it in `STATE.md` and switch
   to specification mode.

## Phase 3: Spec

Specification mode.

1. Invoke the `speckit-specify` skill. Pass the brief's content as the
   feature description and state that decisions in the brief are settled,
   that facts must come from confirmed research claims, and that anything
   guessed goes under Assumptions.
2. Copy `verified.md` to `<feature>/research.md`. Record `<feature>` in
   `STATE.md`.
3. Invoke `speckit-clarify`. Its questions go to the user as a batch.
4. Show the user the spec's Assumptions list as one batch.
5. **Stage A, comprehension**: `egm:egm-comprehension`, `Mode: REVIEW`, no
   standard. Save its full output, including the paraphrase. Compare the
   paraphrase with the approved brief yourself: a paraphrase that differs from
   the intent is a Blocker even if the reviewer listed no findings. Apply the
   gate.
6. **Stage B, design critique**, in parallel, `Mode: REVIEW`, standard
   `.claude/skills/frontload/standards/spec-standard.md`:
   - `egm:egm-design`
   - `ln-local-first`

   Apply the gate.

## Phase 4: Plan

1. Invoke `speckit-plan`. Tell it `<feature>/research.md` already holds
   verified research: keep it, and add new findings under a separate
   "Planning research" heading. Any new factual claim the plan depends on goes
   through a `ln-research-verifier` before the review.
2. Show the user the plan's technology choices and assumptions as one batch.
   The plan is where alternatives are recorded: each significant choice names
   what was rejected and why.
3. **Review**, in parallel, `Mode: REVIEW`, target `plan.md`, reading list
   plus `<feature>/spec.md` and any `data-model.md` or `contracts/`:
   - `egm:egm-design`, standard
     `EGM/resources/document_standards/feature-spec-standard.md`
     (architecture, alternatives, risks are judged here, not in the spec).
   - `egm:egm-implementation-readiness`, standard
     `EGM/resources/document_standards/implementation-readiness-standard.md`.
   - `ln-local-first`, no standard, only if the plan touches networking,
     storage, signals, ranking, or the on-device model.

   Apply the gate.

## Phase 5: Tasks and handoff

1. Invoke `speckit-tasks`.
2. **Review**, in parallel, `Mode: REVIEW`, target `tasks.md`, reading list
   plus `spec.md` and `plan.md`, standard
   `EGM/resources/document_standards/implementation-readiness-standard.md`:
   - `egm:egm-implementation-readiness`
   - `egm:egm-codebase-details`, only once the repository contains source
     code for it to check paths and reuse against.

   The bar is the zero-decision rule: if an implementer would have to choose
   a schema, an interface, a dependency, or how to treat an edge case, that is
   a Blocker. Apply the gate.
3. Invoke `speckit-analyze`. Treat its CRITICAL findings as blockers under the
   same gate.
4. Write the handoff into `STATE.md`: phase `ready-to-implement`, the feature
   path, every waiver in force, every reviewer recorded as not run, and any
   unverified research the documents mention.
5. Tell the user the feature is ready and stop. Implementation is a separate
   session, started with `speckit-implement`, so that the implementer works
   from the documents rather than from this conversation.

## Prototypes

A prototype is throwaway code written to answer one question. It is not
production code, so it may be written before a feature's tasks are reviewed.

- State the question in one sentence before writing anything.
- Keep it under `prototypes/<name>/`. Production code never imports from
  there.
- When it has answered the question, write the answer to
  `docs/egm/<slug>/prototypes/<name>.md`: what was run, what was observed,
  and what it does not show.
- A fact observed by running a prototype counts as verified for exactly what
  was run.
- A prototype the user tries by hand ends with the user's own verdict, in
  their words.

## Retrospective (end of each feature)

Held with the user after the feature is implemented and working. Before
Phase 1 of a new feature, check that the previous feature's retrospective
was held; if not, say so.

1. Collect: principles amended since the last retrospective and the
   documents written under their old wording; contradictions noted;
   provisional principles; the Decisions list in `STATE.md`.
2. Dispatch a new `egm:egm-inventive-catalyst`, `Mode: REVIEW`, target the
   feature's `spec.md`, reading list the constitution and `plan.md`, to find
   decisions that are constraining the work.
3. Put the results to the user, leading with decisions. Amendments follow
   Phase 0.
4. Record the outcome in `STATE.md`.

---

## Isolation check

Run when Preflight asks for it. Spawn `egm:egm-comprehension` with a task
that forbids tool use and asks it to quote any project instruction file
content, memory, or earlier conversation present in its context, or answer
NONE. Put nothing identifying in the task itself.

- All NONE: record "isolation check passed, <date>, <version>" in `CLAUDE.md`.
- Anything quoted: reviewers are seeing more than intended. Tell the user
  what leaked. Reviews may continue only if the user accepts that, and every
  review report produced meanwhile says so at the top.

## STATE.md format

```
# <slug>
Phase: <0-constitution | 1-ideation | 2-research | 2-brief | 3-spec | 4-plan | 5-tasks | ready-to-implement>
Mode: <ideation | specification>
Feature dir: <path or "not yet created">
Updated: <date>

## Decisions
- <date>: <decision, in the user's words where possible>

## Open blockers
- <finding id>: <one line>

## Waivers
- <finding id>: <one line> (reason)

## Reviewers not run
- <stage>: <reviewer> (why)
```
