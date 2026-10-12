# simBrigade planning addendum

This addendum applies when the plan is for a product built by the simBrigade factory, where
the plan ends as issues that SSSF builds under auto pilot. Sections 1-5 shape the plan and its
review; section 6 publishes the approved plan's issues. Where it differs from
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

## 6. Publish the issues to It's a Plan

The plan's issues become issues in It's a Plan only through this step, and only when both
of these hold:

- the plan is APPROVED for its current SHA256 (the runner's `check` passes), and
- the owner says to publish and names the It's a Plan project (its key, for example `H2`).

Without an approval, publish only if the owner explicitly says so for this plan; record that
override in the log.

**Tools.** Use the It's a Plan MCP server (`itsaplan`) and only these tools: `list_projects`,
`get_project`, `list_issues`, `search_issues`, `get_issue`, `create_issue` and
`link_issues`. Never update, close or delete an issue, and never add labels, a Cycle,
assignees or comments.

**Before creating anything,** list the project's issues. If any of the plan's titles already
exist there (a second run), stop and ask the owner instead of creating duplicates.

**Order and keys.** Create the issues one at a time in the plan's order, and note each new
key. A later issue's dependencies refer to the keys already created.

**Title and body.** The title is the plan's issue title without its local id. The body
holds, in this order:

1. the issue's one- or two-sentence goal;
2. `## Requirements`, numbered as in the plan;
3. `## Edge cases`;
4. `## Out of scope`;
5. `## Needs`, with each `depends on:` rewritten to the real key (`depends on: H2-3`).
   Every Needs item is a single line, because the factory reads only the first line of an
   item. `owner:` items are things the owner must do before the issue can start;
6. `## After the merge`, when the plan has an owner check that does not block the sprint;
7. a last line: `Planned in PLAN.md, SHA256 <first 12 characters>.`

Do not add requirements, edge cases or explanations that are not in the approved plan; the
issue text is the contract that SSSF builds and Idea reviews against. When the project
supports issue relations, also link each dependency with `link_issues`; the `depends on:`
line is still what the factory reads.

**No mandate.** Create every issue without labels. A label set at creation never counts as
an auto pilot mandate; the owner reads the issues and adds the label and the Cycle.

**Afterwards.** Do not edit `PLAN.md`, because that would invalidate the approval. Append a
table to the log with the plan id, the new key and the title of each issue. Then tell the
owner the keys, the `owner:` needs that must be met before the first sprint, and that the
issues wait for their label and Cycle.
