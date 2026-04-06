# Module 3 – Answers | Git (Version Control)

> **Role context:** Infrastructure and Cloud Analyst (Intermediate) — Air Canada
> **Interview depth:** Production-grade, enterprise-level answers

---

## Assignment 1 – Git Fundamentals & Repository Setup

---

### Git Architecture: What Problem Does Git Solve?

Before Git, teams used centralised version control systems (CVS, SVN). If the central server went down, no one could commit, diff, or view history. If two developers edited the same file simultaneously, the second person's changes overwrote the first — with no merge mechanism. Rolling back a broken release meant manually finding and reverting files.

**Git is a Distributed Version Control System (DVCS)**. Every developer has a full copy of the entire repository history on their local machine. You can commit, branch, diff, and log completely offline. The central remote (GitHub/GitLab/Azure DevOps) is just one node of many — not a single point of failure.

| Feature | Centralised (SVN) | Distributed (Git) |
|---|---|---|
| Work offline | No | Yes |
| Single point of failure | Yes | No |
| Branching cost | Expensive (full copy) | Cheap (just a pointer) |
| Speed | Network-dependent | Local operations are instant |
| Rollback | Complex | `git revert` or `git reset` |

---

### Git Internals: Blobs, Trees, Commits & Snapshots

Git doesn't store diffs (line-by-line changes). It stores **snapshots** — a complete picture of all tracked files at every commit. This is why Git is fast and why rollbacks are simple.

**The four Git object types:**

```
BLOB   → Stores the raw content of ONE file (no filename, just content)
         Each unique content = unique blob (content-addressed via SHA-1)

TREE   → Stores a directory listing: filenames + permissions + pointer to blob/subtree
         Like an inode table — maps names to content

COMMIT → Points to one tree (the root snapshot) + parent commit(s) + author + message
         A commit is a snapshot in time, not a diff

TAG    → A named pointer to a specific commit (annotated tags store extra metadata)
```

**Visual:**

```
Commit C3
  │
  ├── tree (root)
  │     ├── blob: README.md  (SHA: abc123)
  │     ├── blob: app.py     (SHA: def456)
  │     └── tree: src/
  │           └── blob: main.py (SHA: ghi789)
  └── parent: Commit C2
```

**Why this matters in production:**
- Understanding blobs explains why `git stash`, `git cherry-pick`, and `git bisect` work at the object level — you're not copying files, you're swapping tree references.
- Merge conflicts happen at the blob level — Git compares three blobs: common ancestor, HEAD, and theirs.
- Corrupted repos (rare but real) are diagnosed by understanding object store integrity.

---

### Git Working Areas

```
┌────────────────────────────────────────────────────────────────┐
│  Working Directory    Staging Area (Index)    Local Repo       │   Remote Repo
│  (your files)         (.git/index)            (.git/objects)   │   (GitHub/ADO)
│                                                                │
│  [edit files]──git add──►[staged]──git commit──►[committed]───git push──►[remote]
│       ◄──git restore──────◄                ◄──git reset──      │
│                                                ◄──git pull──────────────◄
└────────────────────────────────────────────────────────────────┘
```

- **Working Directory**: Your actual files. `git status` shows what's changed.
- **Staging Area (Index)**: Files you've explicitly chosen for the next commit. `git add` moves files here. This lets you commit partial changes — critical when you've fixed two unrelated bugs in one session.
- **Local Repository**: The `.git` folder — your complete history, all objects, all branches. Deleting `.git` destroys all history for that repo.
- **Remote Repository**: GitHub, GitLab, Azure DevOps, Bitbucket. Push/pull syncs between local and remote.

---

### Installation & Configuration

```bash
# Install (Ubuntu/Debian)
sudo apt update && sudo apt install git -y

# Install (macOS)
brew install git

# Verify
git --version

# Configure identity (MUST do before first commit — used in every commit object)
git config --global user.name "Dhruwang"
git config --global user.email "dhruwang@aircanada.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main

# View config
git config --list
cat ~/.gitconfig
```

---

