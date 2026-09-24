# Day 5 Report — DEVOPS-005: Git Advanced Workflow & Team Collaboration

**Sprint:** Sprint 9 (29/6 – 12/7)
**Ticket:** DEVOPS-005 — Git Advanced Workflow & Team Collaboration
**Date:** Day 5
**Environment:** Ubuntu 24.04, Git 2.43.0 (continuing the practice repo from Day 4)

---

## Commands Run

Building on the repo created in Day 4, the following commands were run to inspect its remote and history:

```
$ git remote -v
(no output — no remote had been configured yet)

$ git remote add origin https://github.com/example-org/devops-practice-repo.git
$ git remote -v
origin  https://github.com/example-org/devops-practice-repo.git (fetch)
origin  https://github.com/example-org/devops-practice-repo.git (push)

$ git log --oneline
c70c5ac Merge feature/subtract into main, resolve conflict in app.py
b466272 Add multiply() function on main
a26e94b Add subtract() function on feature branch
ad128d4 Add add() helper function
4f8653f Add basic app.py with greet function
ab63152 Initial commit: add README

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

Note: `git remote -v` initially returned nothing because the Day 4 repo was created locally with `git init` and never linked to a remote. A remote named `origin` was added here purely to demonstrate what `git remote -v` shows once one exists — no actual push was made to it in this environment.

---

## Answers to Assigned Questions

### 1. What is a repository?

A repository ("repo") is a project folder that Git is tracking — created with `git init` (or obtained with `git clone`). It contains both the current files and a complete, permanent history of every committed change ever made to them, stored inside a hidden `.git` folder. A repository can exist purely on a local machine, purely on a remote server (like GitHub), or — most commonly — as a local copy linked to a remote one, so changes can be synced between the two.

### 2. What is a commit?

A commit is a saved snapshot of the repository's tracked files at a specific point in time, along with metadata: an author, a timestamp, a commit message describing the change, and a reference to its parent commit(s). Commits are the fundamental unit of history in Git — the chain of commits, each pointing back to the one before it, is what forms a project's timeline. In the practice repo, `ad128d4 Add add() helper function` is an example: a single, self-contained snapshot with a clear message describing exactly what changed.

### 3. What is a branch?

A branch is a lightweight, movable pointer to a specific commit. The default branch is conventionally called `main`. Creating a new branch (e.g. `feature/subtract`) lets you commit new work in isolation, without affecting `main`, and without other branches seeing those commits until they're merged in. Internally a branch is just a small file containing a commit hash — this is why creating or switching branches in Git is nearly instant, unlike copying an entire project folder.

### 4. What is `origin`?

`origin` is simply the conventional *nickname* Git uses for the remote repository a local repo was cloned from (or was explicitly linked to via `git remote add origin <url>`). It's not a special keyword — it's just the default name; a remote could technically be named anything. `git remote -v` lists all configured remotes and their URLs, showing both the fetch (download) and push (upload) address for each — in this repo, `origin` now points to `https://github.com/example-org/devops-practice-repo.git`.

### 5. What is `main`?

`main` is the conventional name for a repository's primary/default branch — the line of history generally considered the current, stable, "production" version of the project (older repos or tutorials sometimes call this `master` instead). It's not technically special to Git itself, just a widely followed naming convention; in this practice repo, `main` is where the `multiply()` commit was made directly, and where the `feature/subtract` branch was ultimately merged back into.

### 6. Why do developers create feature branches?

Feature branches let a developer build, test, and iterate on a change in isolation, without touching the shared `main` branch until that work is actually ready. This has several concrete benefits: `main` stays stable and deployable at all times; multiple developers (or the same developer on multiple tasks) can work in parallel without their in-progress changes colliding; a feature branch can be reviewed (e.g. via a GitHub pull request) before it's merged; and if an experiment doesn't work out, the branch can simply be abandoned or deleted with zero impact on `main`. In the Day 4 exercise, `feature/subtract` let `subtract()` be developed independently while `multiply()` was committed directly to `main` at the same time — exactly the kind of parallel work feature branches are meant to support.

### 7. What happens when two developers modify the same line?

If two branches each change the *same line(s)* of the *same file* since they diverged from a common ancestor, Git cannot automatically determine which version is correct when they're merged — this produces a **merge conflict**. Git pauses the merge, marks the file as "unmerged," and inserts conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) directly into the file showing both competing versions side by side. A person then has to manually edit the file to decide the correct final content — keeping one side, the other, a combination of both, or something new — remove the conflict markers, stage the resolved file (`git add`), and complete the merge with a commit. This was demonstrated directly in the Day 4 repo: `main` and `feature/subtract` both added a new function at the end of the same file, and merging them produced exactly this conflict, which was resolved by keeping both functions.

---

## Summary

Day 5 reinforced the core Git vocabulary — repository, commit, branch, remote/origin, main — by inspecting a real repo's remote configuration and its full commit graph, rather than treating these as abstract definitions. Seeing `git log --graph --oneline --all` render the actual split-and-rejoin shape of the Day 4 merge made concepts like "diverged history" and "merge conflict" concrete rather than theoretical. The recurring theme across both days is that Git's model exists to let people work independently and in parallel with confidence — branches isolate risk, commits create an auditable history, and conflicts are Git's way of asking a human to make a judgment call it can't safely make on its own.
