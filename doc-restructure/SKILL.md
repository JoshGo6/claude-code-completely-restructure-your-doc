---
name: doc-restructure
description: Rework a single existing Markdown documentation page for clarity — reorganize sections, rebuild the comparison table, establish a shared sample fixture, retitle headings, relocate material, and replace examples that assert rather than demonstrate. Use this when the complaint is how the page READS rather than what it's missing — "this is confusing," "I'm losing the bigger picture," "it's hard to read," "too much is in callouts," "merge these two sections," "the table needs to change," or any request to move, merge, reorder, retitle, or delete parts of a page that is otherwise complete. Trigger it even when "restructure" or "clarity" never appear, and expect several review rounds rather than one pass.
---

# Doc Restructure

Take one existing Markdown page that is factually fine but hard to read, and return it rebuilt to the target form. Material moves, merges, and gets re-demonstrated; it does not disappear.

The question this skill answers is "why is this hard to follow?" rather than "what's missing?"

**Content is preserved. Form is not.** Every fact, example, and link survives, rehomed. The scaffolding that carries them — preamble, headings, comparison table, sample data, section order — is always in scope for wholesale rewrite, and a large diff there is the job, not churn. "Never retype the page" means don't relaunder prose the user wrote; it does not license leaving a defective heading or table alone because fixing it would be noisy.

