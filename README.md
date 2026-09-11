# Restructure a Markdown documentation page with Claude Code

`doc-restructure` is a Claude Code skill that rebuilds a Markdown page that's accurate but hard to read. It moves content rather than deleting it, and its closing report says where each fact, example, and link went. The one exception is an example with invented output: the skill replaces it with output from a real run or removes it, and says which.

## Know which file settles a question

The skill is the `doc-restructure` folder. The README and license beside it are packaging:

```text
claude-code-completely-restructure-your-doc/
├── README.md
├── LICENSE
└── doc-restructure/
    ├── SKILL.md
    └── references/
        ├── house-style.md
        ├── exemplar-python.md
        └── exemplar-shell.md
```

`SKILL.md` is the procedure. `references/house-style.md` holds every writing rule, and it wins where the two disagree. The two exemplars are finished pages written to those rules, one Python and one shell, and the skill reads whichever matches your material to settle anything the rules leave open. They come from a larger Obsidian set, so their wiki links point at pages this package doesn't ship.

## Copy one directory to install it

Clone the repo and copy the `doc-restructure` folder into `~/.claude/skills`, which makes the skill available in every project:

```bash
git clone https://github.com/JoshGo6/claude-code-completely-restructure-your-doc.git
cd claude-code-completely-restructure-your-doc
mkdir -p ~/.claude/skills
cp -r doc-restructure ~/.claude/skills/
```

Don't skip the `mkdir`. If `~/.claude/skills` doesn't exist yet, `cp` creates it as a copy of `doc-restructure`, and the skill's files land directly in `~/.claude/skills` instead of in a folder of their own.

To install it in one project only, copy the folder into `.claude/skills` at that project's root instead:

```bash
mkdir -p /path/to/project/.claude/skills
cp -r doc-restructure /path/to/project/.claude/skills/
```

There's nothing to register. Claude Code finds the skill in its folder and fires it from the description in `SKILL.md`.

> [!WARNING]
> Move any existing `doc-restructure` skill aside before you install. Copying over it overwrites the files that share a name. In a project, a user-level skill of the same name wins without any error, and the project copy never appears in the session's skills listing.

## Expect an analysis before any edit

Name the skill, or describe the problem in words like *this page is confusing* or *merge these two sections*:

```text
Restructure ~/notes/Globbing.md. It's hard to follow and I keep losing the bigger picture.
```

The first turn is an analysis and a set of questions, not an edit. Once you've answered, the skill edits the page in place, one region at a time, and closes with a list of what moved where. Expect several rounds.

## Write for the flavor the page already uses

The skill writes Obsidian, GitHub Flavored Markdown (GFM), or CommonMark, and it checks the page before asking. A page with no wiki links or callouts gets CommonMark and no question. A page that already uses Obsidian or GFM syntax, or a mix of both, gets one question about which flavor to write. The choice affects only links and callouts, and the house style gives the form each flavor takes.

## Watch the recorded walkthrough

A recording of the skill running on a real page is at [joshgoldstein.org/claude-skills.html](https://www.joshgoldstein.org/claude-skills.html). Josh Goldstein wrote the skill and its house style, with Claude Code as the coding agent working under his direction.

## Use it under the MIT license

The full license text is in `LICENSE`.
