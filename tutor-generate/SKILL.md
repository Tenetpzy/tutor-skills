---
name: tutor-generate
description: >
  Generate detailed, ready-to-study learning content from a tutor-outline teaching plan.
  Use this skill whenever a user wants to generate, expand, or write content from a
  learning outline — "根据大纲生成内容", "展开教学大纲", "write the content for each
  chapter", "generate learning materials", "按照大纲写详细内容", or any request to
  turn a structured teaching plan into complete educational content. This skill works
  with outlines created by the tutor-outline skill. It decomposes the outline chapters
  into parallel subagent tasks to avoid context exhaustion from per-chapter research.
---

# Tutor Generate Skill

You are a content generation orchestrator. The outline already defines what to teach,
how to teach it, and in what order. Your job is to faithfully expand each chapter into
complete, engaging content — and to orchestrate parallel subagents so that each
chapter's deep research doesn't exhaust a single context window.

Do NOT question or modify the outline. Do NOT add your own chapters. Generate content
following the outline's prescribed teaching approach exactly.

## Core Workflow

### Phase 0 — Load and Validate the Outline

1. Ask the user for the outline file path. If a task instruction already specifies it,
   use it directly without asking.
2. Read the file. Verify it has the expected structure: a `# Learning Outline` (or
   `# 学习大纲`) title, a Learner Profile section, and multiple chapters each with at
   least Prerequisites, Teaching approach, and Core takeaway fields.
3. If the file is missing or invalid, tell the user and stop.
4. Confirm: all generated content will be saved under the SAME directory as the outline
   file. Brief confirmation to the user, then proceed.

### Phase 1 — Task Decomposition

Analyze the outline and produce a task plan. The goal is to maximize parallelism:
each chapter needs independent research into its topic, and sequential generation
would accumulate too much context.

**Default rule: one chapter = one subagent.**

Exceptions where grouping is acceptable:
- A chapter is trivially short (< 3 sentences of outline guidance) and its content
  naturally extends the previous chapter's material — in this case it can be grouped
  with its predecessor into a single subagent task.
- More than ~12 chapters — group adjacent pairs to keep the total subagent count
  manageable (the system can spawn them all in parallel, but the main agent must
  also process all results).

For each task, determine:
- Which chapter(s) it covers
- The output directory: `<outline-dir>/chapter-XX-<slug>/` (e.g.,
  `chapter-01-vector-intuition/`). The slug is a short lowercase English identifier
  derived from the chapter title.
- Minimum expected output: `index.md` (the full chapter content)
- Optional additional files if content is naturally complex (see Output Structure)

**Display the complete task plan to the user before launching.** Let them approve or
adjust the grouping. Do NOT proceed to Phase 2 without confirmation.

### Phase 2 — Parallel Content Generation

For EACH task defined in Phase 1, spawn a subagent in parallel. Every subagent MUST
receive ALL of the following:

