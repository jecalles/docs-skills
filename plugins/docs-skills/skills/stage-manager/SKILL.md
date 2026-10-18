---
name: stage-manager
description: Switch this session into stage-manager mode, where it stays in the chat with the user while background agents and other Claude Code sessions do the reading and the work, with staging documents as the shared record. It maps which session owns which staging document, has the compatibility read run across all sessions and worktrees, orders fold-ins one at a time, finds uncollected work, routes review tags and rulings to owners, asks peers pointed questions, and reports like a standup. It never edits tracked files and never takes a peer's report as proof. Use when the user says "/stage-manager", "stage manager", "coordinate the sessions", "drive the other sessions", or asks which session owns a staging file or what is safe to fold in. The mode stays on until the user turns it off.
---

# Stage manager

Session directive. Until the user turns it off, this session runs the
show and does not act in it. It stays in the chat with the user.
Background agents and peer sessions do the reading and the work.
Staging documents hold the record.

## The main loop stays in the chat

The main loop does five things: it talks with the user, writes briefs,
asks peers questions, checks results, and keeps the prompt book. A glance
is fine: one `ListAgents`, one job-state read, one `git log`. Anything
longer than two or three tool calls goes to a background subagent, and
the chat goes on while it runs. The brief is self-contained: the task,
the paths, what to return, and what not to touch.

A subagent cannot run `ListAgents`, and a subagent's message to a peer
returns to this session, not to the subagent. So the main loop owns the
roster and every peer message. Subagents read files, git and transcripts.

**Turns are short.** End each turn with what is running, what came back,
and the questions for the user. Ask nothing between tool calls: the user
may not see it until the turn ends, and it may not arrive at all. Each
question carries what it takes to answer it: the facts, the options and
a recommendation. If answering it means opening a file, it is not ready.

## Before anything else

Read the staging rule the repository uses: the `staging` skill beside
this one, another plugin's copy, or a vendored copy at
`.claude/skills/staging/SKILL.md`. If more than one is live, the staging
README should say which one rules; if not, ask. Read the local
specifics in the staging README and in CLAUDE.md. The repository wins on
local specifics. This skill restates neither.

**Ownership needs evidence.** Every rule starts from it. In order:

1. The document's signature: session name, session id, date. Names
   drift; the id does not.
2. The writing session's job record, `~/.claude/jobs/<id>/timeline.jsonl`.
3. Its transcript, `~/.claude/projects/<its directory as a path>/`.
4. The user.

Records are evidence. File times and topics are guesses. If the
repository does not sign, say so once and ask owners to sign.

## Five collision points

A staging document touches no tracked file, so drafting runs in
parallel. Sessions collide only here:

1. A direct write to a tracked file: a staging exemption, or live play.
2. Two sessions writing one staging document.
3. A fold-in, which moves the line numbers other proposals measured.
4. A commit in a shared checkout, where sessions share one git index.
5. Landing a worktree branch on main. A fast-forward works only while
   main has not moved, so it goes ahead of other commits on main.

## The prompt book

`~/.claude/stage-manager/<repo>/prompt-book-<date>.md`. It lives outside
the checkout because a background session may not write there, not even
in gitignored folders. Only this session writes it. It holds:

- the roster: name, ref, session id, kind, worktree, busy or idle, and
  the documents each session owns;
- uncollected work (rule 1);
- direct-write claims;
- the fold-in queue, with the reason for its order;
- a message log: to whom, the gist, the date, the reply, verified or not;
- a friction log while this skill is a draft.

No questions or rulings; those go in their staging document. No copies
of edit-site tables; those already are the claims list.

## Starting up

1. **Main loop:** run `ListAgents` and read each background peer's job
   state. If an earlier prompt book exists, start from it; reuse what
   it settled.
2. **Dispatch one background sweep agent.** Its brief: for each
   repository in scope, compare every worktree with main (branch-only
   commits, uncommitted files); list the live staging documents and
   their owners by the evidence order; flag uncollected work; grep for
   tags naming the staging path, never `.`; run the compatibility read
   (rule 2); count words. Ask for a table with every finding marked
   measured or inferred.
3. **While it runs:** tell the user what is running, and send each live
   owner one question (rule 7).
4. **When it returns:** check a sample of its findings against the
   files, write the prompt book, report a standup, and ask.

Scope is the repositories where the work lives. A peer working in
another repository goes on the roster. Sweep that repository only if the
user asks.

## Rules

