# Day 4 Report — DEVOPS-004: Git & GitHub Fundamentals (Industry Workflow)

**Sprint:** Sprint 9 (29/6 – 12/7)
**Ticket:** DEVOPS-004 — Git & GitHub Fundamentals (Industry Workflow)
**Date:** Day 4
**Environment:** Ubuntu 24.04, Git 2.43.0

---

## 1. Git Concepts Learned

- **Repository (repo):** A folder tracked by Git, holding both the current files and the full history of every change ever committed to them. `git init` turns a plain folder into a repo by creating a hidden `.git` directory that stores this history.
- **The three-stage workflow (working directory → staging area → repository):** A change first exists only in the working directory. `git add` moves it into the staging area (an index of what will go into the *next* commit). `git commit` then permanently records the staged snapshot into the repository's history. This staging step is what lets you commit only part of your changes at a time, rather than everything you've touched.
- **Commits as snapshots:** Each commit captures the full state of the tracked files at that point, plus metadata — author, timestamp, and a message — and points back to its parent commit(s). This chain of parent pointers is what forms the project's history.
- **Branches:** A branch is just a movable pointer to a commit. Creating a branch (`git checkout -b`) lets you develop a change in isolation from `main` without affecting it, then bring the work back in later. This is the standard way teams work on features, fixes, or experiments in parallel without stepping on each other.
- **Merging & merge conflicts:** `git merge` combines the history of two branches. If both branches changed the *same* lines of the *same* file since they diverged, Git can't automatically decide which version is correct — it stops and asks a human to resolve the conflict by editing the file directly, choosing (or combining) the competing changes.
- **Diverged history:** Once two branches each have commits the other doesn't, they've "diverged." `git log --graph` visualizes this as two separate lines that split from a common ancestor and (optionally) rejoin later at a merge commit.
- **Merge commits:** A merge commit is unusual in having *two* parent commits instead of one — it's the point where two diverged histories are formally reunited into a single line of history.

---

## 2. Git Commands Used

| Command | Purpose |
|---|---|
| `git init` | Initialize a new, empty Git repository in the current folder |
| `git config user.name / user.email` | Set the identity attached to commits made in this repo |
| `git branch -M main` | Rename/set the default branch to `main` |
| `git add <file>` | Stage a file's changes for the next commit |
| `git commit -m "<message>"` | Record the staged changes as a new commit with a message |
| `git status` | Show the current state of the working directory and staging area |
| `git log --oneline` | Show a compact, one-line-per-commit view of history |
| `git log --oneline --graph --all` | Visualize commit history as a graph, including all branches |
| `git checkout -b <branch>` | Create a new branch and switch to it in one step |
| `git checkout <branch>` | Switch the working directory to an existing branch |
| `git branch` | List local branches (current branch is marked) |
| `git merge <branch>` | Merge another branch's history into the current branch |
| `git branch -d <branch>` | Delete a branch that's already been merged in |

---

## 3. Commit History

A small Python project was built up commit by commit to practice a realistic history:

```
$ git log --oneline
ad128d4 Add add() helper function
4f8653f Add basic app.py with greet function
ab63152 Initial commit: add README
```

Detail on the most recent commit at that point (`git log -1`):

```
commit ad128d456c1ac17ee1fc2cae50dc694d45da9a31
Author: DevOps Learner <learner@example.com>
Date:   Tue Aug 11 2026

    Add add() helper function
```

After the branch/merge work below, the full history (including the merge commit) looks like:

```
$ git log --graph --oneline --all
*   c70c5ac Merge feature/subtract into main, resolve conflict in app.py
|\
| * a26e94b Add subtract() function on feature branch
* | b466272 Add multiply() function on main
|/
* ad128d4 Add add() helper function
* 4f8653f Add basic app.py with greet function
* ab63152 Initial commit: add README
```

Each commit is a discrete, meaningful snapshot (add README → add a function → add another function → branch → merge), which is the kind of small, descriptive commit history real teams expect in code review.

---

## 4. Branch Workflow

The standard **feature-branch workflow** was practiced end to end:

