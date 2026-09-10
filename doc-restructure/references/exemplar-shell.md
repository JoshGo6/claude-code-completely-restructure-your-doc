# Globbing

Globbing is how the shell turns a pattern like `*.md` into a list of filenames before the command ever runs. This page covers the pattern characters, the `shopt` options that change what they match, and the quoting that stops expansion when you want the command to do its own matching. Glob patterns are not regular expressions, and the last section says where the two diverge.

## The sample tree

Every example on this page runs against the same tree, built in a scratch directory so nothing here can touch real files:

```bash
cd "$(mktemp -d)"
mkdir -p docs/api
touch docs/index.md docs/install.md docs/notes.txt docs/.draft.md docs/README.MD \
      'docs/style guide.md' 'docs/release [draft].md' \
      docs/api/auth.md docs/api/rate-limit.md
```

Confirm you got the same tree before running anything else:

```shellsession
$ tree -a docs
docs
├── api
│   ├── auth.md
│   └── rate-limit.md
├── .draft.md
├── index.md
├── install.md
├── notes.txt
├── README.MD
├── release [draft].md
└── style guide.md

2 directories, 9 files
```

Run `cd docs` before the examples; every one of them runs from there. Four of the nine names are doing specific work:

- **`style guide.md`** has a space, so an unquoted expansion splits it into two arguments.
- **`release [draft].md`** contains glob metacharacters in the *filename*, which is the case that breaks naive quoting advice.
- **`.draft.md`** is hidden, so `*` skips it until `dotglob` is set.
- **`README.MD`** is uppercase, so `*.md` skips it until `nocaseglob` is set.

## Lookup

| Use when | Call | Result |
| --- | --- | --- |
| you want every Markdown file here | `echo *.md` | `index.md install.md release [draft].md style guide.md` |
| exactly one character varies | `echo ?ndex.md` | `index.md` |
| the first character is one of a set | `echo [ai]*.md` | `index.md install.md` |
| the first character is none of a set | `echo [!ai]*.md` | `release [draft].md style guide.md` |
| the command should receive the pattern | `echo "*.md"` | `*.md` |
| the names need no files behind them | `echo report-{1,2,3}.md` | `report-1.md report-2.md report-3.md` |
| nothing matches, and you took the default | `ls *.rst` | `ls: cannot access '*.rst': No such file or directory` |
| nothing matches, and it should vanish | `shopt -s nullglob` | a `for` loop over it runs zero times |
| nothing matches, and it should fail | `shopt -s failglob` | `bash: no match: *.rst`, and the rest of the line is skipped |
| hidden files should match too | `shopt -s dotglob; echo *` | the listing gains `.draft.md` |
| case should not matter | `shopt -s nocaseglob; echo *.md` | the listing gains `README.MD` |
| subdirectories should match too | `shopt -s globstar; echo **/*.md` | `api/auth.md api/rate-limit.md index.md install.md release [draft].md style guide.md` |
| only directories should match | `shopt -s globstar; echo **/` | `api/` |
| one name should be excluded | `shopt -s extglob; echo !(index).md` | `install.md release [draft].md style guide.md` |
| globbing should be off entirely | `set -f` | `echo *.md` then prints `*.md` |
| you need an option's current state | `shopt -p globstar` | `shopt -u globstar` |

## Which form to use

Reach for `*` and `?` for anything in one directory, and for a bracket class when the difference is a single character in a known set. Reach for `shopt` when the default matching is wrong for the job rather than the pattern: `globstar` to cross directory boundaries, `dotglob` for hidden files, `nullglob` or `failglob` when a pattern matching nothing must not be handed onward as literal text. Reach for quoting whenever the command has its own pattern matcher, which is the case for `find -name`, `grep --include`, and `rsync --exclude`.

The sections below go in that order: the pattern characters first, then what the command actually receives, then quoting, then the `shopt` options one at a time, then turning globbing off, and finally the two things globs are constantly confused with — brace expansion and regular expressions.

## Match names with `*`, `?` and a bracket class

