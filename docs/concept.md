# Handoff: Mac RSS Curator

Handoff from a voice conversation with Chance, November 16, 2026. Open this in Claude Code as the starting context. Everything below is a starting position, not a decision; Chance intends to negotiate details in Claude Code.

Source: <https://claude.ai/artifact/3ujuebTJWiuTmGCj92KbAZ>

## The idea

A Mac app that recreates the Google Reader + Feedly + Pocket workflow: subscribe to RSS/Atom feeds, read, save items for later, dismiss items. The twist is a local curation layer that learns from the user's signals (saves, dismiss-without-reading, reading time) to rank and surface items of interest, with Apple's on-device model (Foundation Models framework, macOS 26) used for summaries and "why this surfaced" explanations.

Runs entirely on the local machine. No cloud services required.

## Primary goal

Learning, not product. Two learning goals, both equally important:

1. Building *with* agents — driving the whole project through an agentic spec-first workflow (see Process below).
2. Building an agent *into* the app — a small internal agent with tools such as fetch feed, parse, embed, score, summarize.

A working app is the nice side effect.

## Agreed direction from the voice conversation

- Build the foundation first: feed fetching, parsing, reading UI, save-for-later, dismiss. No AI in milestone one.
- Design the AI hooks in from the start even though AI lands later.
- Offload as much as possible to plain algorithms. The on-device Apple model is roughly a 3B-parameter model and is not well suited to fuzzy taste-ranking over hundreds of items a day. Expected split: embeddings plus a lightweight classifier for ranking; the Foundation Models API for summarization, tagging, and explanation on the top slice only.
- The inner agent is the place most likely to get over-engineered. Keep it small.

## Hooks to design in early

- Log every user signal as an event (timestamp, item ID, signal type, optional duration) rather than flipping a boolean on the item. Saves, dismisses, opens, read time, scroll depth are all candidates.
- Store full article text where available, not just the feed snippet. Ranking and summarization will need it.
- Put ranking behind a single scoring interface so the first implementation (recency, or nothing) can be swapped for embeddings and a classifier without touching the UI.
- Keep item identity stable across feed refreshes so signals attach correctly.

## Open questions for EGM brainstorm

- Swift/SwiftUI native vs. anything else — Chance's primary stack is Microsoft technologies, so the Swift side is itself a learning curve.
- Storage: SwiftData, Core Data, SQLite directly?
- What exactly is the internal agent responsible for, and what stays as deterministic pipeline code?
- What does "item of interest" mean well enough to score it?
- Full-text fetching: readability-style extraction, and how to handle paywalled or JS-rendered pages.
- OPML import so existing Feedly subscriptions can come across.

## Process

1. **EGM (Google's Elephant-Goldfish Model) for early requirements.** Run `/egm-brainstorm` using this document as the raw concept. Discuss requirements and trade-offs; let the Goldfish swarm stress-test the concept; produce `concepts_brief.md`. Then `/egm-draft` to turn the brief into a design doc under `docs/designs/`, and `/egm-review` for fresh-context review. Resolve findings with `/egm-resolve`.
2. **GitHub Spec Kit for specification.** Feed the approved EGM design into Spec Kit's specify, plan, and tasks stages.
3. **Goldfish-style reviewers inside Spec Kit.** Spec Kit has no built-in reviewers, so reuse EGM's fresh-context Goldfish personas as challenge-and-review agents at each Spec Kit stage (after specify, after plan, after tasks). The exact wiring is to be worked out in Claude Code.

## Notes

- Apple's Foundation Models framework (macOS 26) exposes the on-device model to Swift with structured output and tool calling. It is not Siri.
- Chance has an existing Agent-Ready Repository framework for structuring repos so coding agents work well; consider whether its conventions apply here even though this is Swift rather than .NET.
