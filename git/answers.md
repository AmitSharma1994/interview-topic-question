# Git Interview Questions and Answers

## Git Basics

### What is Git?

**Answer:**
Git is a distributed version control system (DVCS) that tracks changes in source code during software development. It was created by Linus Torvalds in 2005 for Linux kernel development.

**Key characteristics:**
- **Distributed:** Every developer has a complete copy of the project history
- **Fast:** Most operations are local and don't require network access
- **Branching:** Lightweight branching and merging capabilities
- **Data integrity:** Uses SHA-1 hashing to ensure data integrity
- **Non-linear development:** Supports thousands of parallel branches

**Core concepts:**
- **Repository:** A collection of files and their complete history
- **Commit:** A snapshot of changes at a specific point in time
- **Branch:** A movable pointer to a specific commit
- **Working directory:** The current state of files you're working on
- **Staging area:** A file that stores information about what will go into your next commit

### How is Git different from other version control systems?

**Answer:**
Git differs from other version control systems in several key ways:

**Distributed vs Centralized:**
- **Git (Distributed):** Every clone is a full backup of all data
- **SVN/CVS (Centralized):** Single central server holds all versioned files

**Data storage approach:**
- **Git:** Stores data as snapshots of the entire project
- **Others:** Store data as changes to a base version of each file

**Branching model:**
- **Git:** Lightweight branches that are just pointers to commits
- **Others:** Branches often involve copying entire directories

**Performance:**
- **Git:** Most operations are local and very fast
- **Others:** Many operations require network communication

**Workflow flexibility:**
- **Git:** Supports multiple workflows (centralized, feature branch, gitflow, etc.)
- **Others:** Usually enforce a specific workflow

**Examples:**
```bash
# Git - distributed operations
git log                    # View history locally
git branch feature-x       # Create branch instantly
git commit -m "Changes"    # Commit locally

# SVN - centralized operations
svn log                    # Requires server connection
svn copy trunk branches/feature-x  # Server operation
svn commit -m "Changes"    # Requires server connection
```

### What is a repository in Git?

**Answer:**
A Git repository is a data structure that stores metadata for a set of files and directories. It contains the complete history of changes, branches, tags, and configuration information.

**Repository components:**
- **.git directory:** Contains all Git metadata and object database
- **Working directory:** Current checkout of files from the repository
- **Staging area (Index):** Prepared changes for the next commit

**Types of repositories:**
1. **Local repository:** On your local machine
2. **Remote repository:** On a server (GitHub, GitLab, etc.)
3. **Bare repository:** Contains only Git data, no working directory

**Repository structure:**
```
project/
├── .git/                 # Git metadata directory
│   ├── objects/          # Git object database
│   ├── refs/             # References (branches, tags)
│   ├── HEAD              # Current branch pointer
│   ├── config            # Repository configuration
│   └── hooks/            # Git hooks
├── src/                  # Project files
├── README.md
└── .gitignore
```

**Creating repositories:**
```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone https://github.com/user/repo.git

# Check repository status
git status
```

## Branching and Merging

### What is the difference between git merge and git rebase?

**Answer:**
Both `git merge` and `git rebase` integrate changes from one branch into another, but they do so in fundamentally different ways.

**Git Merge:**
- Creates a new merge commit that combines the histories of both branches
- Preserves the complete history and context of both branches
- Non-destructive operation - existing branches are not changed
- Results in a non-linear history with merge commits

**Git Rebase:**
- Moves or combines commits from one branch to another
- Creates a linear history by replaying commits
- Rewrites commit history - changes commit SHAs
- Results in a clean, linear project history

**Visual comparison:**
```
Before merge/rebase:
    A---B---C feature
   /
  D---E---F---G main

After merge:
    A---B---C feature
   /         \
  D---E---F---G---H main
              (H is merge commit)

After rebase:
  D---E---F---G---A'---B'---C' main
                  (A', B', C' are new commits)
```

