# Git Interview Questions

## Git Basics

- What is Git?
- How is Git different from other version control systems?
- What is a repository in Git?

## Branching and Merging

- What is the difference between git merge and git rebase?
- What happens when you do a commit?
- What are the best branching strategies?
- How to switch from one branch to another branch?
- What is a conflict in Git and how to resolve it?

## Git Commands

- What is git stash and how does it allow changing branches without committing changes?
- What is the difference between git pull and git fetch?
- How to remove files from the local repository?
- What is the cherry-pick command and when would you use it?

## Advanced Git

- What are Git hooks?
- How to use Git for large binary files?
- What is Git LFS?
- How to handle merge conflicts in complex scenarios?

## Best Practices

- What are the best practices for commit messages?
- How to keep a clean Git history?
- How to organize repositories for large projects?

## Code Examples

```bash
# Example: Common Git commands

# Create a new branch and switch to it
git checkout -b feature/new-feature

# Stash changes to work on another branch
git stash save "Work in progress on feature X"
git checkout another-branch
# Do some work on another-branch
git checkout feature/new-feature
git stash pop

# Merge vs Rebase
# Merge (preserves history)
git checkout main
git merge feature/new-feature

# Rebase (linear history)
git checkout feature/new-feature
git rebase main

# Cherry-pick a specific commit
git cherry-pick abc123

# Resolve conflicts
# After conflict is detected
git status  # See conflicted files
# Edit files to resolve conflicts
git add <resolved-files>
git commit -m "Resolved merge conflicts"
```

## Resources

- [Git Resources](https://docs.google.com/document/d/1y-AQQqu0kka6ROs9h7_IUNSo1490s_JQewp0_44t13s/edit)