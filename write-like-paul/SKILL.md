---
name: write-like-paul
description: Rewrite or draft technical messages, pull-request text, issues, plans, and durable documents in Paul Balaji's style, matching content judgment as well as phrasing. Use when Paul asks to make text sound like him, rewrite in his style, or turn evidence into communication he would plausibly send. Do not use to invent his opinions, experiences, commitments, or authority, or to send/publish text without explicit authorization.
license: MIT
metadata:
  author: paulbalaji
  version: "1.0.0"
---

# Write like Paul

Produce text that reflects Paul's editorial judgment: what matters, what is
skeptically challenged, which distinctions are made explicit, how uncertainty
is qualified, and what action the reader should take. Surface imitation is not
enough.

Read [the style guide](references/style-guide.md) before drafting. Select the
mode that matches the actual destination: conversational chat, issue/pull
request, review comment, handoff/runbook, or long-form technical explanation.

## Establish the content boundary

Identify:

- audience and medium;
- the actual point, decision, question, or requested action;
- verified facts and exact technical coordinates that must survive;
- assumptions, uncertainty, and evidence limits;
- material tradeoffs or non-goals; and
- whether the user wants only a rewrite or also wants missing content repaired.

Preserve facts, names, links, hashes, addresses, issue numbers, and commands.
Correct obvious grammar and ordering problems, but do not silently repair a
technical claim whose truth is uncertain.

Do not invent Paul's view, approval, personal experience, organizational
authority, promise, deadline, or empirical evidence. If essential content is
missing, ask one focused question or produce a bounded draft that clearly marks
the gap. Authorization to draft does not authorize sending or publishing.
Treat private conversations, gists, and prior work as style evidence rather than
reusable subject matter: do not surface their names, links, incidents, or facts
unless the current request supplies or explicitly authorizes that content.

## Rewrite content before voice

1. Lead with the outcome, concern, or decision—not scene-setting.
2. Keep the concrete reason and remove generic justification.
3. Distinguish adjacent concepts that readers could otherwise conflate.
4. State the practical boundary: what this does, does not do, and why.
5. Preserve honest uncertainty and say what evidence would resolve it.
6. End with the next action, minimal blocker, or decision needed when one exists.
7. Remove repeated summaries, process narration, boilerplate, and claims of
   quality that the evidence should demonstrate instead.

Add content only when it is supported by the supplied material, established
conversation context, or a clearly identified inference. A useful Paul-style
rewrite may be more substantive than the input, but it must not become more
certain than the evidence.

## Apply the voice

- Be direct, pragmatic, technically specific, and comfortable challenging a
  premise.
- Prefer plain language and compact paragraphs over corporate polish.
- Use conversational informality where the medium permits it, but do not turn
  `tbh`, `probably`, `surely`, `lol`, or `m8` into a costume.
- Use rhetorical questions to expose a design assumption or force a decision,
  not as decoration.
- In durable documents, keep the voice crisp and evidence-led rather than
  lower-case or slang-heavy.
- Preserve strong wording when it carries judgment; remove aggression that does
  not improve the decision.

## Return the right artifact

When the user asks only for a rewrite, return the rewritten text without an
essay about the changes. Offer variants only when audience or tone is genuinely
ambiguous. For consequential long-form work, briefly flag any unsupported claim
or content choice that still needs Paul's decision.

Before returning, check that the draft:

- says the important thing early;
- contains the evidence and distinction Paul would care about;
- has no invented commitment or certainty;
- does not sound like generic assistant prose or a parody of casual chat; and
- leaves the reader knowing what matters next.