**Commands:**
```bash
# Merge approach
git checkout main
git merge feature-branch

# Rebase approach
git checkout feature-branch
git rebase main
git checkout main
git merge feature-branch  # Fast-forward merge
```

**When to use each:**
- **Use merge when:** You want to preserve branch history, working on public branches, or collaborating with others
- **Use rebase when:** You want a clean linear history, working on private branches, or preparing commits for review

### What happens when you do a commit?

**Answer:**
When you execute `git commit`, Git performs several operations to create a permanent snapshot of your changes:

**Step-by-step process:**

1. **Creates objects:**
   - **Blob objects:** For each modified file's content
   - **Tree object:** Represents the directory structure
   - **Commit object:** Contains metadata and points to the tree

2. **Generates SHA-1 hashes:**
   - Each object gets a unique 40-character SHA-1 hash
   - Hash is based on the object's content

3. **Updates references:**
   - Moves the current branch pointer to the new commit
   - Updates HEAD to point to the new commit

4. **Clears staging area:**
   - Staged changes are now part of the commit
   - Staging area is reset for the next commit

**Git objects created:**
```bash
# Example commit structure
commit abc123def456...
├── tree def456abc789...     # Root directory snapshot
├── parent 789abc123def...   # Previous commit (if not first)
├── author John Doe <john@example.com>
├── committer John Doe <john@example.com>
├── date 2023-12-01 10:30:00
└── message "Add user authentication feature"
```

**Commands and workflow:**
```bash
# Stage changes
git add file1.txt file2.txt

# Create commit
git commit -m "Add new features"

# What Git does internally:
# 1. Creates blob objects for file1.txt and file2.txt
# 2. Creates tree object representing current directory state
# 3. Creates commit object with message and metadata
# 4. Updates branch pointer and HEAD
```

### What are the best branching strategies?

**Answer:**
Different branching strategies suit different team sizes, release cycles, and project requirements:

**1. Git Flow:**
- **Main branches:** `main` (production) and `develop` (integration)
- **Supporting branches:** `feature/*`, `release/*`, `hotfix/*`
- **Best for:** Projects with scheduled releases and multiple environments

```bash
# Git Flow example
git checkout develop
git checkout -b feature/user-login
# Develop feature
git checkout develop
git merge feature/user-login
git branch -d feature/user-login

# Release preparation
git checkout -b release/1.2.0 develop
# Bug fixes and version bumps
git checkout main
git merge release/1.2.0
git tag v1.2.0
git checkout develop
git merge release/1.2.0
```

**2. GitHub Flow:**
- **Single main branch:** `main` is always deployable
- **Feature branches:** Short-lived branches for each feature
- **Best for:** Continuous deployment and small teams

```bash
# GitHub Flow example
git checkout main
git pull origin main
git checkout -b feature/add-search
# Develop and test feature
git push origin feature/add-search
# Create pull request, review, and merge
```

**3. GitLab Flow:**
- **Environment branches:** `main`, `pre-production`, `production`
- **Feature branches:** Merged to main, then promoted through environments
- **Best for:** Multiple deployment environments

**4. Trunk-based Development:**
- **Single trunk:** All developers work on `main` branch
- **Short-lived branches:** Very short feature branches (hours/days)
- **Best for:** High-frequency releases and experienced teams

### How to switch from one branch to another branch?

**Answer:**
Git provides several commands to switch between branches:

**Basic branch switching:**
```bash
# Switch to existing branch
git checkout branch-name

# Switch to existing branch (Git 2.23+)
git switch branch-name

# Create and switch to new branch
git checkout -b new-branch-name

# Create and switch to new branch (Git 2.23+)
git switch -c new-branch-name
```

**Advanced switching scenarios:**
```bash
# Switch to previous branch
git checkout -
git switch -

# Switch to remote branch
git checkout -b local-branch origin/remote-branch
git switch -c local-branch origin/remote-branch

# Switch with uncommitted changes (stash first)
git stash
git checkout other-branch
git stash pop

# Force switch (discards local changes)
git checkout --force branch-name
```