1. **Branch off `main`:** `git checkout -b feature/subtract` created a new branch starting from the current tip of `main`, to isolate a new `subtract()` function.
2. **Commit on the feature branch:** Work was committed *only* to `feature/subtract`, leaving `main` untouched — `git branch` at that point confirmed two branches existed, with `feature/subtract` checked out.
3. **`main` kept moving too:** Switching back to `main` (`git checkout main`) and committing a *different* function (`multiply()`) simulated the real-world case where `main` doesn't stand still while a feature branch is being developed — both branches now had commits the other didn't (diverged history).
4. **Merge back in:** `git merge feature/subtract` while on `main` attempted to combine both histories back into one line.
5. **Clean up:** After the merge was committed, `git branch -d feature/subtract` deleted the now-merged branch, since its work is preserved permanently inside `main`'s history.

This mirrors how a feature branch is typically used on a team: branch → develop in isolation → merge back into `main` once ready → delete the now-redundant branch.

---

## 5. Merge Conflict Explanation

Because both `main` and `feature/subtract` added a new function to the *same location* (the end of `src/app.py`) after diverging from the same commit, Git could not automatically decide which addition should come first — this is exactly the condition that produces a merge conflict.

**What `git merge feature/subtract` returned:**
```
Auto-merging src/app.py
CONFLICT (content): Merge conflict in src/app.py
Automatic merge failed; fix conflicts and then commit the result.
```

**What Git left inside the conflicted file**, using conflict markers to show both competing versions:
```python
def add(a, b):
    return a + b

<<<<<<< HEAD
def multiply(a, b):
    return a * b
=======
def subtract(a, b):
    return a - b
>>>>>>> feature/subtract
```

- Everything between `<<<<<<< HEAD` and `=======` is what's currently on the checked-out branch (`main`).
- Everything between `=======` and `>>>>>>> feature/subtract` is what's coming in from the branch being merged.

**Resolution:** Since both functions are independent and valid, the fix was to keep *both* — the conflict markers were manually removed and the file edited so it contained `multiply()` and `subtract()` one after another, with no markers left behind. The resolved file was then staged (`git add src/app.py`) and the merge finalized with `git commit`, which automatically created a **merge commit** (`c70c5ac`) with two parents — one from each branch — permanently recording that these two lines of history were joined here.

The key lesson: a merge conflict isn't Git making a decision incorrectly — it's Git correctly recognizing it *can't* safely guess the right outcome, and handing that judgment call to a person.

---

## 6. Problems Faced

1. **Initial `git branch -M main` failed silently on a brand-new repo with no commits.**
   *Cause:* A branch name can't be meaningfully "renamed" before it has at least one commit to point to — an empty repo has no branch ref yet.
   *Resolution:* Made the first commit (README) before worrying about branch naming; `main` became the active branch naturally once history existed.

2. **Merge conflict on `src/app.py` when combining `main` and `feature/subtract`.**
   *Cause:* Both branches independently appended a new function to the same spot in the same file after diverging from a shared ancestor commit, so Git had two different, incompatible versions of the same lines and couldn't auto-resolve them.
   *Resolution:* Manually edited the file to remove the `<<<<<<<`/`=======`/`>>>>>>>` conflict markers, kept both functions, staged the resolved file, and completed the merge commit — the standard, expected way to handle a content conflict.

3. **Remembering to delete the feature branch after merging.**
   *Observation:* Once `feature/subtract` was merged, leaving it in place would still work fine, but real teams delete merged branches promptly to keep the branch list clean and avoid confusion about which branches still have unmerged work. `git branch -d` refuses to delete a branch with *unmerged* changes, which is a useful safety check against accidentally losing work.

---

## 7. Summary

Day 4 covered the practical Git workflow used on real engineering teams: initializing a repository, building a meaningful commit history with `add`/`commit`, branching off `main` to isolate new work, and merging that work back in — including handling a genuine merge conflict caused by two branches editing the same part of a file. The most important takeaway is that Git's three-stage model (working directory → staging → repository) and its branch-and-merge system exist specifically to let multiple people (or the same person working on multiple things) make independent changes safely, with conflicts surfaced explicitly rather than silently overwritten. Being comfortable reading conflict markers and resolving them deliberately — rather than panicking or discarding one side's work — is a core, everyday DevOps/engineering skill.