### First Repository: Local → Remote Flow

```bash
# 1. Create local repo
mkdir git-learning-devops && cd git-learning-devops
git init

# 2. Create files
echo "# Git Learning" > README.md
echo "DevOps Git notes" > notes.txt

# 3. Check states
git status
# Output: Untracked files: README.md, notes.txt

# 4. Stage files
git add README.md notes.txt
git status
# Output: Changes to be committed (staged)

# 5. Commit
git commit -m "feat: initial project structure with README and notes"

# 6. Add remote
git remote add origin https://github.com/dhruwang/git-learning-devops.git
git remote -v

# 7. Push with upstream tracking (-u sets default remote/branch for future git push/pull)
git push -u origin main
```

---

### Commit Message Best Practices (Conventional Commits)

```
<type>(<scope>): <short description>

Types:
  feat      → new feature
  fix       → bug fix
  docs      → documentation only
  chore     → maintenance (dependencies, config)
  refactor  → code change that neither fixes a bug nor adds a feature
  test      → adding or fixing tests
  ci        → CI/CD pipeline changes
  perf      → performance improvement

Examples:
  feat(auth): add JWT token refresh endpoint
  fix(booking): resolve race condition in seat lock logic
  docs(readme): add SSH setup instructions
  chore(deps): upgrade node to 20.x
  ci(pipeline): add SonarQube quality gate to PR workflow
```

**Why this matters at Air Canada**: CI/CD pipelines parse commit messages. A `feat:` commit in the booking service triggers a minor version bump and full regression suite. A `fix:` commit triggers a patch release. Bad commit messages break automated semantic versioning and release notes generation.

---

### Git vs GitHub | Clone vs Pull | Commit vs Push

| Comparison | Explanation |
|---|---|
| **Git vs GitHub** | Git is the version control tool (local CLI). GitHub is a hosted service built on Git — adds PRs, CI/CD integrations, branch protection, code review. Git works without GitHub; GitHub requires Git. |
| **Clone vs Pull** | `git clone` creates a new local copy of a remote repo (first time). `git pull` fetches and merges updates into an **existing** local repo (ongoing sync). |
| **Commit vs Push** | `git commit` saves a snapshot to the **local** repository. `git push` sends those local commits to the **remote**. You can commit 50 times locally without affecting anyone else — push when your work is ready to share. |

---

## Assignment 2 – Branch Management & Git Diff

---

### What is a Git Branch? (Internal Model)

A branch in Git is **just a file containing a 40-character commit SHA**. It's a lightweight pointer that moves forward automatically with each new commit.

```
main ──────────────► commit C5
                      ▲
feature-auth ─────────┘ (created from C5, now at C7)
                       └──► C6 ──► C7
```

**Creating a branch is free** — it's just creating a new pointer file. This is fundamentally different from SVN where branching copied the entire directory tree.

---

### Why Never Work Directly on `main`?

`main` (or `master`) is the production branch — the branch CI/CD deploys from. Working directly on it means:
- Any in-progress work is immediately visible to the entire team
- A broken commit can halt everyone's work and trigger failed deployments
- There's no review gate — code goes straight to prod
- You can't work on two features simultaneously

**Branch = isolated workspace**. Your experiments don't affect anyone else until you open a Pull Request and it passes review + CI gates.

---

### Branch Commands

```bash
# Check current branch
git branch
git branch -r          # Remote branches only
git branch -a          # All (local + remote)

# Create branch
git branch feature-auth

# Switch (legacy)
git checkout feature-auth

# Create + switch (modern — preferred)
git switch -c feature-auth

# Create from a specific branch (not current)
git switch -c hotfix/login-fix main

# Push with upstream tracking
git push -u origin feature-auth
# -u = --set-upstream: links local branch to remote branch
# After -u, you can just run 'git push' / 'git pull' without specifying remote/branch

# Rename branch
git branch -m feature-auth feature-authentication  # rename locally
git push origin -u feature-authentication          # push under new name
git push origin --delete feature-auth              # delete old remote branch

# Delete branch
git branch -d feature-authentication    # safe: fails if unmerged commits exist
git branch -D feature-authentication    # force: deletes regardless of merge status
git push origin --delete feature-authentication  # delete remote branch
```