**Checking branch status:**
```bash
# List all branches
git branch -a

# Show current branch
git branch --show-current

# Show branch with last commit
git branch -v
```

### What is a conflict in Git and how to resolve it?

**Answer:**
A Git conflict occurs when Git cannot automatically merge changes from different branches because the same lines of code have been modified in different ways.

**When conflicts occur:**
- Merging branches with overlapping changes
- Rebasing with conflicting modifications
- Cherry-picking commits with conflicts
- Pulling changes that conflict with local modifications

**Conflict resolution process:**

1. **Identify conflicts:**
```bash
git merge feature-branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.

git status
# On branch main
# You have unmerged paths.
# Unmerged paths:
#   both modified:   file.txt
```

2. **Examine conflict markers:**
```bash
# file.txt content
<<<<<<< HEAD
This is the main branch version
=======
This is the feature branch version
>>>>>>> feature-branch
```

3. **Resolve conflicts manually:**
```bash
# Edit file.txt to resolve conflicts
This is the resolved version combining both changes

# Or use merge tools
git mergetool
```

4. **Complete the merge:**
```bash
# Stage resolved files
git add file.txt

# Complete the merge
git commit -m "Resolved merge conflicts between main and feature-branch"
```

**Conflict resolution strategies:**
```bash
# Accept current branch changes
git checkout --ours file.txt

# Accept incoming branch changes
git checkout --theirs file.txt

# Abort merge and return to previous state
git merge --abort

# Use external merge tool
git config merge.tool vimdiff
git mergetool
```

## Git Commands

### What is git stash and how does it allow changing branches without committing changes?

**Answer:**
`git stash` temporarily saves uncommitted changes (both staged and unstaged) so you can work on something else, then come back and re-apply them later.

**How git stash works:**
- Saves current working directory and staging area state
- Reverts working directory to match HEAD commit
- Allows clean branch switching
- Changes can be reapplied later

**Basic stash operations:**
```bash
# Stash current changes
git stash
git stash save "Work in progress on feature X"

# List all stashes
git stash list
# stash@{0}: WIP on main: abc123 Last commit message
# stash@{1}: On feature: def456 Work in progress on feature X

# Apply most recent stash
git stash pop          # Applies and removes from stash list
git stash apply        # Applies but keeps in stash list

# Apply specific stash
git stash apply stash@{1}

# Drop a stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

**Advanced stash usage:**
```bash
# Stash only unstaged changes
git stash --keep-index

# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a

# Create branch from stash
git stash branch new-branch-name stash@{0}

# Show stash contents
git stash show -p stash@{0}
```

**Typical workflow:**
```bash
# Working on feature branch
git checkout feature-branch
# Make some changes but not ready to commit

# Need to switch to main for urgent fix
git stash save "Incomplete feature work"
git checkout main
# Fix urgent issue and commit

# Return to feature work
git checkout feature-branch
git stash pop
# Continue working on feature
```

### What is the difference between git pull and git fetch?

**Answer:**
Both commands retrieve changes from a remote repository, but they handle the integration differently:

**Git Fetch:**
- Downloads commits, files, and refs from remote repository
- Updates remote-tracking branches (e.g., `origin/main`)
- Does NOT modify your working directory or current branch
- Safe operation that never changes your local work

**Git Pull:**
- Performs `git fetch` followed by `git merge`
- Downloads changes AND integrates them into current branch
- Can modify your working directory and create merge commits
- May cause conflicts that need resolution

**Visual representation:**
```
Before fetch/pull:
Local:  A---B---C main
Remote: A---B---C---D---E origin/main

After git fetch:
Local:  A---B---C main
        A---B---C---D---E origin/main (updated)

After git pull:
Local:  A---B---C---F main (F is merge commit)
           \     /
            D---E origin/main
