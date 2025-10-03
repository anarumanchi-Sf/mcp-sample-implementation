## Git Workflow

Repo: https://github.com/anarumanchi-Sf/mcp-sample-implementation.git

---

### 1) How to clone this repo over HTTPS and SSH

#### HTTPS
```bash
# Clone the repo over HTTPS into a new folder
git clone https://github.com/anarumanchi-Sf/mcp-sample-implementation.git

# Change directory into the cloned repo
cd mcp-sample-implementation

# Show configured remotes for verification
git remote -v
```

#### SSH (requires SSH key configured on GitHub)
```bash
# Verify SSH connectivity to GitHub (should print an auth success message)
ssh -T git@github.com

# Clone the repo over SSH into a new folder
git clone git@github.com:anarumanchi-Sf/mcp-sample-implementation.git

# Change directory into the cloned repo
cd mcp-sample-implementation

# Show configured remotes for verification
git remote -v
```

---

### 2) How to create a new branch for yourself
```bash
# Make sure your local main is up to date
git fetch origin                            # download latest refs
git switch main                             # switch to local main
git pull --ff-only                          # fast-forward to origin/main

# Create a new branch from main and switch to it
git switch -c feature/<short-description>   # e.g., feature/add-order-tool

# (optional) Push the new branch and set upstream now
git push -u origin feature/<short-description>
```

---

### 3) Once changes are made, how to see the files changed and commit them to a new branch
```bash
# See what changed in your working tree
git status                                   # modified/untracked files

# Review unstaged diffs line-by-line
git diff                                     # changes not yet staged

# Review diffs for staged files
git diff --staged                            # changes already staged

# Stage changes (pick one pattern)
git add .                                    # stage everything
# or
git add path/to/file1 path/to/file2          # stage specific files
# or interactively pick hunks
git add -p                                   # stage selected hunks

# Commit staged changes with a clear message
git commit -m "feat(mcp): add <what-you-added>"

# Push your branch to GitHub (sets upstream if not already set)
git push -u origin feature/<short-description>
```

---

### 4) If you want to merge anything into the main branch, how to do it

Preferred: Create a Pull Request (PR) on GitHub
```bash
# On GitHub UI:
# - Click “Compare & pull request” for your branch
# - Base: main, Compare: your feature branch
# - Describe what/why/how to test, request reviewers, submit PR
# - After approvals and passing checks, click “Merge pull request”
```

Alternative: Merge locally (when PRs aren’t required)
```bash
# Update local main
git fetch origin                             # get latest remote
git switch main                              # go to main locally
git pull --ff-only                           # fast-forward main

# Merge your feature branch into main
git merge feature/<short-description>        # creates merge commit if needed

# Push updated main to origin
git push origin main
```

Keep your feature branch up to date before merging (recommended)
```bash
# Rebase your feature branch on top of latest main for a clean history
git fetch origin
git switch feature/<short-description>
git rebase origin/main                       # resolve conflicts if any
git rebase --continue                        # continue after resolving

# If you rewrote history and already pushed, update remote safely
git push --force-with-lease
```

