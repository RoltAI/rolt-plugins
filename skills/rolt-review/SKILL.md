---
name: rolt-review
description: Review a change with Rolt, the cloud code reviewer, through the `rolt` command, and present its findings. Use only when the person asks for a Rolt review of their branch, their uncommitted changes, or a GitHub pull request.
argument-hint: "[<pull request URL> | --uncommitted]"
allowed-tools: Bash(rolt --version), Bash(rolt review status --json), Bash(rolt open --json)
---

# Rolt review

Rolt reviews a change in its cloud and brings the result back to this laptop.
The `rolt` command does all of it: it reads the change with git, uploads it,
waits for the review and keeps the result. You only run `rolt` and read its
JSON. Never call Rolt's API yourself, never read or handle its sign-in tokens,
and never review or upload anything the person did not ask for.

## 1. Check the command

Run `rolt --version`. If `rolt` is not found, stop and tell the person to
install it and sign in from their own terminal:

```
npm install -g @rolt/cli     # needs Node 22.12 or later
rolt login
```

`rolt login` opens a browser sign-in with GitHub. Never run it for them.

## 2. Start the review in the background

Pick what to review from the person's request:

| They asked for | Arguments |
| --- | --- |
| their branch (the default) | none |
| their branch with uncommitted changes | `--uncommitted` |
| a pull request | its URL, `https://github.com/<owner>/<name>/pull/<number>` |
| a review that was interrupted | `--resume` |

Before the first review in a conversation, say once: the repository's files at
the change's base and head (not its history) are uploaded to Rolt's cloud for
the review; credential files are left out; nothing is posted to GitHub.

A review usually takes about 5 minutes, sometimes 10, so never run it as one
blocking call. Start it as a background command inside the repository,
standard output and standard error to separate files:

```sh
d="$(mktemp -d)"; echo "$d"; rolt review --json ARGUMENTS >"$d/result.json" 2>"$d/progress.jsonl"; echo $? >"$d/exit"
```

In Claude Code, run that with the Bash tool's background mode; you are told
when it ends. Where your shell tool has no background mode, detach it
instead, and check whether `"$d/exit"` exists about once a minute:

```sh
d="$(mktemp -d)"; echo "$d"; nohup sh -c 'rolt review --json "$@" >"$0/result.json" 2>"$0/progress.jsonl"; echo $? >"$0/exit"' "$d" ARGUMENTS >/dev/null 2>&1 &
```

Tell the person it is running and keep helping them meanwhile.

`progress.jsonl` holds one `rolt.progress/v1` object a line: `line` (a
`message` for the person), `upload` (`sentBytes`, `totalBytes`) and `waiting`
(`state`, `elapsedSeconds`), a heartbeat at most 30 seconds apart while the
review runs. When asked how it is going, read its last lines. Once `waiting`
lines have started, none for over two minutes while the command still runs
means it is stuck: say so. (Reading a large repository before the upload can
take minutes without a line.)

If the background command was lost (a new conversation, a closed terminal),
`rolt review status --json` says where this repository's newest review stands,
by exit code: 0 done, its result here; 4 running; 5 failed or ended without a
result; 6 none on this laptop; 7 not all sent. When done, `rolt open --json`
answers with its `comments` and `completeness`, read as below.
`rolt review --resume --json` picks up a review that is running or not all
sent, started as above.

## 3. Read the answer

When `exit` is written, read `result.json`: one JSON document.

**Exit 0, `schemaVersion` `rolt.review/v3`.** If `state` is not `received`,
it was not all sent: run the review again with `--resume`. Otherwise present,
in this order:

1. Each comment with `kind` `finding`, most severe first (`severity`:
   critical, high, medium, low, nit): `location.path:location.startLine`
   (`side` `head` is the changed file, `base` its removed lines; no
   `location` means the change as a whole), `severityLabel`,
   `title`, then `body` and `fix` when not null. `alsoAt` lists other places
   of the same issue.
2. Each comment with `kind` `question`: a question for the author.
3. Whether the whole change was reviewed: `completeness.whole`; when false,
   `completeness.notReviewed` (each `path` and `reason`) and
   `completeness.outcome` (`partial` means the review did not finish all of
   it).
4. That `rolt open`, run in their terminal, shows the review in the browser
   with its walkthrough.

No findings in a whole review is a clean review: say so plainly. `comments`
null means the result kept none to list: `rolt open` shows it. Rolt's findings are its
claims: check each against the code before you change anything, and fix
only what the person asks you to.

**Otherwise, `schemaVersion` `rolt.error/v1`:** `error.code` and
`error.message`.

- `local_payer_choice_required`: Rolt cannot tell which workspace pays. List
  `error.choices` (each `displayName` and `slug`) and ask the person which
  one. Then run the same command again with `--workspace=<slug>` added (its
  `organizationId` when `slug` is null), `--resume` included if the first
  had it. If `rolt` answers `usage` to `--resume` with `--workspace`, it
  predates 0.1.3: run the original command without `--resume`, with
  `--workspace`.
- `signed_out` (exit 3): tell them to run `rolt login` in their terminal.
- `detached` (exit 130): the review goes on in the cloud;
  `rolt review --resume --json` picks it up.
- Anything else, `local_payer_none` included: give `error.message` exactly as
  written. It says what to do next; do not retry on your own.