```

**Commands and usage:**
```bash
# Fetch changes without merging
git fetch origin
git fetch origin main

# See what was fetched
git log HEAD..origin/main
git diff HEAD origin/main

# Pull changes (fetch + merge)
git pull origin main
git pull  # Uses default remote and branch

# Pull with rebase instead of merge
git pull --rebase origin main
```

**When to use each:**
- **Use fetch when:** You want to see changes before integrating, working on sensitive code, or reviewing changes first
- **Use pull when:** You want to quickly update your branch with remote changes and trust the automatic merge

**Best practices:**
```bash
# Safe workflow with fetch
git fetch origin
git log --oneline HEAD..origin/main  # Review incoming changes
git merge origin/main                 # Merge when ready

# Configure pull to use rebase by default
git config pull.rebase true
```

### How to remove files from the local repository?

**Answer:**
Git provides several ways to remove files depending on your specific needs:

**1. Remove files from working directory and staging area:**
```bash
# Remove file completely
git rm filename.txt
git commit -m "Remove filename.txt"

# Remove multiple files
git rm file1.txt file2.txt
git rm *.log

# Remove directory and all contents
git rm -r directory-name/
```

**2. Remove files only from Git tracking (keep local file):**
```bash
# Stop tracking file but keep it locally
git rm --cached filename.txt
git commit -m "Stop tracking filename.txt"

# Stop tracking directory but keep it locally
git rm -r --cached directory-name/

# Useful for files that should be ignored
echo "filename.txt" >> .gitignore
git add .gitignore
git commit -m "Add filename.txt to gitignore"
```

**3. Remove files that have been deleted outside Git:**
```bash
# Stage all deletions
git add -u

# Or use git add with --all flag
git add --all
git commit -m "Remove deleted files"
```

**4. Undo file removal (before commit):**
```bash
# Restore removed file from staging area
git restore --staged filename.txt
git restore filename.txt

# Or using older syntax
git reset HEAD filename.txt
git checkout filename.txt
```

**5. Remove files from history (dangerous operation):**
```bash
# Remove file from entire Git history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch filename.txt' \
  --prune-empty --tag-name-filter cat -- --all

# Modern approach using git filter-repo
git filter-repo --path filename.txt --invert-paths
```

**Common scenarios:**
```bash
# Remove sensitive file that was accidentally committed
git rm --cached secrets.txt
echo "secrets.txt" >> .gitignore
git add .gitignore
git commit -m "Remove secrets.txt and add to gitignore"

# Clean up build artifacts
git rm -r --cached build/
echo "build/" >> .gitignore
git add .gitignore
git commit -m "Stop tracking build directory"
```

### What is the cherry-pick command and when would you use it?

**Answer:**
`git cherry-pick` applies the changes from specific commits to your current branch, creating new commits with the same changes but different commit hashes.

**How cherry-pick works:**
- Takes a commit from any branch
- Applies its changes to the current branch
- Creates a new commit with the same changes
- Original commit remains unchanged

**Basic cherry-pick usage:**
```bash
# Cherry-pick a single commit
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick a range of commits
git cherry-pick abc123..def456

# Cherry-pick without committing (stage changes only)
git cherry-pick --no-commit abc123
```

**When to use cherry-pick:**

1. **Hotfixes:** Apply critical fixes to multiple branches
```bash
# Fix applied to main branch
git checkout main
git commit -m "Fix critical security issue" # Creates commit abc123

# Apply same fix to release branch
git checkout release-1.0
git cherry-pick abc123
```

2. **Selective feature porting:** Move specific features between branches
```bash
# Feature branch has multiple commits, only want one
git checkout main
git cherry-pick feature-branch~2  # Pick specific commit from feature branch
```

3. **Bug fixes:** Apply fixes from development to production branches
```bash
# Bug fix in develop branch
git checkout develop
git commit -m "Fix user login issue" # Creates commit def456

