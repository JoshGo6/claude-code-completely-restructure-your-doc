# Restructure a Markdown documentation page with Claude Code

`doc-restructure` is a Claude Code skill that takes one Markdown page that's factually fine but hard to read, and rebuilds it to a known form: one shared sample fixture, which is the sample data every example on the page runs against; a lookup table covering everything the page teaches; prose saying which call to reach for; and body sections whose headings carry a claim rather than a label. Content moves rather than disappearing: every fact, example, and link is rehomed, and the closing report names where each one went. The skill doesn't fill in material the page is missing, because the complaint it answers is how the page reads.

One kind of material does come out rather than move. An example that states an outcome it never runs is an assertion, and the rules the skill obeys require a demonstration instead. Invented output is either replaced by output from a real run or taken out, and the report says which.

## Know which file settles a question

Everything the skill needs sits inside `doc-restructure/`, and the two files above it are packaging:

```text
claude-code-doc-restructure/
├── README.md
├── LICENSE
└── doc-restructure/
    ├── SKILL.md
    └── references/
        ├── house-style.md
        ├── exemplar-python.md
        └── exemplar-shell.md
```

`SKILL.md` carries the procedure and holds no writing rules of its own: analyze, interview, diagnose against a list of standard defects, edit in place, verify, and report. Every rule about how the prose should read is in `references/house-style.md`, which is the only place any of them lives. Where the house style leaves something ambiguous, an exemplar settles it, since each one is a finished page written to those rules rather than a description of them. Where `SKILL.md` and the house style disagree, the house style wins.

## Copy one directory to install it

Install the skill for every project with one copy, leaving the README and the license behind:

```bash
cp -r doc-restructure ~/.claude/skills/
```

For one project alone, copy the same directory into `.claude/skills/` at the root of that project instead. Nothing else needs registering, because Claude Code picks a skill up from the directory and fires it from the description in its frontmatter.

> [!WARNING]
> A `doc-restructure` skill you already have collides with this one, and the two installs collide differently. Copying over an existing skill directory overwrites the files inside it that share a name. Installing into a project while a user-level copy of that name exists fails more quietly: the user-level copy wins, and the project copy never appears in the session's skills listing, so nothing tells you the wrong one fired. Move the existing skill aside first, or install this one under another directory name.

## Expect an analysis before any edit

Ask for the skill by name, or describe the problem in the words its description already matches, such as *this page is confusing*, *too much is in callouts*, or *merge these two sections*. A request that fires it looks like this:

```text
Restructure ~/notes/Globbing.md. It's hard to follow and I keep losing the bigger picture.
```

The first turn is an analysis rather than an edited file. It names what is structurally wrong, what the skill would change in order of leverage, where it disagrees with your diagnosis, and the defect you didn't raise. Questions follow, since restructuring is the job the skill's rules set to the most questions, and the turn ends there. Only once you've answered does the skill edit, in place and one region at a time, closing with a change list that says what moved where and what stayed byte-identical. Expect several rounds, each one after the first a small, targeted diff.

## Write for the flavor the page already uses

The skill handles Obsidian, GitHub Flavored Markdown, and CommonMark, and it reads the page before it asks you anything. A page carrying neither a wiki link nor a callout gets CommonMark and no question at all, since it has expressed no preference worth preserving. A page already using one of the two flavors gets a single question, which is whether to keep that flavor or convert to CommonMark. A page mixing both gets the four-way form of the same question: Obsidian, GFM, both, or CommonMark.

Two things differ by flavor, and the house style carries the form each one takes. Links are `[[Page]]` and `[[#Heading]]` in Obsidian, and `[Page](page.md)` and `[Heading](#heading)` in the other two. Callouts differ three ways: Obsidian's typed blocks take a title on the header line; GitHub's five uppercase alert types — `> [!NOTE]`, `> [!TIP]`, `> [!IMPORTANT]`, `> [!WARNING]`, `> [!CAUTION]` — have no title slot at all, so what the title would have said moves into the body's first sentence; and CommonMark, which has no callout syntax whatever, takes a blockquote whose first line is the title in bold. Where Obsidian folds a callout to recap a fixture without repasting it, the other two get a `<details>` block doing the same job.

## Read the exemplars as real pages

The two exemplars are real reference pages lifted from a larger Obsidian set, one Python and one shell, so their wiki links point at pages this package doesn't ship. Leave them. Stripping them would show a page shape the skill doesn't actually produce, and the links are part of what each exemplar demonstrates. The skill reads whichever one matches your material, so a page about `curl` is checked against the shell exemplar.

## Watch the recorded walkthrough

A recording of the skill running against a real page is at [joshgoldstein.org/claude-skills.html](https://www.joshgoldstein.org/claude-skills.html), which carries a short highlight first and the full run after it. Josh Goldstein wrote the skill and the house style it obeys, with Claude Code as the coding agent working under his direction.

## Use it under the MIT license

The license is MIT and its text is in `LICENSE`. Copy the skill, change it, or ship it inside something else, as long as the copyright notice travels with it.
