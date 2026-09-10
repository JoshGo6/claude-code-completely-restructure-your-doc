# Reading and Writing Files

This page covers getting text out of a file and back into one: whole-file reads, line-by-line reads for large files, writing, appending, and the encoding and newline settings that decide whether any of it round-trips correctly. Building the paths these calls take is on [[Path Objects]], and checking whether a file exists before you open it is on [[Inspecting Files and Directories]].

## Setup

Nothing to install: `pathlib` and `open()` are both in the standard library.

## The samples

Every example on this page runs against three small files, created here in one go:

```bash
printf '# Guide\nline two\nline three\n' > page.md
printf 'a\r\nb\r\n' > crlf.md
printf 'café\n' > utf.md
```

`page.md` has three lines so a line-by-line read has a visible shape. `crlf.md` uses Windows line endings, because the newline translation is invisible on a file that doesn't have them. `utf.md` holds `café`, whose `é` is two bytes in UTF-8, which is what makes a wrong `encoding=` argument fail rather than quietly succeed.

Most examples start from a `Path` pointing at the first of those files:

```python
from pathlib import Path

p = Path("page.md")
```

## Lookup

| Use when                                        | Call                                            | Result                                                     |
| ----------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| you want the whole file as one string           | `p.read_text(encoding="utf-8")`                 | `'# Guide\nline two\nline three\n'`                        |
| you want the whole file as a list of lines      | `p.read_text(encoding="utf-8").splitlines()`    | `['# Guide', 'line two', 'line three']` — newlines removed |
| you want lines with their newlines attached     | `f.readlines()`                                 | `['# Guide\n', 'line two\n', 'line three\n']`              |
| the file is too large to hold in memory         | `for line in f:`                                | one line per pass, newline still attached                  |
| you read from the same handle twice             | `f.read()` then `f.read()`                      | full text, then `''` — the handle is at the end            |
| you want a handle something closes for you      | `with open(path, encoding="utf-8") as f:`       | handle closed at the end of the block, including when the block raises |
| you open a handle without `with`                | `f = open(path, encoding="utf-8")`              | a usable handle; nothing closes it while the name still refers to it |
| you consume a file inside one expression        | `json.load(open(path, encoding="utf-8"))`       | the parsed data; the handle has no name and is released as the statement ends |
| you want Python to name the handles nothing closed | `python3 -X dev script.py`                   | `ResourceWarning: unclosed file …`, one per handle          |
| you want to write, replacing everything         | `p.write_text("x\ny\n", encoding="utf-8")`      | `4` (characters written); file truncated then written      |
| you want to add to the end of a file            | `open(path, "a", encoding="utf-8")`             | existing content kept, new content appended                |
| you open with mode `"w"`                        | `open(path, "w", encoding="utf-8")`             | file emptied immediately, before you write anything        |
| you read a file that isn't there                | `Path("nope.md").read_text()`                   | raises `FileNotFoundError`                                 |
| you write into a directory that isn't there     | `open("newdir/x.md", "w")`                      | raises `FileNotFoundError` — parent dirs are not created   |
| the file has Windows line endings               | `Path("crlf.md").read_text(encoding="utf-8")`   | `'a\nb\n'` — `\r\n` translated to `\n`                     |
| you need the bytes exactly as stored            | `open("crlf.md", encoding="utf-8", newline="")` | `'a\r\nb\r\n'` — no translation                            |
| the file's encoding doesn't match what you said | `Path("utf.md").read_text(encoding="ascii")`    | raises `UnicodeDecodeError`                                |
| you are done with a handle you opened yourself  | `f.close()`                                     | `None`; anything still buffered is written to disk         |
| you need the bytes on disk but the handle open  | `f.flush()`                                     | `None`; the buffer is written out, the handle stays usable |
| you read the file before closing the handle     | `f.write("x")` then `p.read_text()`             | `''` — the text is in the buffer, not on disk yet          |
| you use a handle after closing it               | `f.write("more")`                               | raises `ValueError: I/O operation on closed file.`         |
| you want to know whether a handle is closed     | `f.closed`                                      | `True` after `close()`, and after a `with` block ends      |

## Which call to use

Use `read_text()` and `write_text()` for anything that fits comfortably in memory, which for documentation work is essentially everything. They are one line each, they open and close the file internally so there is no handle to mismanage, and they take the same `encoding=` argument `open()` does.

Use `open()` inside a `with` block when the file is large enough to matter, when you are appending, or when you need to read and write in the same pass. The `with` block closes the file even if the code inside it raises — without it, a crash can leave a half-written file on disk with its buffer unflushed.

Use a bare `open()` with no `with` only inside a single expression that consumes the file immediately, the shape `json.load(open(path))` has. That form fits in a lookup-table cell and a two-line `with` block does not, which is why the table on [[JSON]] is written that way; in a script, write the `with` block.

**Always pass `encoding="utf-8"`.** Without it, Python uses a platform default that differs between machines, which means a script that reads a file correctly on your laptop can raise `UnicodeDecodeError` on a colleague's. This is the single highest-value habit on this page.