# Apply to production branch
git checkout production
git cherry-pick def456
```

**Handling cherry-pick conflicts:**
```bash
# If conflicts occur during cherry-pick
git cherry-pick abc123
# CONFLICT: Merge conflict in file.txt

# Resolve conflicts manually
# Edit conflicted files
git add file.txt

# Continue cherry-pick
git cherry-pick --continue

# Or abort cherry-pick
git cherry-pick --abort
```

**Advanced cherry-pick options:**
```bash
# Cherry-pick and edit commit message
git cherry-pick -e abc123

# Cherry-pick only the changes, don't commit
git cherry-pick -n abc123

# Cherry-pick and sign off
git cherry-pick -s abc123

# Cherry-pick from another repository
git remote add other-repo https://github.com/user/other-repo.git
git fetch other-repo
git cherry-pick other-repo/main~3
```

**Best practices:**
- Use cherry-pick sparingly to avoid duplicate commits
- Prefer merging or rebasing for regular integration
- Document why commits were cherry-picked
- Be careful with cherry-picking merge commits

## Advanced Git

### What are Git hooks?

**Answer:**
Git hooks are scripts that Git executes automatically at certain points in the Git workflow. They allow you to customize Git's behavior and automate tasks.

**Types of Git hooks:**

**Client-side hooks (local repository):**
- `pre-commit`: Runs before commit is created
- `prepare-commit-msg`: Runs before commit message editor opens
- `commit-msg`: Validates commit messages
- `post-commit`: Runs after commit is completed
- `pre-push`: Runs before push to remote
- `pre-rebase`: Runs before rebase operation

**Server-side hooks (remote repository):**
- `pre-receive`: Runs before any references are updated
- `update`: Runs for each branch being updated
- `post-receive`: Runs after all references are updated
- `post-update`: Runs after post-receive

**Hook location and setup:**
```bash
# Hooks are stored in .git/hooks/ directory
ls .git/hooks/
# applypatch-msg.sample    pre-commit.sample
# commit-msg.sample        pre-push.sample
# post-update.sample       prepare-commit-msg.sample

# Make hook executable
chmod +x .git/hooks/pre-commit
```

**Example hooks:**

**1. Pre-commit hook (code quality checks):**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run tests before commit
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi

# Check code formatting
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit aborted."
    exit 1
fi

echo "Pre-commit checks passed."
exit 0
```

**2. Commit-msg hook (message validation):**
```bash
#!/bin/sh
# .git/hooks/commit-msg

commit_regex='^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "Invalid commit message format!"
    echo "Format: type(scope): description"
    echo "Example: feat(auth): add user login functionality"
    exit 1
fi
```

**3. Pre-push hook (prevent pushing to main):**
```bash
#!/bin/sh
# .git/hooks/pre-push

protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ $protected_branch = $current_branch ]; then
    echo "Direct push to main branch is not allowed!"
    echo "Please create a feature branch and submit a pull request."
    exit 1
fi
```

**Managing hooks in teams:**
```bash
# Store hooks in version control
mkdir scripts/git-hooks
cp .git/hooks/pre-commit scripts/git-hooks/

# Setup script for team members
#!/bin/sh
# setup-hooks.sh
ln -sf ../../scripts/git-hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

### How to use Git for large binary files?

**Answer:**
Git is optimized for text files and can struggle with large binary files. Here are strategies to handle them:

**Problems with large binary files in Git:**
- Repository size grows quickly
- Cloning becomes slow
- Binary files can't be merged meaningfully
- History becomes bloated

**Solutions:**

**1. Git LFS (Large File Storage):**
```bash
# Install Git LFS
git lfs install

# Track large file types
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"

# Add .gitattributes file
git add .gitattributes

# Add and commit large files normally
git add large-file.psd
git commit -m "Add design file"

