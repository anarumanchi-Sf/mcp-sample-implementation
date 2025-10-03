## Git Workflow: Clone, Branch & Commit, Merge into Main

Repo: https://github.com/anarumanchi-Sf/mcp-sample-implementation.git

---

### 1) Clone this repository

#### Option A: HTTPS
```bash
# Clone the repo over HTTPS into a new folder
git clone https://github.com/anarumanchi-Sf/mcp-sample-implementation.git

# Change directory into the cloned repo
cd mcp-sample-implementation

# Show configured remotes for verification
git remote -v
```

#### Option B: SSH (requires SSH key configured on GitHub)
```bash
# Verify SSH connectivity to GitHub
ssh -T git@github.com

# Clone the repo over SSH into a new folder
git clone git@github.com:anarumanchi-Sf/mcp-sample-implementation.git

# Change directory into the cloned repo
cd mcp-sample-implementation

# Show configured remotes for verification
git remote -v
```

---

### 2) Make changes, see what changed, and commit them to a new branch
```bash
# Download the latest remote refs without altering local files
git fetch origin

# Switch to the local main branch
git switch main

# Fast-forward local main to the latest origin/main
git pull --ff-only

# Create a new branch from main and switch to it
git switch -c feature/<short-description>

# Show the working tree status (what’s modified/untracked)
git status

# Show unstaged diffs line-by-line
git diff

# Show diffs of currently staged files (if any)
git diff --staged

# Stage all changes recursively (or specify paths instead)
git add .

# Interactively pick and stage specific hunks
git add -p

# Commit staged changes with a clear message
git commit -m "feat(mcp): add <what-you-added>"

# Push the new branch to origin and set upstream tracking
git push -u origin feature/<short-description>
```

---

### 3) Merge your changes into main

Preferred: Create a Pull Request (PR) on GitHub
```bash
# On GitHub UI:
# - Click “Compare & pull request” for your branch
# - Base: main, Compare: your feature branch
# - Add description (what/why/how to test), request reviewers, submit PR
# - After approvals and passing checks, click “Merge pull request”
```

Alternative: Merge locally (when PRs aren’t required)
```bash
# Fetch latest changes from the remote
git fetch origin

# Switch to local main branch
git switch main

# Fast-forward local main to latest origin/main
git pull --ff-only

# Merge the feature branch into main (creates merge commit if needed)
git merge feature/<short-description>

# Push updated main to origin
git push origin main
```

Keep your feature branch up to date before merging (recommended)
```bash
# Fetch latest state of main from origin
git fetch origin

# Switch to your feature branch
git switch feature/<short-description>

# Replay your branch commits on top of latest main for a clean history
git rebase origin/main

# Continue rebase after resolving any conflicts
git rebase --continue

# Update the remote branch safely after rewriting history
git push --force-with-lease
```


