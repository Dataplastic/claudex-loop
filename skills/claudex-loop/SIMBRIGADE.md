# simBrigade planning addendum

This addendum applies when the plan is for a product built by the simBrigade factory, where
the plan ends as issues that SSSF builds under auto pilot. Where it differs from
[SKILL.md](SKILL.md), this addendum wins.

Why it exists: test 1 (12 October 2026, a blind replay of html2idml's planning from commit
`dba588a`) foresaw six of eight defects the factory later hit. But the review never
converged: all 7 rounds ended in REVISE, and the host accepted all 21 findings. From round 4
on, the findings were about safeguards the host itself had added the round before. The plan
grew a local HTTP server, a streaming fetch, DevTools font isolation, gzip decoding and size
limits for a local tool with one user. Two issues became too large for auto pilot.

## 1. A review brief opens the plan

`PLAN.md` starts with `## Review brief`, written with the owner in Phase 1:

- **Goals:** what the owner wants v1 to do, in their words.
- **Decisions:** the settled decisions, or a pointer to their table.
- **Threat model:** who runs the product, on what input and where, in one or two sentences.
  For example: "one user converts their own HTML files on their own Mac".
- **Out of scope:** what v1 deliberately does not do.

The reviewer judges the plan against this brief. Keep it short and change it only with the
owner.

## 2. The host arbitrates every finding

Classify each finding before acting on it, and record the class in the log:

- **Defect:** the plan fails a goal, a decision or an acceptance criterion, contradicts
  itself, or breaks within the threat model. Fix it.
- **Scope:** hardening, robustness, features or edge cases beyond the brief. Reject it with
  a one-line reason, or put it to the owner as an explicit choice with a recommendation and
  what it would add (requirements, issues). Never accept a scope finding silently.

Each new mechanism in the plan, such as a server, a network fetch or a protocol hook, must
trace to a goal or a decision in the brief. Otherwise it goes. When in doubt, prefer the
simpler design that meets the brief.

## 3. Issues have a size limit

Each issue covers one module or layer, with at most 8 requirements and 12 edge cases. No
requirement has numbered sub-steps. Split a larger issue. If it cannot be split, tell the
owner why.

## 4. Reference files from the program that must accept the output

When the output must be accepted by a program the factory cannot run in CI (InDesign, Word,
a print workflow), ask the owner in Phase 1 for reference files saved by that program:

- Each writer issue requires the output to follow the references' byte-level conventions,
  such as namespace prefixes, processing instructions, XML declarations and part order. A
  namespace-aware parser treats different prefixes as equal, but the program may not.
- List the references as an `owner:` need on the first writer issue.
- Put an owner check in the program after the first writer sprint, as an `owner:` need on
  the next sprint's first issue.

## 5. Stop on quality, not on budget

Stop the review loop and ask the owner when:

- every finding in a round concerns text the host added in the previous round's
  dispositions, or
- two consecutive rounds bring only scope findings or low/medium defects.

Do not extend the round budget by default. Present what is open, the host's position and a
recommendation. An approval of an earlier revision still counts only for that revision's
SHA256.