Four characters do nearly all the work, and this block shows each against the sample tree:

```shellsession
# * matches any run of characters, including none
$ echo *.md
index.md install.md release [draft].md style guide.md

# ? matches exactly one character
$ echo ?ndex.md
index.md

# a bracket class matches one character from the set
$ echo [ai]*.md
index.md install.md

# a leading ! negates the class
$ echo [!ai]*.md
release [draft].md style guide.md
```

Notice what `*.md` did not return. `README.MD` is absent because matching is case-sensitive, and `.draft.md` is absent because a leading dot is never matched by `*` unless you ask for it. Both are `shopt` settings, and both have their own section below.

Ranges work inside a class as well, so `[a-d]*` matches names starting with `a`, `b`, `c` or `d`. Use `[!...]` for negation rather than `[^...]`: both work in Bash, but only `!` is specified by POSIX.

## Expect the command to receive filenames, not the pattern

The single most useful thing to know about globbing is that the command never sees the pattern. The shell expands it first and hands over the resulting words, so `printf` reports exactly what was passed:

```shellsession
$ printf '[%s]\n' *.md
[index.md]
[install.md]
[release [draft].md]
[style guide.md]
```

Four arguments, one per matched file. That is why an unmatched pattern behaves so strangely by default, and why a filename containing a space is dangerous the moment you stop quoting.

The awkward name makes the point directly. Typing it out unquoted gives the shell two words, not one:

```shellsession
$ ls release [draft].md
ls: cannot access 'release': No such file or directory
ls: cannot access '[draft].md': No such file or directory
```

Quoting the whole name is what makes it a single argument:

```shellsession
$ ls "release [draft].md"
release [draft].md
```

## Quote the pattern when the command does its own matching

`find`, `grep --include` and `rsync --exclude` each carry their own pattern matcher. When you leave a pattern unquoted, the shell expands it first and the command's matcher never sees it. With one matching file the mistake is invisible:

```shellsession
$ find . -name *.txt
./notes.txt
```

That worked by accident. `*.txt` expanded to the single word `notes.txt`, so `find` received the pattern it would have built anyway. Where four files match, the same command fails:

```shellsession
$ find . -name *.md
find: paths must precede expression: `install.md'
find: possible unquoted pattern after predicate `-name'?
```

Quoting keeps the pattern intact, and `find` walks the tree with it:

```shellsession
$ find . -name "*.md" | sort
./api/auth.md
./api/rate-limit.md
./.draft.md
./index.md
./install.md
./release [draft].md
./style guide.md
```

> [!warning] An unquoted pattern that works today breaks when a file is added
> The `*.txt` search above succeeded only because exactly one file matched. Nothing about that command has to change for it to start failing — one more `.txt` file in the directory is enough. Quote every pattern meant for a command's own matcher, including the ones you have watched work.

## Decide what an unmatched pattern should do

By default a pattern that matches nothing is passed through literally, so the command receives the pattern as a filename and reports it missing:

```shellsession
$ ls *.rst
ls: cannot access '*.rst': No such file or directory
```

That default is at its worst in a loop, where the body runs once with the pattern itself as the value:

```shellsession
$ for f in *.rst; do echo "saw: $f"; done
saw: *.rst
```

`nullglob` removes an unmatched pattern instead, which is almost always what a loop wants:

```shellsession
$ shopt -s nullglob
$ for f in *.rst; do echo "saw: $f"; done; echo "the loop body never ran"
the loop body never ran
```

`failglob` treats it as an error and abandons the rest of the command list:

```shellsession
$ shopt -s failglob
$ echo *.rst; echo SAME_LINE
bash: no match: *.rst
```

`SAME_LINE` never printed, because everything after the failure on that line was skipped. The next line of a script still runs — `failglob` ends the command list, not the shell.

## Include hidden files and ignore case with `shopt`

`dotglob` makes `*` match names beginning with a dot, which it otherwise never does:

```shellsession
$ shopt -s dotglob
$ echo *
api .draft.md index.md install.md notes.txt README.MD release [draft].md style guide.md
```