# Push with LFS
git push origin main
```

**2. Exclude binary files with .gitignore:**
```bash
# .gitignore
*.exe
*.dll
*.so
*.dylib
build/
dist/
node_modules/
*.log
```

**3. Use external storage:**
```bash
# Store references to external files
echo "s3://bucket/large-file.zip" > large-file.txt
git add large-file.txt
git commit -m "Add reference to large file"
```

**4. Git attributes for binary files:**
```bash
# .gitattributes
*.png binary
*.jpg binary
*.pdf binary
*.docx binary
```

### What is Git LFS?

**Answer:**
Git Large File Storage (LFS) is an extension that replaces large files with text pointers inside Git, while storing the file contents on a remote server.

**How Git LFS works:**
1. Large files are stored on LFS server
2. Git repository contains only small pointer files
3. Pointer files contain metadata about actual files
4. Files are downloaded on-demand during checkout

**Setting up Git LFS:**
```bash
# Install Git LFS (one-time setup)
git lfs install

# Track file types
git lfs track "*.psd"
git lfs track "*.ai"
git lfs track "*.zip"
git lfs track "assets/**"

# View tracked patterns
git lfs track

# Add .gitattributes to repository
git add .gitattributes
git commit -m "Configure Git LFS tracking"
```

**Working with LFS files:**
```bash
# Add large files normally
git add large-design.psd
git commit -m "Add design file"

# Push to LFS server
git push origin main

# Clone repository with LFS files
git clone https://github.com/user/repo.git
cd repo
git lfs pull  # Download LFS files
```

**LFS commands:**
```bash
# Show LFS files in repository
git lfs ls-files

# Show LFS storage usage
git lfs env

# Fetch LFS files
git lfs fetch
git lfs pull

# Push LFS files
git lfs push origin main

# Migrate existing files to LFS
git lfs migrate import --include="*.zip"
```

**LFS pointer file example:**
```
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a214614ab2935c943f9e0ff69d22eadbb8f32b1258daaa5e2ca24d17e2393
size 12345
```

### How to handle merge conflicts in complex scenarios?

**Answer:**
Complex merge conflicts require systematic approaches and specialized tools:

**Types of complex conflicts:**

**1. Three-way conflicts (multiple branches):**
```bash
# Merge multiple branches
git checkout main
git merge branch-a branch-b

# Resolve conflicts considering all three sources
# Use merge tools that show base, local, and remote versions
git mergetool --tool=vimdiff
```

**2. Rename conflicts:**
```bash
# File renamed differently in each branch
# Git shows both versions
git status
# renamed: old-name.txt -> new-name-a.txt
# renamed: old-name.txt -> new-name-b.txt

# Choose one name and stage
git add new-name-a.txt
git rm new-name-b.txt
```

**3. Binary file conflicts:**
```bash
# Binary files can't be merged automatically
git status
# both modified: image.png

# Choose one version
git checkout --ours image.png    # Keep current branch version
git checkout --theirs image.png  # Keep incoming branch version
git add image.png
```

**Advanced conflict resolution strategies:**

**1. Using merge strategies:**
```bash
# Recursive strategy with options
git merge -X ours branch-name      # Favor current branch
git merge -X theirs branch-name    # Favor incoming branch
git merge -X ignore-space-change branch-name

# Octopus strategy for multiple branches
git merge branch-a branch-b branch-c
```

**2. Manual conflict resolution workflow:**
```bash
# 1. Identify all conflicted files
git status | grep "both modified"

# 2. For each file, examine conflict
git show :1:filename  # Common ancestor
git show :2:filename  # Current branch (ours)
git show :3:filename  # Incoming branch (theirs)

# 3. Edit file to resolve conflicts
# Remove conflict markers and choose/combine changes

# 4. Stage resolved file
git add filename

# 5. Continue merge
git commit -m "Resolved complex merge conflicts"
```

**3. Using external merge tools:**
```bash
# Configure merge tool
git config merge.tool kdiff3
git config merge.tool meld
git config merge.tool vscode