---

### `git diff` — The DevOps Engineer's Debugging Tool

```bash
# Working directory vs last commit (unstaged changes)
git diff

# Staged changes vs last commit (what will go into next commit)
git diff --staged
git diff --cached   # same thing

# Compare two branches
git diff main feature-auth

# Compare specific files across branches
git diff main feature-auth -- src/app.py

# Compare two commits
git diff a1b2c3d e4f5g6h

# Summary only (no line diffs, just file names and stats)
git diff --stat main feature-auth

# Show word-level diff (useful for documentation)
git diff --word-diff main feature-auth
```

**Why `git diff` matters in production:**
- **Debug broken deployments**: `git diff v1.1.0 v1.2.0` shows exactly what changed between releases — isolate which change caused the regression
- **PR reviews**: Reviewers use diff to understand scope of changes before approving
- **Validate pipeline triggers**: Before pushing, confirm you're only changing what you intended — avoid accidentally committing a debug flag or env var change
- **Audit**: `git diff HEAD~1` quickly shows what the last deployment changed

---

### Branch Naming — Why It Impacts CI/CD

Branch names drive pipeline automation:

```yaml
# Example: GitHub Actions pipeline triggers
on:
  push:
    branches:
      - main          # → deploy to production
      - release/**    # → deploy to staging
      - feature/**    # → run unit tests + SAST only
      - hotfix/**     # → fast-track to prod pipeline with approvals

# If your branch is named "myfix" instead of "hotfix/login-fix",
# it won't trigger the hotfix pipeline — a critical miss during an incident
```

**Air Canada naming convention example:**
```
feature/<jira-ticket>-<short-description>   → feature/AC-1234-add-loyalty-points-api
bugfix/<ticket>-<description>               → bugfix/AC-5678-fix-seat-map-null-pointer
hotfix/<ticket>-<description>              → hotfix/AC-9012-critical-payment-timeout
release/<version>                          → release/2.4.1
chore/<description>                        → chore/upgrade-node-18
```

---

## Assignment 3 – Merge, Rebase & Conflict Resolution

---

### Git Merge: Three Types

```bash
# 1. FAST-FORWARD MERGE (no divergence — feature branch is ahead of main)
git checkout main
git merge feature-readme
# Git simply moves the main pointer forward — no merge commit created
# Clean, linear history

# 2. THREE-WAY MERGE (both branches have new commits — common in teams)
git merge feature-auth
# Git finds the common ancestor, creates a NEW merge commit
# History shows the merge point — useful for tracking when features landed

# 3. SQUASH MERGE (all feature commits collapsed into one on main)
git merge --squash feature-auth
git commit -m "feat: add authentication flow"
# Feature branch history is hidden — main stays clean
```

---

### Merge vs Rebase: The Critical Distinction

| Aspect | Merge | Rebase |
|---|---|---|
| History | Preserves full branch history with merge commit | Creates linear history — replays commits on top of target |
| Merge commit | Yes (three-way merge) | No |
| Safety | Safe on shared branches | **Never rebase shared/public branches** |
| Use case | Integrating feature into main | Updating a feature branch with latest main |
| Conflict frequency | Once (at merge time) | Once per replayed commit |
| Auditability | Full context of when/what merged | Cleaner log, harder to trace original commit sequence |

```bash
# REBASE PATTERN (feature branch workflow)
git checkout feature-auth
git rebase main
# This replays your feature commits on top of the latest main
# Your feature branch now appears as if it was created from the current main

# If conflict during rebase:
# 1. Fix the conflict in the file
git add conflicted-file.py
git rebase --continue
# OR abort entirely:
git rebase --abort

# After rebase, push needs force (history was rewritten)
git push --force-with-lease origin feature-auth
# --force-with-lease is safer than --force: fails if someone else pushed to the branch
```