The sections below cover the three ways of opening a file, whole-file reads, line-by-line reads, writing, closing, and the encoding and newline settings.

## Use `with` so the file always closes

`with open(...) as f:` is the form to write by default. It closes the handle at the end of the block, on the normal path and when the code inside raises, and it is the only form where you know exactly when the file closed:

```python
with open("page.md", encoding="utf-8") as f:
    text = f.read()

f.closed        # True — the block closed the handle on the way out
```

The name `f` outlives the block; only the open file behind it is gone. A bare `open()` returns that same kind of handle and closes nothing:

```python
f = open("page.md", encoding="utf-8")
text = f.read()
# nothing here closes f
```

That is not an immediate disaster, which is why the pattern survives in so much code. CPython closes a file object as soon as the last reference to it disappears, so this handle closes when `f` goes out of scope — at the end of the function it sits in, or at the end of the process for a script's top level. What you give up is knowing *when*, and the guarantee itself: prompt closing is a CPython reference-counting detail rather than a language rule, and an implementation that collects on its own schedule, such as PyPy, holds the file open longer.

Python will name every handle nothing closed if you ask it to. Run the script in development mode:

```bash
python3 -X dev script.py
```

For the bare `open()` above, that reports the handle as still open at interpreter shutdown — `sys:1` is the shutdown, not a line in your file:

```text
sys:1: ResourceWarning: unclosed file <_io.TextIOWrapper name='page.md' mode='r' encoding='utf-8'>
```

Nothing is reported for the `with` version, because the block already closed it. That warning is silenced by default, so a script that leaks handles looks exactly like one that doesn't until you go looking.

### Skip `with` only where the handle dies with the statement

One shape is genuinely safe without `with`: a single expression that opens the file and consumes it in the same breath, leaving no name to hold the handle afterwards.

```python
import json

data = json.load(open("repo.json", encoding="utf-8"))
```

Nothing refers to that file object once `json.load` returns, so it is released as the statement ends rather than at the end of the run. This is the form the lookup tables use, here and on [[JSON]], because a table cell holds one line and a `with` block needs two.

In a script, write the two-line form anyway:

```python
with open("repo.json", encoding="utf-8") as f:
    data = json.load(f)
```

It costs one line, and the one-line version stops being safe the moment someone edits it into `f = open(...)` to reuse the handle — which is the ordinary way that code grows.

### Call `close()` yourself when the handle must outlive a block

Sometimes the handle has to stay open across code a `with` block cannot wrap, such as a file opened in one function and written from another. Closing is then yours, and `try` / `finally` is what makes it survive an exception:

```python
f = open("page.md", encoding="utf-8")
try:
    first = f.readline()
finally:
    f.close()
```

`finally` runs whether or not the body raised, which is exactly the guarantee `with` gives you in one line instead of four. If you find yourself writing this, check first whether the block really cannot be a `with`.

## Read the whole file when it fits in memory

`read_text()` hands back the entire file as one string, and `splitlines()` turns that string into a list of lines:

```python
p.read_text(encoding="utf-8")
# '# Guide\nline two\nline three\n'

p.read_text(encoding="utf-8").splitlines()
# ['# Guide', 'line two', 'line three']
```

`read_text()` returns one string containing the entire file including its newlines. `splitlines()` breaks it into lines and drops the newlines, which is what you want when processing content and not what you want when writing the file back out. See [[Strings]] for `splitlines(keepends=True)` and the round-trip.

The equivalent with a handle needs the `with` block:

```python
with open("page.md", encoding="utf-8") as f:
    text = f.read()
```

A handle is consumed as you read it, so a second read of the same handle finds nothing left:

```python
with open("page.md", encoding="utf-8") as f:
    f.read()    # '# Guide\nline two\nline three\n'
    f.read()    # ''
```

The second call returns an empty string rather than raising, because the position is already at the end. This is the same shape as the exhausted-iterator problem in [[Inspecting Files and Directories]]: no error, just nothing.

## Loop the handle for a file too large to hold

Iterating the handle itself reads one line per pass, and `rstrip()` takes the newline off each one:

```python
with open("page.md", encoding="utf-8") as f:
    for line in f:
        print(repr(line.rstrip()))
```

The `repr()` is there so the stripping is visible — each line prints with its quotes and without a trailing `\n`:

```text
'# Guide'
'line two'
'line three'
```

Iterating the handle reads one line at a time and never holds the whole file, which is what makes it safe on a log of any size. Each line arrives with its `\n` still attached, which is why a file loop almost always begins with `rstrip()`.

`f.readlines()` returns every line as a list, newlines attached — it reads the whole file, so it has no memory advantage over `read_text().splitlines()` and is mostly worth recognizing in other people's code.

## Overwrite with `w`, add to the end with `a`

`write_text()` replaces the file's entire contents and returns how many characters it wrote:

```python
Path("out.md").write_text("hello", encoding="utf-8")   # returns 5, the character count
Path("out.md").write_text("hi", encoding="utf-8")      # returns 2; file now contains 'hi'
```