# Use merge tool for conflicts
git mergetool

# Custom merge tool configuration
git config mergetool.vscode.cmd 'code --wait $MERGED'
git config mergetool.vscode.trustExitCode false
```

**4. Conflict resolution for specific scenarios:**
```bash
# Resolve conflicts in favor of one side
git checkout --ours .     # Keep all current branch changes
git checkout --theirs .   # Keep all incoming changes

# Abort merge and try different approach
git merge --abort
git rebase branch-name    # Try rebase instead of merge

# Create custom merge commit message
git commit -m "Merge branch 'feature' resolving conflicts in:
- file1.txt: kept current implementation
- file2.txt: combined both approaches
- file3.txt: used incoming version"
```

## Best Practices

### What are the best practices for commit messages?

**Answer:**
Good commit messages are crucial for project maintainability and team collaboration. Here are established best practices:

**Conventional Commit Format:**
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Commit message structure:**

**1. Subject line (first line):**
- Keep under 50 characters
- Use imperative mood ("Add feature" not "Added feature")
- Capitalize first letter
- No period at the end
- Be specific and descriptive

**2. Body (optional):**
- Separate from subject with blank line
- Wrap at 72 characters
- Explain what and why, not how
- Use bullet points if needed

**3. Footer (optional):**
- Reference issues, breaking changes
- Include co-authors if applicable

**Commit types:**
```bash
feat: add user authentication system
fix: resolve memory leak in image processing
docs: update API documentation
style: fix code formatting issues
refactor: restructure user service
test: add unit tests for payment module
chore: update dependencies
perf: optimize database queries
ci: update GitHub Actions workflow
```

**Good commit message examples:**
```bash
# Good - clear and specific
git commit -m "feat(auth): add JWT token validation middleware"

# Good - explains the why
git commit -m "fix: handle null pointer in user profile

The user profile component was crashing when accessing
users without profile pictures. Added null checks and
default avatar fallback.

Fixes #123"

# Good - breaking change
git commit -m "feat!: change API response format

BREAKING CHANGE: API responses now use camelCase instead
of snake_case for consistency with frontend conventions.

Migration guide: https://docs.example.com/migration"
```

**Bad commit message examples:**
```bash
# Bad - too vague
git commit -m "fix stuff"
git commit -m "update"
git commit -m "changes"

# Bad - too long subject
git commit -m "fix the bug where users couldn't login because of validation"

# Bad - not imperative mood
git commit -m "fixed login bug"
git commit -m "updating documentation"
```

**Commit message templates:**
```bash
# Set up commit message template
git config commit.template ~/.gitmessage

# ~/.gitmessage content:
# <type>(<scope>): <subject>
#
# <body>
#
# <footer>
```

### How to keep a clean Git history?

**Answer:**
A clean Git history improves code review, debugging, and project understanding. Here are strategies to maintain it:

**1. Use interactive rebase to clean up commits:**
```bash
# Rebase last 3 commits interactively
git rebase -i HEAD~3

# Interactive rebase options:
# pick = use commit as-is
# reword = change commit message
# edit = modify commit
# squash = combine with previous commit
# fixup = like squash but discard message
# drop = remove commit
```

**2. Squash related commits:**
```bash
# Before squashing (messy history):
abc123 fix typo in readme
def456 add user validation
ghi789 fix validation bug
jkl012 update validation tests

# After squashing (clean history):
xyz789 feat: add user validation with tests
```

**3. Use meaningful branch names:**
```bash
# Good branch names
feature/user-authentication
bugfix/memory-leak-fix
hotfix/security-patch
refactor/database-layer

# Bad branch names
my-branch
temp
fix
test-branch
```

**4. Atomic commits (one logical change per commit):**
```bash
# Good - separate concerns
git add user-model.js
git commit -m "feat: add User model with validation"

git add user-controller.js
git commit -m "feat: add User controller with CRUD operations"