**Golden Rules:**
- `main` is **always merged into**, never rebased
- Rebase is for keeping your **feature branch** up to date with main
- Never `git push --force` on `main` — ever

---

### Conflict Resolution: Step-by-Step

```bash
# Conflict markers in a file:
<<<<<<< HEAD
Deployment strategy: Rolling          ← your current branch (HEAD)
=======
Deployment strategy: Blue-Green       ← incoming branch being merged
>>>>>>> conflict-feature-2

# Resolution steps:
# 1. Open the file, decide what the final state should be
# 2. Remove ALL conflict markers (<<<, ===, >>>)
# 3. Leave only the correct content:
Deployment strategy: Blue-Green

# 4. Stage the resolved file
git add notes.txt

# 5. Complete the merge
git commit -m "merge: resolve deployment strategy conflict — blue-green chosen per RFC-42"

# OR abort and start over
git merge --abort
```

**Production conflict resolution mindset:**
- Never resolve a conflict alone if you don't understand what both sides do — call the author
- The merge commit message should explain **why** a particular resolution was chosen, not just "resolve conflict"
- After resolving, always run tests before pushing — a syntactically correct merge can still be semantically broken

---

### Visualise Git History

```bash
git log --oneline --graph --all --decorate

# Sample output:
* a1b2c3d (HEAD -> main) merge: resolve deployment strategy conflict
|\
| * d4e5f6g (conflict-feature-2) feat: update deployment strategy differently
* | e7f8g9h (conflict-feature-1) feat: update deployment strategy
|/
* i0j1k2l chore: add undo practice file
* m3n4o5p feat: initial project structure
```

---

## Assignment 4 – Git Undo & Recovery

---

### The Decision Framework: Reset vs Revert vs Stash vs Cherry-pick

```
Has the commit been pushed to remote (shared with others)?
    │
    ├── YES → Use GIT REVERT
    │         Creates a new commit that undoes the bad commit
    │         Safe for shared branches — history preserved
    │
    └── NO  → Use GIT RESET
              Rewrites local history
              Three modes: soft / mixed / hard
```

---

### `git revert` — Safe Undo for Pushed Commits

```bash
# Find the bad commit hash
git log --oneline -10

# Revert it (creates a NEW undo commit — history is preserved)
git revert a1b2c3d

# Revert without opening the editor (use default message)
git revert a1b2c3d --no-edit

# Revert a merge commit (must specify which parent to keep)
git revert -m 1 <merge-commit-hash>

# Push the revert
git push origin main
```

**Why `revert` for pushed commits**: In a team, other engineers may have already pulled the bad commit. A `reset` would rewrite history and force everyone to re-sync — dangerous and disruptive. `revert` adds a new commit that everyone can simply pull.

---

### `git reset` — Local History Rewriting

```bash
# SOFT: commit removed, changes stay STAGED
# Use when: you want to re-commit with a different message or combine with other changes
git reset --soft HEAD~1

# MIXED (default): commit removed, changes stay in WORKING DIRECTORY (unstaged)
# Use when: you want to review and selectively re-stage changes
git reset HEAD~1
git reset --mixed HEAD~1   # same

# HARD: commit removed AND working directory changes DELETED
# Use when: you want to completely discard the last commit and its changes
# WARNING: changes are gone — no recovery without reflog
git reset --hard HEAD~1

# Recovery using reflog (Git's safety net — keeps a log of where HEAD has been)
git reflog
# Find the commit hash before the reset
git reset --hard <hash-from-reflog>
```

---

### Detached HEAD: What It Is and How to Recover

```bash
# Detached HEAD occurs when you checkout a commit directly (not a branch)
git checkout a1b2c3d
# Output: HEAD is now at a1b2c3d (detached HEAD)

# In this state:
# - You CAN look at files, run the code, debug
# - You CAN commit, BUT those commits belong to no branch — they'll be garbage collected
# - You CANNOT push (no branch to push to)

# CORRECT RECOVERY: create a branch immediately
git checkout -b debug/investigate-v1.0.0
# Now your work is safe on a named branch

# If you made commits in detached HEAD accidentally:
git reflog            # find those commit hashes
git branch recovery-branch <commit-hash>   # save them to a branch
```

