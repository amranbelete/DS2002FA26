# 01 — Foundations

The first two weeks of DS2002. Everything here exists to make one loop automatic: **open a notebook in the browser, do the work, get it into your GitHub repo, submit the links in Canvas.** Every later folder assumes you can do that without thinking about it.

| Notebook | Day | What it covers |
|---|---|---|
| `2026-08-26 — Data Science Systems Overview — Lecture.ipynb` | Wed | What a data science system is, and the smallest possible pipeline |
| `2026-08-28 — Environment and GitHub Setup — Lab.ipynb` | Fri | Prove your notebook runs and your repo exists — **graded** |
| `2026-08-31 — Git for Data Science — Lecture.ipynb` | Mon | Where changes live, undoing things, merge conflicts |
| `2026-09-02 — Kaggle and Colab Workflow with Notebook Hygiene — Studio.ipynb` | Wed | Cells that give the same answer every time |
| `2026-09-04 — Submission Drill and Fix a Broken Notebook — Lab.ipynb` | Fri | Debugging drill plus a full submission — **graded** |

Your repo for the semester is `ds2002-fa26`. Notebooks go in folders that mirror this one, and filenames follow `YYYY-MM-DD — Topic — Type.ipynb` exactly.

---

## Colab tips

**Connect once, at the start of a session.** Files panel on the left → **GitHub** tab → **Connect to GitHub**. Sign in as the account that owns *your* repo. If the wrong account is connected, disconnect and reconnect rather than fighting it.

**Two ways to get work into GitHub.** Cloning is less work once you are pushing every week:

```bash
!git clone https://github.com/<you>/ds2002-fa26.git
%cd ds2002-fa26
```

The one-notebook route is **File → Save a copy in GitHub**. Pick your repo and a path like `notebooks/01-foundations/`.

**The runtime is temporary and the repo is not.** Colab wipes `/content` when it disconnects, which it will do if you idle. Anything you committed and pushed is safe on GitHub; anything you did not is gone. Re-run the clone cell and keep going.

**Shell commands need a prefix.** `!` runs one command in a subshell. `%cd` changes directory for the whole session — plain `!cd` does nothing that lasts, which is the single most common reason a `git` command reports the wrong directory.

**Keys and tokens go in the sidebar, never in a cell.** Colab has a key icon for secrets, read with `from google.colab import userdata`. On Kaggle it is Add-ons → Secrets. A token pasted into a notebook is public the moment you push, and deleting it later does not remove it from the history.

**Worth memorizing:** `Ctrl/Cmd+Enter` run cell, `Shift+Enter` run and advance, `Ctrl/Cmd+M B` new cell below, `Ctrl/Cmd+M M` convert to Markdown, `Ctrl/Cmd+Shift+P` command palette.

**On Kaggle instead?** Kaggle has no GitHub connection. Download the notebook and push from your machine, or upload it through the GitHub web interface. Every Git command in these notebooks still runs.

---

## Notebook hygiene

Cell order is invisible to whoever grades your work, so a notebook that only runs because of the order you happened to click is a notebook that does not run.

Before every submission: **Restart the kernel, Run All, top to bottom.** No errors, and outputs actually present. Never overwrite your source data — derive a new column or a new frame from it, so running a cell twice cannot change the answer.

---

## Common commands

The everyday loop, in order:

| Command | What it does |
|---|---|
| `git status` | Which of the four places is my work in? Run this constantly |
| `git add <file>` | Stage a file for the next commit |
| `git add -u` | Stage every file you have already been tracking |
| `git commit -m "message"` | Record the staged changes in local history |
| `git pull` | Bring down commits from GitHub before you start working |
| `git push` | Send your commits to GitHub |

Reading and comparing:

| Command | What it does |
|---|---|
| `git log --oneline` | Compact history, one commit per line |
| `git log --oneline --stat` | Same, plus which files changed |
| `git diff` | Changes you have made but not staged |
| `git diff --staged` | Changes that are staged for the next commit |
| `git show HEAD` | Everything the most recent commit did (`HEAD~1` for the one before) |

Undoing, from safest to most dangerous:

| Command | What it does | Destructive? |
|---|---|---|
| `git restore --staged <file>` | Unstage it; your edit is untouched | No |
| `git revert <hash>` | New commit that undoes an old one — right choice for pushed work | No |
| `git restore <file>` | Throw away an uncommitted edit | **Yes** |
| `git reset --hard` | Throw away everything uncommitted | **Yes — avoid** |

Setup, once per machine:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@virginia.edu"
```

Commit messages are for whoever opens `git log` in three weeks, which is usually you. `Lab 03: add HAVING query for Q4` beats `update`. One idea per commit — if the message needs "and" twice, it should have been two commits.

---

## If things go wrong

Run `git status` before you run anything else. Most confusion is really "I thought my change was somewhere else."

| What you see | What happened | What to do |
|---|---|---|
| `Authentication failed` | GitHub stopped accepting account passwords in 2021 | Use a [personal access token](https://github.com/settings/tokens) as the password, or push with **Save a copy in GitHub** |
| `Updates were rejected... non-fast-forward` | GitHub has commits you do not | `git pull`, resolve anything it flags, then push |
| `Please tell me who you are` | No name or email configured | Run the two `git config --global` lines above |
| `nothing to commit, working tree clean` | You never saved the file, or `.gitignore` skips it | Save it, then check `.gitignore` |
| `file is 142.00 MB; this exceeds GitHub's limit` | A data file got committed | Remove it, add it to `.gitignore`, commit the code that fetches the data instead |
| `fatal: not a git repository` | You are not inside the cloned folder | `%cd ds2002-fa26` — remember `!cd` does not stick |
| No **Connect to GitHub** option in Colab | Blocker or browser issue | Try Chrome, turn off blockers, or an incognito window |
| Wrong GitHub account connected | Colab remembered an old session | GitHub tab → manage the connection → disconnect → reconnect |
| Colab reset and your files are gone | The runtime recycled `/content` | Re-run `!git clone ...`; your pushed commits are safe |
| Notebook shows `<<<<<<<` and will not open | Two people edited the same `.ipynb` | Keep one side whole, re-apply the other person's cells by hand, re-run |

**Notebook conflicts are the one to avoid rather than fix.** A `.ipynb` is JSON holding code, outputs, and an execution counter, so a conflict lands inside that JSON and breaks the file. On the projects: one owner per notebook at a time, pull before you start, and split the work into separate files instead of three people in one.

**You almost never actually lose work.** Anything committed is recoverable even when the branch looks wrong — `git log --oneline` and `git reflog` will show you where it went. Ask in Discord before running anything with `--hard` in it.

---

## Before you ask for help

1. Run `git status` and read it.
2. Restart and Run All — confirm the error is real and repeatable.
3. Post in Discord with the command you ran and the **exact** error text, not a paraphrase.

Questions in the channel help whoever hits the same wall next. Anything personal goes to email.

---

*Last updated: August 2026.*