# Bad - mixing concerns
git add user-model.js user-controller.js config.js
git commit -m "add user stuff and fix config"
```

**5. Use feature branches and pull requests:**
```bash
# Feature branch workflow
git checkout -b feature/payment-integration
# Make commits on feature branch
git push origin feature/payment-integration
# Create pull request for review
# Merge with squash or rebase
```

**6. Avoid merge commits in linear history:**
```bash
# Use rebase instead of merge for feature branches
git checkout feature-branch
git rebase main
git checkout main
git merge feature-branch  # Fast-forward merge

# Or use squash merge
git merge --squash feature-branch
git commit -m "feat: add payment integration"
```

**7. Clean up before pushing:**
```bash
# Review commits before pushing
git log --oneline origin/main..HEAD

# Rebase to clean up if needed
git rebase -i origin/main

# Force push after rebase (be careful!)
git push --force-with-lease origin feature-branch
```

### How to organize repositories for large projects?

**Answer:**
Large projects require thoughtful repository organization to maintain productivity and code quality:

**Repository organization strategies:**

**1. Monorepo (Single repository):**
```
project/
├── services/
│   ├── user-service/
│   ├── payment-service/
│   └── notification-service/
├── shared/
│   ├── common-utils/
│   └── shared-types/
├── frontend/
│   ├── web-app/
│   └── mobile-app/
├── docs/
├── scripts/
└── tools/
```

**Advantages:**
- Unified versioning and releases
- Easy code sharing and refactoring
- Simplified dependency management
- Atomic commits across services

**Disadvantages:**
- Large repository size
- Slower clone times
- Complex CI/CD pipelines

**2. Multi-repo (Multiple repositories):**
```
organization/
├── user-service/
├── payment-service/
├── notification-service/
├── shared-utils/
├── web-frontend/
├── mobile-app/
└── documentation/
```

**Advantages:**
- Independent development and deployment
- Smaller repository sizes
- Team ownership boundaries
- Technology diversity

**Disadvantages:**
- Complex dependency management
- Difficult cross-service refactoring
- Version synchronization challenges

**3. Hybrid approach:**
```
# Core services in monorepo
core-services/
├── user-service/
├── payment-service/
└── shared/

# Independent applications
web-frontend/
mobile-app/
admin-dashboard/
```

**Repository structure best practices:**

**1. Consistent directory structure:**
```
service-name/
├── src/
│   ├── main/
│   └── test/
├── docs/
├── scripts/
├── Dockerfile
├── README.md
├── .gitignore
└── package.json
```

**2. Documentation organization:**
```
docs/
├── README.md
├── CONTRIBUTING.md
├── CHANGELOG.md
├── api/
├── architecture/
├── deployment/
└── troubleshooting/
```

**3. Configuration management:**
```
config/
├── development/
├── staging/
├── production/
└── shared/
```

**Git workflow for large projects:**

**1. Branch protection rules:**
```bash
# Protect main branch
# Require pull request reviews
# Require status checks
# Require up-to-date branches
# Restrict pushes to main
```

**2. Automated workflows:**
```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: npm test
      - name: Run linting
        run: npm run lint
```

**3. Code ownership:**
```bash
# CODEOWNERS file
# Global owners
* @team-leads

# Service-specific owners
/services/user-service/ @user-team
/services/payment-service/ @payment-team
/frontend/ @frontend-team

# Documentation
/docs/ @tech-writers
```

**4. Release management:**
```bash
# Semantic versioning
git tag v1.2.3
git push origin v1.2.3

# Release branches
git checkout -b release/1.2.0
# Prepare release
git checkout main
git merge release/1.2.0
git tag v1.2.0
```

**Tools for large projects:**
- **Git submodules:** For shared dependencies
- **Git subtree:** For including external projects
- **Git LFS:** For large binary files
- **Conventional commits:** For automated changelog generation
- **Semantic release:** For automated versioning