---

### `git stash` — Temporary Work Parking

```bash
# Save uncommitted work (staged + unstaged)
git stash push -m "wip: feature-auth half-done — switching to hotfix"

# Include untracked files
git stash push --include-untracked -m "wip: new files included"

# List all stashes
git stash list
# Output: stash@{0}: wip: feature-auth half-done...
#         stash@{1}: wip: another stash

# Apply and REMOVE from stash stack (most common)
git stash pop

# Apply but KEEP in stash stack (apply to multiple branches)
git stash apply stash@{0}

# Delete a specific stash
git stash drop stash@{0}

# Nuke all stashes
git stash clear
```

**Production scenario — stash workflow:**
```bash
# Mid-feature, urgent hotfix needed
git stash push -m "wip: booking-api pagination feature"
git switch -c hotfix/AC-9012-payment-timeout main
# ... fix, commit, PR ...
git switch feature/AC-8888-pagination
git stash pop
# Resume where you left off
```

---

### `git cherry-pick` — Surgical Commit Selection

```bash
# Apply one specific commit from another branch to current branch
git cherry-pick a1b2c3d

# Cherry-pick a range of commits
git cherry-pick a1b2c3d^..e4f5g6h    # from a1b (inclusive) to e4f

# Cherry-pick without committing (stage changes only — review before committing)
git cherry-pick --no-commit a1b2c3d

# If conflict during cherry-pick:
git add resolved-file.py
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

**Production use case:** A critical security fix is committed on `hotfix/v1.2.1` but `main` has diverged significantly. Rather than merging the entire hotfix branch, cherry-pick the single fix commit onto `main` — clean, targeted, minimal blast radius.

---

## Assignment 5 – Git Tags & Squash

---

### Git Tags: Release Management

```bash
# ANNOTATED TAG (recommended for releases — stores author, date, message)
git tag -a v1.0.0 -m "Release v1.0.0 — first production release of booking API"
git push origin v1.0.0

# LIGHTWEIGHT TAG (quick marker — just a pointer, no metadata)
git tag v1.0.0-rc1
git push origin v1.0.0-rc1

# List all tags
git tag
git tag -l "v1.*"     # filter pattern

# Inspect a tag
git show v1.0.0

# Tag an older commit (not HEAD)
git tag -a v0.9.0 a1b2c3d -m "Retroactive tag for pre-release build"

# Push all tags at once
git push origin --tags

# Delete tag
git tag -d v1.0.0-rc1              # local
git push origin --delete v1.0.0-rc1  # remote

# Semantic Versioning standard (MAJOR.MINOR.PATCH)
# v1.0.0 → v1.0.1 (patch: bug fix)
# v1.0.0 → v1.1.0 (minor: new feature, backward compatible)
# v1.0.0 → v2.0.0 (major: breaking change)
```

---

### Hotfix from a Tag (Production-Grade Flow)

```bash
# Production is running v1.2.0 and it's broken

# 1. Checkout the exact production state
git checkout v1.2.0
# HEAD is now detached at v1.2.0

# 2. Branch from the tag (NEVER commit in detached HEAD)
git checkout -b hotfix/v1.2.1

# 3. Apply the fix
vim src/payment/processor.py
git add src/payment/processor.py
git commit -m "fix(payment): resolve timeout causing transaction failures (AC-9012)"

# 4. Tag the hotfix
git tag -a v1.2.1 -m "Hotfix v1.2.1 — critical payment timeout fix"

# 5. Push branch and tag
git push -u origin hotfix/v1.2.1
git push origin v1.2.1

# 6. Merge hotfix back to main AND dev (keep branches in sync)
git checkout main
git merge hotfix/v1.2.1
git push origin main

git checkout dev
git merge hotfix/v1.2.1
git push origin dev
```

---

### Git Squash: Clean Commit History

```bash
# APPROACH 1: Interactive Rebase Squash (before PR merge, on feature branch)