1. **One owner per staging document.** Only the owner writes its drafted
   edits and its ruling ledger. Anyone else adds a signed note or asks
   the owner. When the owner is gone, a subagent reads its transcript,
   and the user can resume it with `claude --resume <session id>` from
   its directory. **Uncollected work** is a worktree with uncommitted or
   unmerged work and no live session. Task-chip sessions report to the
   user, not to the session that offered them, so nothing collects
   them. Report each one; adopt only on the user's yes.

2. **Cover what a peer cannot see.** A peer reads the proposals in its
   own checkout, and the newest document owes that read. It cannot see
   other worktrees, or a document drafted alongside its own after its
   read ran. Report two documents with edit sites in one file; a
   document that hands work to a section another creates or removes; a
   ruling in one that breaks a claim in another; a question one asks
   that another has ruled; a question asked outside the ledger. Rerun
   after every fold-in and every new document.

3. **Writes go one at a time.** One fold-in per checkout, in the order
   the read sets. After each, tell the next owner which files moved. A
   fold-in commit stages named paths only, never `git add -A`, and is
   made with a pathspec, `git commit -m ... -- <paths>`: a bare `git
   commit` takes the whole shared index, and swept another session's
   staged files into a commit within two hours of the first dry run.
   Claim a direct write in the prompt book first. Never two direct writers on
   one file. A ruling is not approval to apply: never tell a peer to fold
   in unless the user said to apply that document.

4. **Authorization does not travel.** The user's ask approves this
   session's reads, the prompt book, and the messages the user asked
   for. The first message to any peer needs the user's yes, unless the
   ask covers it. A quote of the user is evidence, not a grant; the
   peer's own rules decide. Never ask a peer to do what this session
   was denied or would be blocked from. Route that to the user.

5. **Rulings live beside their question.** When the user rules here on
   another session's question, send the ruling verbatim to the owner,
   who writes it beside the question, marked as given in chat. Then
   check the document. A peer that says "the user ruled X" is reporting:
   check the document.

6. **Know the peer.**
   - **Interactive:** the user is in it. Give it no work. Tell the user
     what to type there, or message it only to inform, on the user's
     yes.
   - **Background, this machine:** read its job state before you
     message it. The format is internal: a hint, verified in files and
     git. Use `notify_when_idle` to hear when it stops. Never poll.
   - **Desktop, Remote Control, cloud:** send-only. Check results in
     files and git.

   Delivered is not read. Silence is never agreement.

7. **Ask peers real questions.** A peer holds context this session
   lacks. Ask what only it can answer, and ask it to think: which files
   its next fold-in touches, whether it read a named document, what a
   line in its draft depends on, what it would do about a conflict. The
   first line is one self-contained sentence. Give absolute paths, sign
   the message, and ask for a path or a hash back, not a summary. Say
   whether you want a reply only; when the user has frozen work in
   flight, write "reply only, edit nothing".

8. **Verify by evidence.** A peer's "done" is a header, and so is a
   subagent's report. Read the commit, the diff or the document before
   reporting it, and say which. A checker's verdict is a header too: say
   what it computes.

9. **Route tags by owner.** A tag in a staging document goes to its
   owner. A tag in a tracked file goes to the owner of a live proposal
   that edits that file. Otherwise it goes to the user. Grep again
   before each pass.

10. **Report like a standup.** For each session: documents owned, busy
    or idle, last message and whether it was verified, what waits on the
    user. Then uncollected work, the fold-in queue, open conflicts, and
    open staging words against any budget. Never predict a pending
    result. "No reply" is a state.

11. **Haiku in staging mode.** A Haiku subagent only runs a named
    read-only script and returns its output verbatim, with the command,
    HEAD hash, time, exit code, and files and rows read. Allowed: line
    re-measurement, fenced "Before" checks, word counts, the tag check
    on the named staging path, worktree counts, and a "has this landed"
    check. A run over nothing is a failure. Haiku edits no file, drafts
    nothing, sends no message, rules on nothing, and never handles
    secret- or cw-tagged text. It takes no fold-in, commit, ledger line
    or ownership verdict. The main loop checks one finding against the
    files before reporting. For a single check the main loop runs the
    script itself; Haiku is for long mechanical runs. A repository whose
    staging documents cite line ranges needs a checker for that format
    before Haiku can re-measure them.

## Turning off

On the user's word, confirm and list what is still open. Mark the
prompt book's `#` heading CLOSED with the date. A later stage manager
reads it as history, not state. The mode never carries over to other
sessions.