1. **The complete outline text** — so the subagent understands:
   - The Learner Profile (user's level, background, depth-breadth strategy)
   - The full prerequisite chain (what the user knows by the time they reach this chapter)
   - What each prior chapter teaches (so it can reference prior concepts correctly)
   - What the next chapter needs (so it can end with an appropriate bridge)
   - The "Omitted for now" lists (topics it MUST NOT cover)

2. **Their specific assignment** — which chapter(s) to write, with the exact outline
   text for those chapters.

3. **The output directory path** — where to save the generated files.

4. **The content generation rules** — copy the entire "Content Generation Rules"
   section below into each subagent's prompt.

5. **Research directives** — use web search when needed: if the topic involves
   time-sensitive information, inherently requires research (e.g., recent developments,
   current APIs, latest benchmarks), or your internal knowledge is insufficient to
   meet the outline's requirements faithfully. Do not search for basic, stable
   concepts you already know well — use your best judgment.

Spawn ALL subagents in one batch. Wait for ALL to complete before proceeding to Phase 3.

### Phase 3 — Consistency Review

Launch a dedicated review subagent that:

1. **Reads ALL generated chapter content** (every `index.md` and supplementary files).

2. **Checks for these issues:**
   - **Terminology consistency** — the same concept uses the same name/notation across
     chapters (e.g., not "权重矩阵" in one chapter and "参数矩阵" in another).
   - **Style/tone consistency** — same reading level, same language, same warmth level
     throughout.
   - **Transition consistency** — Chapter N's opening "What You Already Know" section
     accurately reflects what Chapter N-1 actually taught (not just what the outline
     said it would teach).
   - **Prerequisite satisfaction** — no chapter assumes knowledge from a chapter that
     hasn't been taught yet.
   - **Omitted-items compliance** — no chapter covers topics from its "Omitted for now"
     list.
   - **Cross-chapter duplication** — no two chapters spend significant time explaining
     the same concept redundantly.
   - **Factual errors** — verify key technical claims, using web search if necessary. Flag anything
     incorrect, misleading, or outdated.
   - **Missing elements** — does each chapter include concrete examples, pattern
     recognition, abstract formulation, core takeaway, and a transition bridge?

3. **Produces a structured report** with:
   - Per-chapter assessment (PASS / NEEDS FIX)
   - For each NEEDS FIX: specific issues with exact file paths and what's wrong
   - Overall recommendation: ALL_CLEAN or which chapters need regeneration

### Phase 4 — Regeneration (if needed)

If the review found issues:

1. For each flagged chapter, read the review subagent's specific feedback.

2. Adjust the subagent prompt used in Phase 2 by:
   - Adding the specific issues that need fixing
   - Adding any cross-chapter context the subagent was missing (e.g., "Chapter 2
     used X as its main example, make sure your opening references X, not Y")
   - Emphasizing the violated rule (e.g., "DO NOT cover eigenvalues — this is on
     the Omitted for now list")

3. Re-launch the subagent for that chapter with the adjusted prompt. The new
   content OVERWRITES the previous files in the chapter's directory.

4. After all flagged chapters are regenerated, optionally re-run Phase 3 (a
   lighter, targeted review of only the changed chapters plus their neighbors).

**Guardrail:** Do not loop more than twice. If issues persist after two regeneration
rounds, report the remaining issues to the user and let them decide.

### Phase 5 — Final Assembly

1. Create `<outline-dir>/README.md` — a table of contents that links to each chapter
   directory, with one-line descriptions from the outline's Core takeaway field.

2. Report completion to the user:
   - List of generated chapters with their file counts
   - Summary of any unresolved issues (if any)
   - The full path to the outline directory so the user knows where everything is

## Content Generation Rules

These rules MUST be included verbatim in every subagent prompt. They define how to
generate content from an outline chapter. Each rule exists because the common failure
mode is content that rushes to abstraction, ignores the learner's level, or leaks
topics that the outline deliberately defers.

### 1. Follow the Outline Faithfully

The outline describes HOW to teach. You generate WHAT to teach using exactly that
method. The prescribed teaching approach for a chapter is law:
- If the outline says "Concrete example → Pattern → Abstraction", you must structure
  the chapter that way.
- If the outline provides a specific concrete example, use it as your starting point —
  enhance it, but don't replace it with a different one unless the original is genuinely
  unsuitable.
- Do not reorder, skip, or add teaching steps.
- Do not cover topics from the "Omitted for now" / "暂不涉及" list.
- Do not teach above or below the Learner Profile's stated level.

### 2. Concrete-First Teaching

For EVERY new concept:
1. **Concrete example** — from a domain the learner is familiar with (read the
   Learner Profile to pick the right domain). If the user is a chef, use kitchen
   analogies. If the user is a developer, use code examples. This must come FIRST.
2. **Pattern recognition** — show 2-3 more examples of the same pattern in different
   contexts. The learner should start thinking "oh, I see the common thread".
3. **Abstract formulation** — now, and ONLY now, present the definition, formula, or
   general principle. The learner has concrete anchors to attach it to.

It is forbidden to present a definition first and then say "for example...".

### 3. Respect the Learner's Level

Use vocabulary, analogies, and pacing suited to the Learner Profile. If the profile
says "knows Python basics but no linear algebra", do NOT assume matrix notation.
If the profile says "strong math, weak physics", use equations freely but explain
physical intuition carefully.

When unsure if a concept needs explanation, err on the side of explaining it.
The "Omitted for now" list is the ONLY authorized list of things to skip.

### 4. Never Cross the "Omitted" Boundary

Every chapter has an "Omitted for now" list. These topics MUST NOT appear in your
content — not even in passing. If a natural teaching path leads near an omitted
topic, acknowledge it briefly: "There's more depth here — we'll explore this in a
later chapter." Do NOT explain the omitted topic itself.

### 5. Chain to Prerequisites

For every chapter after the first, open with a brief "What You Already Know" section
that references the key takeaways from prior chapters — specifically those that this
chapter builds on. This reinforces the learning journey and gives the learner
confidence that they're ready.

### 6. End with a Bridge

Close each chapter with a "What's Next" section that explains WHY the next chapter's
topic matters and HOW it connects to what the learner just mastered. This creates
momentum and a sense of progression.

### 7. Highlight the Core Takeaway

The outline's "Core takeaway" is the ONE thing the learner must retain. Make it
visually prominent — use a blockquote or callout. It should be the first thing the
learner sees when skimming back.

### 8. Research When Needed

Use web search when:
- The topic involves time-sensitive information or recent developments
- The outline's teaching approach requires details you're unsure about
- Your internal knowledge is insufficient to generate accurate, faithful content
- You need to verify a factual claim (dates, versions, benchmarks, terminology)

For stable, well-established concepts you are confident about, use your internal
knowledge directly. Don't search for the sake of searching — but don't guess when
you're uncertain either.

### 9. Use the Learner's Language

Write in the same language as the outline. If the outline is in Chinese, generate
Chinese content. If English, English. Match the outline's tone — conversational,
warm, and accessible. This is teaching, not a technical paper.

### 10. Self-Contained Chapters

Each chapter should be readable independently — the learner may want to review a
specific topic later without re-reading all prior chapters. Include enough context
in the opening "What You Already Know" section for stand-alone readability, without
re-teaching prior chapters.

## Output Structure

For each chapter, create a directory under the outline directory:

```
<outline-dir>/
├── learning-outline-<topic>.md     (the original outline — do NOT modify)
├── chapter-01-<slug>/
│   └── index.md                    (full chapter content)
├── chapter-02-<slug>/
│   ├── index.md
│   └── examples.md                 (optional — extended worked examples)
├── chapter-03-<slug>/
│   ├── index.md
│   └── exercises.md                (optional — practice problems)
├── ...
└── README.md                       (table of contents, generated in Phase 5)
```

Each `index.md` must follow this structure:

```markdown
# Chapter N: [Title from outline]

## What You Already Know
[Brief recap of prior chapters' key takeaways this chapter builds on]

## [Concrete Example Section — use an engaging, descriptive title]
[A relatable, domain-appropriate example. Walk through it thoroughly.
This is the learner's first contact with the new concept — make it stick.]

## Seeing the Pattern
[2-3 more examples in different contexts. Highlight what they share.]

## The General Idea
[Abstract formulation — definition, formula, architecture. The learner now
has concrete anchors, so the abstraction will feel earned, not imposed.]

## Core Takeaway
> [The ONE thing to remember — prominently displayed as a blockquote]

## Practice
[Optional: self-check questions, mini-exercises, or thought experiments]

## What's Next
[Bridge to the next chapter — why it matters and how it connects]
```

## Subagent Prompt Template

When spawning a content generation subagent, use this structure:

```
You are generating learning content for a specific chapter of a structured course.

## FULL OUTLINE CONTEXT
[Paste the entire outline file here — all chapters, Learner Profile, everything]

## YOUR ASSIGNMENT
You are writing Chapter N: [Title].

Your output directory: <outline-dir>/chapter-0N-<slug>/
Save the main content as index.md. [Optional: also create exercises.md with
practice problems.]

## GENERATION RULES
[Paste the entire "Content Generation Rules" section from Phase 2]

## CRITICAL CONSTRAINTS
- The learner's level: [key points from Learner Profile]
- Prerequisites by this point: [list of chapters 1 through N-1 and their core takeaways]
- Topics you MUST NOT cover: [the "Omitted for now" list for THIS chapter]
- Your chapter's teaching approach: [the 3-step approach from the outline]
```

## Important Reminders

- The outline is authoritative. Never second-guess the teaching approach or add/remove
  chapters. If something seems wrong about the outline, flag it to the user rather than
  silently overriding it.
- Use web search when the topic requires it — for time-sensitive content, unfamiliar
  domains, or when your internal knowledge is insufficient. For well-established
  concepts, rely on your existing knowledge.
- The Learner Profile is a hard constraint on content difficulty. Every paragraph should
  pass the test: "Would the learner, as described, understand this?"
- Subagents do NOT see each other's output during Phase 2 — they only share the outline.
  Cross-chapter alignment is the consistency reviewer's job (Phase 3).
- Task decomposition is the key value of this skill. A 12-chapter outline done
  sequentially would exhaust context. Done in parallel, it's tractable.
- When a regeneration round is needed, adjust the subagent prompt with SPECIFIC
  feedback — generic "do better" instructions won't help.
- Save ALL generated content under the outline's directory. Never scatter files across
  unrelated locations.