There is no "only if changed" and no confirmation. The old contents are gone as soon as the call runs, with no prompt and no backup — and gone even if the write that follows fails, which is what the callout below is about.

Mode `"a"` adds to the end instead, and mode `"w"` on a handle throws the file away the same way `write_text()` does:

```python
with open("append.md", "a", encoding="utf-8") as f:
    f.write("one\n")
with open("append.md", "a", encoding="utf-8") as f:
    f.write("two\n")
# file contains 'one\ntwo\n'

with open("append.md", "w", encoding="utf-8") as f:
    f.write("fresh\n")
# file contains 'fresh\n' — the previous two lines are gone
```

`f.write()` does not add newlines. Every line you write needs its own `\n`, or the file ends up as one long line.

> [!warning] Mode `"w"` empties the file at open time
> The truncation happens when the file is opened, not when you write — so an exception between opening and writing leaves you with an empty file and no original.
> Read the whole file, build the new text in memory, then write. Better still, copy the original first: see [[Bulk Editing a Docs Folder]].

### Create the parent directory before writing into it

Neither form makes a missing directory on the way, and the failure names the whole path rather than the part that is missing:

```python
open("newdir/x.md", "w")
# FileNotFoundError: [Errno 2] No such file or directory: 'newdir/x.md'
```

Call `Path("newdir").mkdir(parents=True, exist_ok=True)` first, which is safe whether or not the directory already exists.

## Close the handle before you read the file back

`f.write()` does not put text on disk. It puts it in a buffer that is flushed when the file closes, which means the file can be empty at a point where your script has already written to it:

```python
f = open("report.md", "w", encoding="utf-8")
f.write("first line\n")
Path("report.md").read_text(encoding="utf-8")   # ''
f.close()
Path("report.md").read_text(encoding="utf-8")   # 'first line\n'
```

`close()` returns `None` and flushes on the way out. Calling it twice is harmless. After it, the handle is unusable in both directions:

```python
f.write("more\n")
# ValueError: I/O operation on closed file.
f.closed
# True
```

The `with` block performs that same `close()` at the end of the block, including when the code inside it raises, which is why the reading and appending examples above are all written that way:

```python
with open("report.md", "a", encoding="utf-8") as f:
    f.write("second line\n")
# f.closed is True here
```

The reason this rarely bites in a small script is that Python closes open handles when the interpreter exits, so a script that writes and then ends produces the right file whether or not you closed anything. It bites when the same run does something else with the file — reads it back, hands the path to another tool, checks its size — and gets the state from before the write.

When the handle genuinely has to stay open and the bytes have to be on disk now, `f.flush()` empties the buffer without closing anything:

```python
f = open("report.md", "w", encoding="utf-8")
f.write("first line\n")
f.flush()
Path("report.md").read_text(encoding="utf-8")   # 'first line\n'
```

## Pass `encoding="utf-8"` every time

Reading with an encoding the file does not use raises `UnicodeDecodeError` and names the byte it stopped on:

```python
Path("utf.md").read_text(encoding="ascii")
# UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 3
```

The file held `café`, and `é` is two bytes in UTF-8 that ASCII cannot interpret. The error names the byte and the position, which makes it identifiable but not obviously about encoding at a glance.

Omitting `encoding=` entirely does not mean "figure it out." It means "use this machine's default," which is UTF-8 on most modern systems and something else on others. Passing it explicitly makes the script behave the same everywhere and turns a class of intermittent, machine-specific failures into no failure at all.

## Keep `\r\n` intact with `newline=""`

Reading a Windows-authored file gives you `\n` line endings, whatever is actually stored:

```python
Path("crlf.md").read_text(encoding="utf-8")
# 'a\nb\n'   — the file on disk contains a\r\nb\r\n
```

Python translates `\r\n` to `\n` when reading text and translates back when writing, so most of the time you can ignore line endings entirely. That is the right default: it means a Windows-authored file processes identically to a Unix one.

It matters when you are trying to preserve a file exactly, because reading and rewriting a CRLF file on Linux converts it to LF and shows up as every line changed in a diff. Suppress the translation with `newline=""`, in the one-expression form that needs no `with`:

```python
open("crlf.md", encoding="utf-8", newline="").read()
# 'a\r\nb\r\n'
```

Pass `newline=""` on both the read and the write when a file's endings must survive the round trip.

## See also

- [[Path Objects]] — building the paths these calls take
- [[Inspecting Files and Directories]] — checking a file exists before opening it
- [[Bulk Editing a Docs Folder]] — the backup-and-dry-run discipline for writing to real docs
- [[Strings]] — `splitlines(keepends=True)`, `rstrip()`, and rejoining text
- [[Exceptions]] — catching `FileNotFoundError` and `UnicodeDecodeError`
- [[Moving Renaming and Deleting Files]] — copying a file before you overwrite it
- [[JSON]] — the `json.load(open(...))` row this page's `with` rule explains
