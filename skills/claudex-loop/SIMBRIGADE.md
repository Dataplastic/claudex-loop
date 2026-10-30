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

## 3. Issues have a size limit, and every requirement can be checked

Each issue covers one module or layer, with at most 8 requirements and 12 edge cases. No
requirement has numbered sub-steps. Split a larger issue. If it cannot be split, tell the
owner why.

Every requirement has at least one example or edge case with a concrete, observable result
that would be wrong if the requirement were not met. "Opens Chromium with the page's
viewport" is not enough; "an element with `width: 100vw` measures 794 px on A4" is. "A file
without an extension is opened as HTML" is not enough; "its body holds a `div`, not a
`pre`" is. The factory's builder turns these examples into tests, and a requirement without
one gets a test that passes whether the code is right or not (html2idml-v2, Sprint 2:
H2I2-6, H2I2-7). When a behaviour cannot be shown reliably by a test (for example that
measuring waits for `document.fonts.ready` when fonts load from `file://` before the `load`
event), say so in the requirement and name how it is checked instead, such as the owner's
check after the sprint. Otherwise the factory's reviewer asks for a test that cannot be
written.

Work out every number in the examples (sizes, offsets, ids, rounding) and write the result,
not the formula alone. Put "Recompute every number in the examples" in the review brief, so
the reviewer checks them too. html2idml-v2's plan said A4 gives a viewport of
`(794, 1122)`, but `round(841.89 / 0.75)` is 1123, and the review did not notice.
Literal formats need the same care: `<Leading type="unit">pt</Leading>` was meant as "the
value in points", and the builder wrote `18 pt`; write the literal output, `18`.

A measured value (a width, an offset, a position from the browser) has a tolerance of at
least 1 px, or is a whole number. The factory's CI runs Chromium on Linux, which measures text
in whole pixels; the Mac the plan was checked on gives fractions. html2idml-v2's N12 asked for
marker widths within 0.5 px of `30.53`, CI measured `30`, and the issue went to the owner
(H2I2-37, Sprint C).

Before an issue adds a field to a dataclass, or changes a type's fields in any way, search the
product's tests for assertions on that type's fields: the exact set of field names, the order,
or which field is last. Each such test is named in the issue's Needs as
`existing test: <path>` with the exact change it needs, and the requirement that keeps the
existing tests as they are names it as the exception. Otherwise the issue cannot be met:
html2idml-v2's N12 and N13 added fields to `RawElement` and `ParagraphStyle` while
`test_measure_requirements.py` and `test_all_caps_writer.py` froze their fields, and both went
to the owner as plan defects (H2I2-37, H2I2-38).

A sprint holds at most the product's issues per sprint (`SB_SPRINT_MAX_ISSUES` in Cockpit's
Settings, 5 unless the owner changed it). Group the issues into sprints within that limit, or
ask the owner to raise it; Cockpit's Publish warns when a sprint holds more.

The format of an issue (its sections, their limits, the Needs words and how a check is
named) is one contract in simBrigade, `contracts/issue-text.json` (simBrigade's decision
0028), which Cockpit's Publish and the factory's preflight read; this addendum and the
factory's skill say why it is as it is.

Before writing the issues, read the factory's own rules for them in the product repository:
`skills/writing-auto-pilot-issues.md` (list `skills/` and read every skill whose `when` fits
writing issues). They hold what the factory learned from earlier sprints, such as giving
exact examples for every relevant `white-space` mode, whitespace-only content included.
Follow them in every issue, and ask the reviewer to check the issues against them too. If a
skill and this addendum disagree, put the choice to the owner. html2idml-v2's H2I2-8 gave one
example per `white-space` mode and none with leading or trailing spaces in `pre`, or with a
nested `white-space`; the skill asks for them, and the factory built them wrong.

Read `knowledge/plan-feedback.md` in the product repository too, when it exists. Foreman, the
factory's floor manager, adds a lesson there each time a sprint finds that a plan or an issue
asked for the wrong thing (simBrigade decision 0022). Apply every lesson in it to the new
plan, say in the review brief which ones you applied, and ask the reviewer to check the plan
against them.

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

**When Cockpit runs the plan** (simBrigade's decision 0026; Cockpit says so when it starts
the session), Cockpit publishes: it checks the approval, creates the issues with the body
below and links them. The host does not publish then; it tells the owner the plan is ready
and that Publish is on the Planning page. Cockpit may put the issues in the active Cycle when
the owner chooses that at Publish, and it never adds a label. Run without Cockpit, the host
publishes as the rest of this section says.

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
6. `## After the merge`, when the plan has an owner check that does not block the sprint.
   Each item names a check in the product's `adws/adw_sssf_config/checks.toml` as
   `run the check "NAME"`, so Cockpit can prepare it. When no check there fits, the plan
   proposes one (its name, its command writing into `{out}`, the files to open) for the owner
   to add before the sprint; Cockpit's Publish and the sprint's preflight say so until it is
   there (html2idml-v2, Sprint 7: H2I2-22 asked for a bare landscape command);
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