# View messy commits
git log --oneline -5
# d1e2f3g wip: fix typo
# b4c5d6e wip: fix typo again
# a7b8c9d wip: add validation
# 9e8f7g6 feat: start validation logic   ← keep this base

# Squash last 4 commits into 1
git rebase -i HEAD~4

# In the editor:
# pick 9e8f7g6 feat: start validation logic
# squash a7b8c9d wip: add validation
# squash b4c5d6e wip: fix typo again
# squash d1e2f3g wip: fix typo

# Then write the final commit message:
# feat(validation): add input validation for booking form fields

# Push (force needed since history was rewritten)
git push --force-with-lease origin feature/validation

# APPROACH 2: Squash Merge (at PR merge time — GitHub/GitLab "Squash and Merge" button)
git checkout main
git merge --squash feature/validation
git commit -m "feat(validation): add input validation for booking form fields"
git push origin main
```

| Method | When to Use |
|---|---|
| Interactive rebase squash | On your own feature branch before opening a PR — you control the process |
| Squash merge | At merge time — the whole branch history collapses into one commit on main |
| `--force-with-lease` | Safer force push — fails if someone else pushed to the branch since your last fetch |

---

## Assignment 6 – Enterprise Branching Strategy

---

### Strategy 1: Environment-Based Branching (dev → qa → ppd → prod → dr)

This is the model used at large enterprises with strict change management (airlines, banks, government).

```
main (baseline)
  ├── dev   → continuous integration, developers merge here daily
  ├── qa    → QA validation, locked for direct commits
  ├── ppd   → pre-production (UAT, performance, security validation)
  ├── prod  → production (protected branch — merge only via approved PR)
  └── dr    → disaster recovery (mirror of prod, tested independently)
```

**CI/CD pipeline mapping:**
```yaml
# Each branch triggers its own pipeline
dev  → build + unit test + SAST → deploy to dev namespace
qa   → integration test + DAST  → deploy to QA namespace
ppd  → performance test + UAT   → deploy to pre-prod cluster
prod → smoke test + approval    → deploy to production
dr   → health check             → deploy to DR cluster
```

**Promotion flow:**
```bash
# Feature → dev
git checkout dev
git merge feature/AC-1234-loyalty-api
git push origin dev

# dev → qa (after dev testing complete)
git checkout qa
git merge dev
git push origin qa

# qa → ppd → prod (each step requires approvals)
git checkout ppd && git merge qa && git push origin ppd
git checkout prod && git merge ppd && git push origin prod

# After prod hotfix — sync BACKWARDS to all branches
git checkout ppd && git merge prod
git checkout qa  && git merge ppd
git checkout dev && git merge qa
git checkout dr  && git merge prod
```

---

### Strategy 2: GitFlow (Feature/Release/Hotfix Model)

Best for teams with scheduled release cycles (e.g., monthly releases).

```
main ──────────────────────────────────────────► (production releases, tagged)
  │                                  ▲
  └── develop ──────────────────────────────────► (integration branch)
       │           ▲        │         ▲
       ├── feature/A ──────►│         │
       ├── feature/B ──────►│         │
       │                             │
       └── release/1.2 ─── testing ──┘
              │
              └── hotfix/1.2.1 ──────────────────► (merge to main AND develop)
```

---

### Strategy 3: GitHub Flow (Simplified — Best for Continuous Delivery)

Best for teams deploying multiple times per day (SaaS, cloud-native).

```
main (always deployable)
  ├── feature/AC-1234-add-ssr
  ├── feature/AC-5678-fix-seat-map
  └── hotfix/AC-9012-payment-fix
