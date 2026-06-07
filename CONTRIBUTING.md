# Contributing

Conventions for commits, branches, PR/MR titles, and the stable IDs that tie an epic to its stories,
tasks, and shipped code. These are what the gates and checks rely on — a diff that follows them is
traceable back to its task, story, and contract; one that doesn't will fail a gate.

New to the workflow? Start with **`TEAM-GUIDE.md`**; the full reference is **`README.md`**.

---

## Stable IDs (immutable once assigned)

IDs are assigned once and **never renamed** — every downstream link (branch, commit trailer, spec,
PR, build log) is keyed on them, so renaming one breaks the chain.

| Thing | Format | Example |
|-------|--------|---------|
| Epic | `EP-<slug>` (lowercase words + hyphens) | `EP-istifta-inquiries` |
| Story | `EP-<slug>-S0N` (zero-padded) | `EP-istifta-inquiries-S01` |
| Task | `EP-<slug>-S0N-T0N` (zero-padded) | `EP-istifta-inquiries-S01-T03` |

---

## Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/) — **lowercase after the type**. The
**final trailer must be the task ID**; it is the anchor the spec-link gate and the PR read to connect the
diff to its spec and story.

```
<type>: <subject — what changed, lowercase>

<body — what and why, 1–3 lines>

Task: <story-id>-<task-id>
[Contract-Change: yes]
```

- Common `<type>`s: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.
- `Task: <story-id>-<task-id>` (e.g. `Task: EP-istifta-inquiries-S01-T01`) is **required** on
  implementation commits — the spec-link check looks for it.
- `Contract-Change: yes` appears **only** when the diff alters the locked contract surface (see below).
  Omit it for normal work.

Example:

```
feat: create inquiry endpoint

Add POST /inquiries to the agreed contract shape, with validation and
the in-build status default.

Task: EP-istifta-inquiries-S01-T01
```

---

## Branches

One atomic task = one branch = one PR/MR. Branch off the code repo's default branch:

```
feat/<story-id>-<task-id>-<short-slug>
```

`<short-slug>` is 2–4 hyphenated words naming the change. Example:
`feat/EP-istifta-inquiries-S01-T01-create-inquiry`. Never reuse a branch for a different task, and never
fork a second branch for the same task.

---

## PR / MR titles

PR and MR titles follow the same Conventional Commits style (lowercase after the type). The
story/task linkage travels in the **commit `Task:` trailer** and the PR template body — fill the
template's **Story / task** and **Impact & Risk** blocks (`sdlc-pr-template` installs it). Example title:

```
feat: create inquiry endpoint
```

`high` risk (or a touched contract / auth / payments surface) routes the review to domain owners — the
same escalation `sdlc-review-gate` applies. Run `bash checks/risk-route.sh <description>` to list them.

---

## The two hard rules behind the trailers

**File boundary.** Each task in `tasks.md` declares a `Files:` list (≤3 where possible). The diff must
stay inside it. If a task genuinely needs another file, **stop** — treat it as a spec bug, correct the
task's declared files (re-run `sdlc-spec` / re-scope), then implement. A diff that quietly spreads beyond
its declared files is the easiest way to smuggle unreviewed scope past the gates.

**Contract change.** *Consuming* the locked contract (building an endpoint/event/entity to its already
agreed shape) is normal — no trailer. *Changing* the agreed shape itself is not an implementation
decision: stop, go back to the **architecture gate**, amend and re-lock `contract.md`, then implement
with `Contract-Change: yes`. This keeps the contract singular and owned upstream — a code repo can never
widen the shared surface from inside an implementation branch.

Full detail: `skills/sdlc-implement/references/implement-conventions.md`.

---

## Before you push

- Run the check gates: `sdlc-checks repo:<repo> action: run` (spec-link, contract-check, build/test/lint
  must pass).
- Make atomic commits — one logical change per commit.
- Open the PR/MR with the wired template; let the engineer review (a human) be the merge gate.
