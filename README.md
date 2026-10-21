# Emacs Lisp for coding agents

Practical notes for AI agents (and the people running them) that write Emacs
Lisp. Start here; each topic has its own file with the details.

**Run every command from the project root**, with this repository in the
project as `emacs-for-agents/`. The tool is `emacs-for-agents/tools/elisp-agent`; it is not on
`PATH`, so always type that path. Target Emacs 29 or newer unless the
project says otherwise.

## The rules that matter most

1. **Check after every edit.**
   `emacs-for-agents/tools/elisp-agent -L . check FILE` reports missing or
   extra parentheses in plain words, then byte-compiles in a clean Emacs.
   Fix structure first, then compiler warnings, then anything else. Never
   leave a warning behind. → [tools.md](tools.md)
2. **Never guess a function.** Before using a function you have not just seen
   in the code, run `emacs-for-agents/tools/elisp-agent describe NAME`. It
   says whether a clean Emacs has it, which `require` it needs and how to
   call it. Invented functions (`string-length`) and wrong arguments are the
   most common LLM mistake.
3. **Look for a built-in first.** Before writing a helper, run
   `emacs-for-agents/tools/elisp-agent builtins GROUP` (string, list,
   file-name, …). Emacs probably has it. To learn how things fit together,
   read the Elisp manual:
   `emacs-for-agents/tools/elisp-agent info --index elisp TOPIC`.
   → [use-builtins.md](use-builtins.md), [tools.md](tools.md#info)
4. **Edit whole top-level forms.** Replace an entire `defun` rather than
   patching a few parentheses in the middle.
   `emacs-for-agents/tools/elisp-agent forms FILE` gives the line range of
   each form.
5. **Test first, in batch.** Write an ERT test that fails, then make it pass.
   A test that never failed proves nothing. → [testing.md](testing.md)
6. **Follow the conventions.** `lexical-binding`, a package prefix,
   `prefix--private`, `provide`, docstrings, no side effects on load.
   → [best-practices.md](best-practices.md)
7. **Verify in steps.** Batch checks first, then a clean `emacs -Q`, then a
   real Emacs, without disturbing the user's own session. Never change the
   user's Emacs configuration. → [verification.md](verification.md)
8. **No project? Work in a temporary directory.** For a one-off task
   ("write a function that adds two numbers"), following these rules still
   means writing files: the code, a test, probe scripts. Make a new
   directory for them in the system's temporary directory and work there:
   `/tmp` on Linux and macOS (`$TMPDIR` if set; `mktemp -d` makes one), and
   `%TEMP%` on Windows (`C:\Users\NAME\AppData\Local\Temp`). Treat it
   as the project root and link `emacs-for-agents` into it (see
   [Use it in a project](#use-it-in-a-project)). Never write these files to
   the current directory or any other real directory, unless you have been
   given a directory of your own.
9. **Small models: smaller steps.** One form per edit, a check after each, and
   a hard limit on retries. → [small-models.md](small-models.md)

Work is done only when `check` and `test` pass, and you show their output.

## Files

| File | What it covers |
|---|---|
| [tools.md](tools.md) | `elisp-agent`: structure check, compile, describe, built-ins, manuals, eval, tests |
| [best-practices.md](best-practices.md) | Conventions: files, naming, loading, data, regexps, modes, buffers, processes |
| [use-builtins.md](use-builtins.md) | What not to reinvent, and how to find what exists |
| [testing.md](testing.md) | ERT tests: layout, fixtures, mocking, testing commands and buffers |
| [verification.md](verification.md) | The verification ladder, debugging, working in a live Emacs |
| [small-models.md](small-models.md) | Workflow for small local LLMs: parentheses, invented functions, loops |
| [pitfalls.md](pitfalls.md) | Traps that cost real debugging time, each with the fix |
| [templates.md](templates.md) | Skeletons for a package file, a major mode and a test file |
| [background.md](background.md) | Why this exists, related work, sources |

## Quick start

```sh
T=emacs-for-agents/tools/elisp-agent
$T -L . check mypkg.el              # parens, then byte-compile; exit 1 on problems
$T describe string-trim seq-reduce  # does it exist, and how is it called?
$T builtins string                  # built-in string functions
$T info --index elisp add-hook      # find it in the Elisp manual
$T info elisp "Setting Hooks"       # read that section
$T -L . --require mypkg eval '(mypkg-parse "a,b")'
$T -L . test test/mypkg-test.el     # run ERT
```

## Use it in a project

This README is the entry point; no other instruction file is needed. Make
it available in the project root as `emacs-for-agents/`: copy it there, or
link to your clone (and add `emacs-for-agents` to the project's
`.gitignore`):

```sh
ln -s /path/to/emacs-for-agents emacs-for-agents
```

Then tell the agent:

> Read emacs-for-agents/README.md and follow it.