```

- Every feature branch merges to `main` via Pull Request
- `main` deploys automatically to production on every merge
- No separate `develop` branch — lean and fast

**At Air Canada**, given the size and compliance requirements, a hybrid of environment-based branching + GitFlow is most likely — scheduled releases with protected `main` + environment promotion gates + hotfix process anchored to tags.

---

## Key Interview Questions: Git at Air Canada

---

### "Walk me through how you'd handle a critical bug in production."

> Production is running tag `v2.4.1`. A payment timeout bug is causing 503 errors for ~5% of bookings.
>
> 1. **Create hotfix branch from the production tag** — not from `main` (which may have unreleased features):
>    ```bash
>    git checkout v2.4.1
>    git checkout -b hotfix/AC-9012-payment-timeout
>    ```
> 2. **Apply the fix**, get it peer-reviewed via a PR — even under pressure, code review is non-negotiable.
> 3. **Tag the hotfix**: `git tag -a v2.4.1 -m "Hotfix: payment processor timeout"`
> 4. **Deploy via the hotfix pipeline** — which has expedited approval but still runs smoke tests.
> 5. **Merge back to `main` and `develop`** — never leave a hotfix stranded on its own branch.
> 6. **Post-incident review** — document what happened, why, and what process improvement prevents recurrence.

---

### "What's the difference between `git pull` and `git fetch`?"

> `git fetch` downloads changes from the remote into your remote-tracking branches (`origin/main`) but does NOT modify your local working branch. You can inspect what changed with `git log origin/main..main` before merging.
>
> `git pull` = `git fetch` + `git merge`. It fetches and immediately merges into your current branch.
>
> In production, I prefer `git fetch` followed by manual inspection before merging — especially on shared branches. `git pull --rebase` is also commonly used to keep history linear.

---

### "How do you protect the `main` branch in a team environment?"

> Branch protection rules in GitHub / Azure DevOps:
> - **Require pull request before merging** — no direct pushes to `main`
> - **Require N approvals** (typically 1–2 for feature PRs, 2+ for releases)
> - **Require status checks to pass** — CI pipeline (build, tests, SAST) must be green
> - **Require branches to be up to date** — prevents merging stale branches
> - **Restrict who can push** — only specific teams/roles
> - **Do not allow force pushes** — prevents history rewriting on main
> - **Require signed commits** — GPG signing for audit trails (important for compliance)
>
> In Terraform/IaC repos at Air Canada, I'd add a policy that infrastructure changes also require a Checkov/tfsec scan to pass before merge.

---

### "What is `git reflog` and when have you used it?"

> `git reflog` is Git's safety net. It records every position HEAD has been — even commits that no longer appear in `git log` because of a `reset --hard` or accidental branch deletion.
>
> I've used it in production when a junior engineer accidentally ran `git reset --hard HEAD~3` on a shared branch, losing three commits. With `git reflog` I found the SHA of the commit before the reset and recovered everything with `git reset --hard <sha>`. No data loss, five-minute recovery.
>
> `git reflog` entries expire after 90 days by default — so "accidentally deleted" doesn't mean "permanently deleted" as long as you act within that window.

---

## Summary: Git Commands Every Cloud Infrastructure Analyst Must Know

```bash
# Daily workflow
git status                          # Current state
git log --oneline --graph --all     # Visual history
git diff --staged                   # Review before committing
git add -p                          # Interactive staging (partial file staging)

# Branch management
git switch -c feature/<name>        # Create + switch
git push -u origin feature/<name>   # Push with tracking
git fetch --prune                   # Sync + remove deleted remote branches

# Undo / Recovery
git revert <hash>                   # Safe undo (pushed commits)
git reset --soft HEAD~1             # Undo last commit, keep staged
git reset --hard HEAD~1             # Undo last commit, discard changes
git stash push -m "description"     # Park work temporarily
git reflog                          # Find lost commits

# Release
git tag -a v1.0.0 -m "Release"     # Annotated tag
git push origin v1.0.0              # Push tag
git cherry-pick <hash>              # Port one commit to another branch

# Inspection / Debugging
git blame src/app.py                # Who changed each line and when
git bisect start                    # Binary search for regression-introducing commit
git log --author="Dhruwang"         # Filter by author
git log --since="2 weeks ago"       # Filter by time
git shortlog -sn                    # Contribution summary by author
```
