# Spec Standard (Lantern)

The quality bar for a Spec Kit `spec.md` in this project. It replaces EGM's
`feature-spec-standard.md` for the spec, because a Spec Kit spec deliberately
contains no architecture. Architecture and alternatives are judged later, in
`plan.md`, against EGM's own standard.

## Core principle

A spec passes if a competent engineer or a fresh AI session, unfamiliar with
this project, can read it alone and state what will be true for the reader
(the person using the app) when the work is done, what is deliberately left
out, and how each claim would be checked, without asking a question.

## Must contain

- **Problem**: who has it and what they cannot do today.
- **Goals and non-goals**: non-goals are explicit, not implied by silence.
- **User stories**, each independently testable and prioritised.
- **Functional requirements**, each one testable by observation. A requirement
  that cannot fail a test is not a requirement.
- **Edge cases**, including malformed or changing feeds, items that reappear
  after a refresh, no network, and the on-device model being unavailable.
- **Success criteria** that are measurable and say how they are measured.
- **Assumptions**, each one something the user has seen and accepted.
- **Risks and limits**: what this will not do well, and for which feeds or
  articles.

## Must not contain

- Technology choices, languages, frameworks, file names, or data structures.
  Those belong in the plan. Naming RSS, Atom, OPML, and "an on-device model"
  is not a technology choice here; it is the subject matter.
- Requirements that only restate a goal ("ranking surfaces interesting
  items").
- Facts about feed formats, Apple's frameworks, the on-device model, or other
  products that do not trace to a confirmed claim in `research.md`.
- More than three open `[NEEDS CLARIFICATION]` markers at review time.

## Consistency

- Nothing in the spec contradicts the constitution. A contradiction is a
  Blocker regardless of how small it looks.
- Every requirement serves a stated goal; every goal has at least one
  requirement.
- Anything listed under Review Waivers is settled. Do not raise it again.