`nocaseglob` makes matching case-insensitive, which is what finally reaches `README.MD`:

```shellsession
$ shopt -s nocaseglob
$ echo *.md
index.md install.md README.MD release [draft].md style guide.md
```

Both are off by default, both are per-shell, and neither survives into a script you run as a child process. Set them inside the script that depends on them.

## Descend into subdirectories with `globstar`

Without `globstar`, `**` means exactly what `*` means, so `**/*.md` reads as `*/*.md` and reaches one level down and no further:

```shellsession
$ echo **/*.md
api/auth.md api/rate-limit.md
```

With it set, `**` crosses directory boundaries and the same pattern covers the whole tree:

```shellsession
$ shopt -s globstar
$ echo **/*.md
api/auth.md api/rate-limit.md index.md install.md release [draft].md style guide.md
```

A trailing slash restricts the match to directories:

```shellsession
$ shopt -s globstar
$ echo **/
api/
```

> [!warning] `globstar` needs Bash 4, and macOS ships Bash 3.2
> On a stock macOS system `shopt -s globstar` fails and `**` silently keeps its one-level meaning, so a script that relies on it processes only the top directory and reports no error. Use `find` there, or install a current Bash from Homebrew and point the shebang at it.

## Exclude names with `extglob`

`extglob` adds five pattern forms, of which `!(...)` is by far the most useful — it matches everything except what it lists:

```shellsession
$ shopt -s extglob
$ echo !(index).md
install.md release [draft].md style guide.md
```

Alternatives are separated with a pipe, so several names can be excluded at once:

```shellsession
$ shopt -s extglob
$ echo !(index|install).md
release [draft].md style guide.md
```

The other four forms are `?(x)` for zero or one, `*(x)` for zero or more, `+(x)` for one or more, and `@(x)` for exactly one of the listed alternatives.

> [!warning] `extglob` must be set before the line is parsed
> Bash parses a whole command list before running any of it, so `shopt -s extglob; echo !(index).md` on one line is a syntax error — the `!(` is read before the option takes effect. In a script file it works, because each line is parsed as it is reached. Set `extglob` on its own line, or start the shell with `bash -O extglob`.

## Turn globbing off when you need the pattern intact

`set -f` disables filename expansion entirely, which is occasionally the cleanest way to pass a pattern onward untouched:

```shellsession
$ set -f
$ echo *.md
*.md
```

`set +f` turns it back on. To check any option's current state rather than guessing, ask:

```shellsession
$ shopt -p globstar
shopt -u globstar
```

The output is a command you could run, so `shopt -u` means the option is currently unset and `shopt -s` means it is set.

## Distinguish brace expansion from a glob

Brace expansion looks like globbing and is not. It happens earlier, it is pure text generation, and it does not care whether the files exist:

```shellsession
$ echo report-{1,2,3}.md
report-1.md report-2.md report-3.md
```

Not one of those files is in the tree. That is the whole difference: a glob is filtered against the filesystem and disappears or passes through when nothing matches, while a brace always produces its words. The two combine, because braces expand first and the results are then globbed, which is why `*.{md,txt}` works.

## Read a glob as a glob, never as a regex

The same characters mean different things in the two languages, and knowing regex well is what makes the mistake easy:

| Pattern | As a glob | As a regex |
| --- | --- | --- |
| `*` | any run of characters, including none | zero or more of the *previous* item |
| `?` | exactly one character | zero or one of the previous item |
| `.` | a literal dot | any single character |
| `[abc]` | one character from the set | one character from the set |

Only the bracket class means the same thing in both. A glob is also anchored at both ends — it must match the entire filename, where a regex matches anywhere in the string unless you anchor it — which is why `*.md` needs its leading `*` at all.

## See also

- [[Command Index]] — every command, builtin and expansion the curriculum covers, and the page that owns each.
- [Pattern Matching](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html) in the Bash manual — the authority on `extglob` forms and bracket-class syntax.
- [The Shopt Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html) — the full list of options, including the ones this page does not use.