Three failure modes to design against: **silent loss** (a fact, example, or link disappears in the reshuffle), **an unreviewable diff** (so much churn the user must re-read the page to find the changes), and **repair without reshape** (fixing the defects the user named while leaving the page in a form the exemplar wouldn't recognize).

## The rules this skill obeys

This skill holds no writing rules of its own. The authorities, in order:

1. **The house style** — `references/house-style.md`. Clarity, headings, tables, fixtures, callouts, fences, verification. Read it before analyzing anything.
2. **The exemplar** — the page those rules produce. Where the house style leaves something ambiguous, it decides. Read the one matching your material:
   - **Python** — `references/exemplar-python.md`.
   - **Shell** — `references/exemplar-shell.md`.

Where anything below disagrees with those, they win. In particular: headings are imperative and carry no em dash, the intro is complete sentences, every fence has a lead-in above it, and the lookup table is *Use when / Call / Result* placed after the fixture and before the navigation prose.

## The target form

Six elements, as realized in the exemplar:

1. **Intro** — the house style's opening rule, under no heading, and one to three sentences of it. A preamble that restates the table's column headers is a defect.
2. **Shared fixture** — one named sample set under `## The samples`, defined once, ahead of the table, with a stated reason each sample exists. Later blocks assume it and don't repeat the imports. It is built to feed the table.
3. **Lookup table** — *Use when / Call / Result*: every row a copy-pasteable expression against the fixture beside its literal output, with each success adjacent to its matching failure.
4. **Prose recommendation** — `## Which call to use`, **after** the table, naming the default and characterizing each alternative, written so it would still guide a choice if the table were deleted. It closes by naming the order of the sections that follow.
5. **Body sections** — an imperative heading carrying a claim, and every code block preceded by a lead-in and followed by a sentence saying what happened and what it costs the reader. Every item in the table is treated here; the table is an index, not a replacement.
6. **`## See also`** last, and callouts kept to the things that bite at the keyboard.

## Workflow

The ten steps below run in order, and you don't edit the page until the sixth. Steps 1 and 2 settle what the page needs, 3 to 5 establish what's wrong with it and what its examples really print, 6 makes the edits, 7 to 9 check and report them, and 10 covers the rounds that follow.

### 1. Analyze before touching anything

The first response is analysis, not an edited file. Do not open the editor in the same turn unless the user explicitly said "just do it."

The analysis covers four things:

1. **What is actually wrong**, named structurally — not "it's dense" but "the decision guidance is split above and below the table" or "the page has no fixture, so every block re-establishes its own sample data."
2. **What you would change**, in order of leverage.
3. **Where you disagree with the user's proposal.** They can see the symptom; they may have misdiagnosed the cause. Say so plainly and say why.
4. **What they haven't raised** — the defect they didn't mention, the link that will be orphaned, the promise the page makes that the change would break.

An analysis that only agrees is a wasted turn.

### 2. Interview until the guesses are gone

Use `AskUserQuestion`, under the house style's interview rule. Restructuring is the reshaping case that rule sets to the most questions, and it lists the forks worth asking.

The exemplar settles some of those forks, so don't spend a question on one it answers. The turn ends after asking.

One fork is settled by reading the page first and only then by asking. Three flavors are live — Obsidian, GitHub Flavored Markdown, and CommonMark — and they differ on links and on callouts, for which the house style carries the form each one takes. Inventory what the page already uses, then take the branch that matches:

- **Neither marker present: write CommonMark, and ask nothing.** A page using no wiki link and no callout has expressed no preference, so there is nothing to preserve and no question worth spending.
- **One flavor present: ask whether to keep it or convert to CommonMark.** Ask even though the page has plainly shown you which flavor it is written in. This is the one fork the house style's *do not ask what the material already answers* does not reach, because the page answers what it currently **is** and the question is what it should **become**.
- **Both present: ask which of the four the output should be** — Obsidian, GFM, both, or CommonMark. Offer all four. A page mixing the two has settled nothing, and keeping both is a legitimate answer rather than a fallback.

Four things mark a page as Obsidian: a wiki link (`[[Page]]`, `[[#Heading]]`, `![[Embed]]`), a callout carrying a title on its header line, a folded callout (`> [!note]-`), and a callout whose type is outside GFM's five. The GFM shape is narrower — an alert whose type is one of those five, sitting alone on its header line with no title after it. Ignore tables, task lists, and strikethrough when deciding: Obsidian renders all three, so finding one discriminates nothing.

One shape is ambiguous, and it is settled here by rule rather than left to judgment. A bare five-type alert renders as a callout in Obsidian too, so it can always be argued away as an untitled Obsidian callout. Do not argue it away. **Count it as GFM wherever it appears**, which is what lands a page carrying both a wiki link and a bare alert on the four-way question rather than the two-way one. Deciding this case differently from one run to the next is the non-determinism this skill exists to remove.

Carry the answer into every link and callout the restructure writes.

### 3. Diagnose against the standard defects

Walk this list explicitly. Most confusing pages have four or five of these at once.

| Defect | What it looks like |
|---|---|
| Dimension-list preamble | The opening restates the table's column headers instead of naming the trap. |
| Prose-free table | The grid is the only decision guidance; delete it and the reader can't choose. |
| Split guidance | The same decision advice appears above *and* below the dense element, in different words. |
| Load-bearing table | Prose says "see the table above" instead of explaining. |
| No entry point | Sibling options with no comparison table, so choosing requires reading every section. |
| Pointer or sentence cell | A cell says "see below," or holds a clause of commentary where data belongs. |
| Fixture-less examples | Every block invents its own sample data and repeats the imports. |
| Assertion-only example | The output can't distinguish the claim from its alternative — one match where the lesson is "one per match," a stated outcome that is never run. |
| Broken promise | The page claims every item is covered below, and two of them have no section. |
| Bare API-name heading | `## shutil.move()`, or any heading built as `name — claim`. Make the claim the heading. |
| Placeholder heading | `Putting it together`, `Common Pitfalls`, `Overview` — a label where a claim belongs. |
| Callout as storage | Normal behavior and recommended practice parked in callout blocks. |
| Triplicated rule | One fact stated in a subsection, a callout, and a pitfalls section. |
| Caveat before usage | The gotcha appears before the example that shows anyone using the thing. |
| API-order roster | Options follow the module's order rather than how often each is reached for. |
| Orphaned link | The only occurrence of a link sits in a section the restructure would dissolve. |

### 4. Apply the clarity standards

The ones this skill adds, about moving material rather than writing it, are under **Clarity standards specific to restructuring** below.

### 5. Verify every example by running it

The verification standard is in the house style, including the two parts that catch people here: the examples you are **keeping unchanged** get run too, and the table's **Result** column is output and gets run like any other. Restructuring changes sample data more often than expected, and a stale offset in a `span()` comment is indistinguishable from a lie.

### 6. Make targeted edits only

The house style's *Editing what already exists* governs the mechanics: a pristine original, one region at a time, never a retype. Where the page is committed and clean, `git show HEAD:<path>` is the pristine original and there is nothing to copy.

One line of that rule is this skill's to draw, because this skill is licensed to rewrite the scaffolding. A rewritten heading, table, or preamble is a deliberate change; a re-flowed paragraph beneath it is not.

### 7. Check the result against the exemplar

Before diffing, walk the six elements of **The target form** against the edited copy and name where each one lives. Any element you can't point to is unfinished work, not a judgement call. This step catches what the defect list doesn't: the page whose listed defects are all fixed and whose shape is still wrong.

### 8. Diff before delivering

Run the diff the house style requires, over the page you edited; its *Diff before delivering* carries the command. Four more things are this skill's to confirm from that diff: the link inventory is unchanged, the callout count has not crept back up, the imports are not restated once the fixture exists, and no double blank lines were introduced.

### 9. Report the change list

The page is edited in place, so there is no file to deliver.

Report briefly, in this shape: **Moved / merged** (what went where, and what it fixed), **Corrected** (errors fixed, quoting the original wording), **Added**, **Removed** (with where the content landed instead), **Incidental** (whitespace, typos, malformed markup).

Close by confirming the rest is byte-identical, and by flagging anything you chose not to change but think is still wrong. No summary of the page's contents — the user wrote it.

### 10. Expect more rounds

This job converges over several passes. Each round after the first should be a small, targeted diff. When a later round contradicts an earlier instruction because the premise changed, say so out loud rather than silently reverting — "you asked me to trim this when the table was carrying the examples; it can't, so I'd restore them" keeps the user's model of the page accurate.

Flag defects in the user's own edits with the same directness as defects in the source. They act on flags; hedging wastes a round.

## Clarity standards specific to restructuring

Clarity, headings, fixtures, tables, examples, callouts, fences and verification are all in
the house style, which this skill does not restate. Read it before analyzing. Only the rules
below are this skill's own, because they are about *moving* material rather than writing it.

### Rehome every link rather than losing it

The house style settles where content belongs, that an in-page cross-reference is evidence of
misfiling, and what renaming a heading costs. One rule is this skill's own, because only a
restructure dissolves a section: **preserve every existing link, including the ones
inside sections you dissolve.** Rehome each one rather than letting it go with its section.

### Deduplicate by relocation

The same rule stated in three places is a placement problem, not a content problem. Pick the
one place the reader needs it, move it there, and delete the other two occurrences — not the
material. Removing a section is correct when its content has a better home. Removing a
section to save space is not.

### A fixture that can't serve a section is the wrong fixture

A section needing sample data the page's fixture can't provide is a signal to fix the
fixture, not to give that section its own. That judgement belongs here rather than in the
house style, because only a restructure is in a position to change the fixture.

## Hard Rules

- **Never delete a fact, an example, or a link without rehoming it.** Length reduction is not the goal; findability is.
- **Always run the examples**, including the ones you didn't write and the ones in table cells.
- **Always check against the exemplar matching the material** — `exemplar-shell.md` for shell, `exemplar-python.md` for Python — and always diff. A run that skips either is incomplete.
- **Never add links outside the page** unless the user asks.
- **Never restructure before the analysis turn** unless the user explicitly waived it.
- **Edit the page in place.** Never return the page as chat text.

## Common Failure Modes

**Editing on the first turn.** The user asked what you think. An immediate rewrite skips the disagreement that would have saved two rounds, and commits them to a direction they never chose.

**Repairing the named defects and stopping.** The listed complaints are the symptoms the user noticed, not the diagnosis. A page can have every reported problem fixed and still open with a dimension list, run seven redundant imports, and title its sections with bare function names. Step 7 exists because this failure feels like success.

**Treating a form change as churn.** Rewriting a preamble or rebuilding a table produces a big diff and a better page. The conservatism protects the user's sentences, not the page's skeleton.

**Elegance that costs a demonstration.** A single reused sample, a uniform row shape, a symmetrical section structure — these feel like quality and are worth nothing if they prevent an example from showing the behavior. Completeness of evidence beats consistency of presentation.

**Treating the table as the source of truth.** The table is an index. The moment prose refers to it instead of explaining, the page has no explanation in it.

**Deleting to deduplicate.** The instinct to cut the redundant copy is right; cutting the content rather than moving it is how pages lose their only statement of a rule.

**Fixing a broken cross-reference instead of removing the need for it.** If you find yourself repairing an anchor link, ask first whether the two sections should simply be one.
