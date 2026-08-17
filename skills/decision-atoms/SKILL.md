---
name: decision-atoms
description: Use when a prose document, thread, or memo reaches a decision and you need to see what the decision actually rests on — reducing it to numbered atomic facts, assumptions, constraints and unknowns plus the minimum chain linking them to the conclusion. MANDATORY TRIGGERS: /decision-atoms <file or text>, "what does this decision rest on", "break down this memo", "strip the prose out of this argument", "which assumptions are load-bearing". DO NOT trigger for: writing a new decision doc, summarising a document, general code review.
user_invocable: true
---

# decision-atoms (`/decision-atoms`)

Replaces a decision document with two artifacts: a **ledger** of numbered atomic propositions, and a **composition** that reaches the decision using only those numbers. The reader should never need the prose again.

The output is not a summary. A summary is shorter prose that says the same things. This is a change of representation: propositions become addressable, and the argument becomes a chain over addresses.

## Output contract

Your entire output is exactly two sections, `## 1. Ledger` and `## 2. Composition`. Section 2 ends with the `RESTS ON:` line, and that line is the last thing you write.

### Section 1 — Ledger

One markdown table. Columns, in order: `ID | Type | Proposition | Basis | Load-bearing`.

**ID** — type prefix plus number: `F1, F2…` `A1, A2…` `C1, C2…` `U1, U2…`

**Type** — assign by test, in this order. First match wins.

| Type | Test |
|---|---|
| Constraint (`C`) | A limit **imposed by someone or something outside the decision** — a cap, a deadline, a contract clause, a regulation, a committed target. Name the imposer in Basis: if you cannot, it is not a `C`. A measurement of how big or hard something is remains a fact no matter how much it constrains — "34 call sites" and "3 sprints of work" are `F` and `A`; "spend is capped at $800/mo by the CFO" is `C`. |
| Unknown (`U`) | The source raises the question and does not answer it. Phrase it as a question. |
| Assumption (`A`) | Asserted without verification, or flagged in the source as an estimate, projection, sample, or someone's read. |
| Fact (`F`) | Asserted with a stated basis in the source — measured, counted, confirmed, on the record. |

**Proposition** — exactly one claim per row. If a source sentence carries two claims, it becomes two rows. A row containing "and", "but", or a subordinate clause is usually two rows.

**Basis** — where in the source this comes from and how it was established ("4 weeks of metrics", "planning meeting, no customer data", "legal, in writing", "unaudited"). Every row has one.

**Load-bearing** — `yes` only if the ID appears in a chain line of Section 2 (a line containing `->`). An ID named solely in a `gates` or `FLIP:` line is `no`: it is what would change the decision, not what currently holds it up. Fill this in after writing Section 2.

Two structural rules:

- **A row with no Basis in the source does not exist.** If you cannot point to where the source establishes or raises it, it is not a row. Observations of your own, alternatives the source did not consider, and critiques you find persuasive all fail this test.
- **Uncertainty is a property of the row, not something to resolve.** An `A` stays an `A` even when it is probably true. A `U` stays a question even when you can guess the answer. Never promote a row to `F` by supplying reasoning the source did not contain.

### Section 2 — Composition

Lines only. Every line is built from IDs and operators. Permitted forms:

```
F3 + A1        -> S1: <≤8 words naming what this step establishes>
S1 + C2        -> S2: <≤8 words>
S2 + F7        -> D:  <the decision, quoted from the source>
U4 gates A1
FLIP: if U4 resolves <specific outcome> then D becomes <specific alternative>
NOT LOAD-BEARING: <IDs from Section 1 that appear in no chain line above>
RESTS ON: <the IDs feeding D, weakest type first>
```

- `Sn` are intermediate steps you introduce; each must be reached by a line above it.
- The `≤8 words` label names what the step establishes. It does not restate the content of the IDs feeding it — the reader has the ledger for that.
- **Minimum means minimum.** Include an ID only if removing it would break the chain to `D`. IDs that support nothing land in `NOT LOAD-BEARING`, which is often the most informative line.
- **A chain inherits its weakest input.** If any `A` or `U` feeds a step, mark that step `[A]` or `[U]` and carry the mark forward to `D`. A `D` marked `[U]` is a decision resting on an open question — the mark is how you say that.
- `gates` takes exactly one `U` on the left and any single ID or step on the right: `U4 gates A1`. It marks an unknown whose resolution determines whether the right side holds at all. **Gating propagates the `[U]` mark**: every step consuming a gated ID carries `[U]`, and so does `D`. A `D` reached only through ungated rows carries no `[U]` — do not add one for emphasis.
- At least one `FLIP:` line. It names an ID, a specific resolution of it, and the specific different decision that follows. "More information would help" is not a flip condition.
- `RESTS ON:` is one line of IDs, no prose. It is the last line of your output. Whatever you were about to observe about the shape of the argument, the reader gets from this line, the marks, and `NOT LOAD-BEARING:` — those four are the finding.

## Quick reference

| Question | Answer |
|---|---|
| Source contradicts itself? | Both sides get rows. Add a `U` for which holds. |
| Source gives a number with no provenance? | `A`, Basis `stated without source`. |
| Decision is implicit? | Quote the sentence closest to it as `D`. |
| Nothing is load-bearing but assumptions? | That is the finding. Let the marks show it. |
| Source is a thread with disagreement? | Speaker attribution goes in Basis, not the Proposition. |

## Common mistakes

- **Two buckets instead of four.** Sorting rows into "solid" and "shaky" loses constraints entirely — constraints are the rows that bound the answer no matter how well evidenced everything else is. Run the four tests in order.
- **Effort and scale typed as constraints.** Call-site counts, sprint estimates, and headcounts describe the work; they do not bound the option space from outside it. Ask who imposed the limit. If the answer is "nobody, that's just how big it is", the row is `F` or `A`.
- **Compound propositions.** "SQS costs ~$450/mo assuming no FIFO" is two rows: the estimate, and the assumption it rests on. Splitting them is what makes the `gates` relation visible.
- **Restating the ledger in Section 2.** If a composition line is readable without the ledger, it is carrying content that belongs in a row.
- **Grading the decision.** The composition shows what the decision rests on. Whether that is enough is the reader's call; the marks and the `FLIP` line give them what they need to make it.
- **Filling `NOT LOAD-BEARING` with nothing.** Real documents always carry material that reaches no conclusion. An empty line here usually means the chain was drawn to include everything rather than to the minimum.
