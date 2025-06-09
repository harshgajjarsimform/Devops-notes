# Git and GitHub: A Comprehensive Guide

*Last updated: June 9, 2025*

## Table of Contents
- [Introduction](#introduction)
- [Setting Up Git](#setting-up-git)
  - [Installation](#installation)
  - [Configuration](#configuration)
  - [Authentication](#authentication)
- [Git Basics](#git-basics)
  - [Creating Repositories](#creating-repositories)
  - [Basic Workflow](#basic-workflow)
  - [Understanding Git States](#understanding-git-states)
- [Branching and Merging](#branching-and-merging)
  - [Working with Branches](#working-with-branches)
  - [Merging Strategies](#merging-strategies)
  - [Handling Merge Conflicts](#handling-merge-conflicts)
- [Rebasing](#rebasing)
  - [Basic Rebasing](#basic-rebasing)
  - [Interactive Rebasing](#interactive-rebasing)
  - [Rebase vs. Merge](#rebase-vs-merge)
- [Working with Remotes](#working-with-remotes)
  - [Remote Management](#remote-management)
  - [Fetching and Pulling](#fetching-and-pulling)
  - [Pushing Changes](#pushing-changes)
- [Tags and Releases](#tags-and-releases)
  - [Working with Tags](#working-with-tags)
  - [Creating Releases](#creating-releases)
  - [Release Management](#release-management)
- [Advanced Git Techniques](#advanced-git-techniques)
  - [Stashing Changes](#stashing-changes)
  - [Cherry-Picking](#cherry-picking)
  - [Reflog](#reflog)
  - [Submodules](#submodules)
  - [Hooks](#hooks)
- [GitHub Specific Features](#github-specific-features)
  - [Pull Requests](#pull-requests)
  - [GitHub Actions](#github-actions)
  - [Issues and Projects](#issues-and-projects)
  - [GitHub Pages](#github-pages)
- [Best Practices](#best-practices)
  - [Commit Messages](#commit-messages)
  - [Workflow Patterns](#workflow-patterns)
  - [Code Review](#code-review)
- [Troubleshooting](#troubleshooting)
  - [Recovering Lost Work](#recovering-lost-work)
  - [Fixing Mistakes](#fixing-mistakes)
  - [Common Errors](#common-errors)

## Introduction

Git is a distributed version control system that allows developers to track changes in source code during software development. Unlike centralized version control systems (such as Subversion or Perforce), Git gives every developer a complete copy of the repository with full history, enabling offline work, faster operations, and better branching capabilities.

### What Makes Git Different

**Distributed Architecture:** Each developer has a complete copy of the repository (including history), allowing for offline work and creating a natural backup system across all users.

**Snapshot-Based:** Instead of storing file differences, Git takes a "snapshot" of all files at each commit. Files that don't change aren't stored again, just linked to the previous identical file.

**Data Integrity:** Everything in Git is checksummed before storage, making it impossible to change content without Git knowing. This integrity is built into the system at the lowest levels.

**Staging Area:** Unlike other VCS, Git has a unique "staging area" (or "index") where changes are prepared before committing, giving developers fine-grained control over what gets recorded in history.

**Branching Model:** Git's branching is lightweight and fast, making it practical to have multiple local branches that can be entirely independent of each other.

### GitHub and Similar Platforms

GitHub is a web-based hosting service for Git repositories that adds collaborative features beyond what Git itself provides:

**Collaboration Tools:** Pull requests, code reviews, issue tracking, and project management boards facilitate teamwork.

**CI/CD Integration:** Built-in continuous integration and deployment capabilities with GitHub Actions.

**Community Features:** Forking, starring, and following repositories creates a social coding environment.

**Documentation:** GitHub Pages, wikis, and automatic README rendering make documentation more accessible.

Other similar platforms include GitLab, Bitbucket, and Azure DevOps, each with their own unique features but building on Git's foundation.

This guide covers Git and GitHub from basics to advanced use cases, with practical scenarios and detailed explanations to help you understand not just how to use Git commands, but when and why to use them and the underlying concepts that make Git work.

## Setting Up Git

### Installation

**Linux (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install git
```

**Verify the installation:**
```bash
git --version
```

**Scenario:** Before starting a new project, ensure your development environment has Git installed and properly configured. This step is crucial for new team members joining a project.

### Configuration

Git configuration exists at three levels: system, global (user), and repository.

#### Configuration Levels Explained

Git's configuration is layered, with each level overriding the previous one:

1. **System level** (`--system`): Applied to every user on the system and all their repositories
   - Location: `/etc/gitconfig`
   - Requires admin privileges to change
   - Rarely needs modification

2. **Global level** (`--global`): Applied to all repositories for the current user
   - Location: `~/.gitconfig` or `~/.config/git/config`
   - Most common place for your personal preferences
   - Affects all your repositories

3. **Repository level** (default): Applied only to the current repository
   - Location: `.git/config` in the repository
   - Overrides system and global settings
   - Useful for repository-specific settings

When Git looks for a configuration value, it checks the repository level first, then global, and finally system level, using the first value it finds.

**Setting up your identity (global):**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Scenario:** You've just joined a new company and need to set up Git on your work laptop with your company email, while keeping your personal projects associated with your personal email.

```bash
# For work repositories
cd /path/to/work/repo
git config user.email "your.work@company.com"

# Your personal projects keep using global settings with personal email
```

#### Understanding Key Configuration Options

**Common configuration settings:**

```bash
# Set default editor
git config --global core.editor "vim"

# Set default branch name
git config --global init.defaultBranch main

# Configure line ending behavior
git config --global core.autocrlf input  # For Linux/Mac
git config --global core.autocrlf true   # For Windows

# Enable colorful output
git config --global color.ui auto

# Create useful aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
```

**Line ending configuration explained:**
- `core.autocrlf = true`: (Windows setting) Converts LF endings to CRLF when checking out files, and CRLF to LF when committing
- `core.autocrlf = input`: (Linux/Mac setting) Converts CRLF to LF when committing but makes no changes when checking out
- `core.autocrlf = false`: Makes no changes in either direction (can cause issues in mixed environments)

This prevents endless conflicts in projects where team members use different operating systems.

**Using aliases effectively:**
Aliases are shortcuts for longer Git commands, saving time and typing. Well-chosen aliases make you more productive and reduce errors. For example:

```bash
# Log aliases for better history visualization
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.today "log --since=midnight --author='$(git config user.name)' --oneline"

# Workflow helpers
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD --stat"
```

**Viewing your configuration:**
```bash
# View all settings
git config --list

# View specific setting
git config user.name

# View all settings with their origin (which config file they're from)
git config --list --show-origin
```

**Scenario:** You're working in a team with different coding styles and need to ensure consistent formatting. You can configure Git to automatically handle line endings and whitespace issues:

```bash
# Warn about whitespace issues
git config --global core.whitespace trailing-space,space-before-tab

# Auto-correct git commands
git config --global help.autocorrect 1
```

The `help.autocorrect` setting is particularly useful as it automatically corrects mistyped commands after a delay (measured in tenths of a second). For example, `git statsu` would be corrected to `git status` after the specified delay.


### Authentication

Authentication in Git ensures that you have permission to interact with remote repositories. There are two main methods: SSH keys and HTTPS credentials.

#### SSH Authentication

SSH (Secure Shell) keys provide a secure way to authenticate without entering a password each time. It uses a pair of keys: a private key (kept secret) and a public key (shared with services like GitHub).

**Why use SSH keys?**
- More secure than password authentication
- No need to enter credentials for each operation
- Can be protected with a passphrase for added security
- Easier to manage multiple accounts and services

**SSH Keys for GitHub:**
```bash
# Generate SSH key with Ed25519 algorithm (more secure and modern than RSA)
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start the SSH agent (manages your keys in memory)
eval "$(ssh-agent -s)"

# Add your SSH key to the agent (so you don't need to re-enter passphrase)
ssh-add ~/.ssh/id_ed25519

# Copy the public key to clipboard
cat ~/.ssh/id_ed25519.pub
# Then add this to your GitHub account settings under SSH and GPG keys
```

**The key generation process explained:**
- `-t ed25519`: Specifies the algorithm (Ed25519 is secure and modern)
- `-C "email"`: Adds a comment to identify the key (not required but helpful)
- During generation, you'll be prompted for a save location (default is fine) and an optional passphrase
- The passphrase adds an extra layer of security but requires entry when the key is used (unless using ssh-agent)

**Testing SSH connection:**
```bash
ssh -T git@github.com
# Expected response: "Hi username! You've successfully authenticated..."
```

#### HTTPS Authentication

HTTPS authentication uses your GitHub username and password or a personal access token. Credential helpers can store these credentials to avoid repetitive entry.

**Using credential helper for HTTPS:**
```bash
# Cache credentials for 15 minutes
git config --global credential.helper cache

# Cache credentials with custom timeout (in seconds)
git config --global credential.helper "cache --timeout=3600"

# Store credentials permanently
git config --global credential.helper store  # Less secure but convenient

# On macOS, use the keychain
git config --global credential.helper osxkeychain

# On Windows, use Windows Credential Manager
git config --global credential.helper manager-core
```

**Credential helper security implications:**
- `cache`: Stores credentials temporarily in memory (most secure, but temporary)
- `store`: Saves credentials to a plain text file (convenient but less secure)
- `osxkeychain`/`manager-core`: Uses system's secure credential storage (good balance)
- For HTTPS, consider using personal access tokens instead of passwords (especially with GitHub, which now requires this)

#### Managing Multiple Accounts

**Scenario:** You need to work with multiple GitHub accounts (personal and work) on the same computer. You can create SSH keys for each and manage them using SSH config:

```bash
# Create separate keys for each account
ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_personal
ssh-keygen -t ed25519 -C "work@company.com" -f ~/.ssh/id_work

# Create or edit ~/.ssh/config file
touch ~/.ssh/config

# Add the following configuration
cat << 'EOF' >> ~/.ssh/config
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_personal
    IdentitiesOnly yes

Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_work
    IdentitiesOnly yes
EOF
```

**How the SSH config works:**
- `Host github.com-personal`: Creates an alias for the connection
- `HostName github.com`: Specifies the actual host to connect to
- `User git`: The username to use for SSH connection (always 'git' for GitHub)
- `IdentityFile`: Path to the specific private key for this connection
- `IdentitiesOnly yes`: Forces SSH to only use the specified key

Then clone repositories using the specific host:
```bash
git clone git@github.com-work:company/project.git
git clone git@github.com-personal:yourname/project.git
```

This setup allows you to use different GitHub accounts seamlessly without authentication conflicts.

## Git Basics

### Creating Repositories

**Initialize a new repository:**
```bash
mkdir my-project
cd my-project
git init
```

**Clone an existing repository:**
```bash
git clone https://github.com/username/repository.git

# Clone with SSH
git clone git@github.com:username/repository.git

# Clone to specific directory
git clone https://github.com/username/repository.git custom-folder
```

**Scenario:** Your team is starting a new project. After creating the initial files locally, you need to set up version control and share it on GitHub:

```bash
# Initialize locally
mkdir new-project && cd new-project
git init
touch README.md
git add README.md
git commit -m "Initial commit"

# Create remote repository on GitHub, then:
git remote add origin git@github.com:username/new-project.git
git push -u origin main
```

### Basic Workflow

**Checking status:**
```bash
git status
```

**Staging changes:**
```bash
# Stage specific files
git add filename.txt

# Stage all changes
git add .

# Stage changes interactively
git add -i

# Stage changes by patch (interactively choose parts of files)
git add -p
```

**Scenario:** You've made multiple unrelated changes to a file but only want to commit some of them:

```bash
git add -p
# Git will show changes in chunks and ask "Stage this hunk?" 
# Respond with:
# y - yes, stage this hunk
# n - no, don't stage this hunk
# s - split into smaller hunks
# e - manually edit the hunk
```

**Committing changes:**
```bash
# Standard commit
git commit -m "Add login feature"

# Add and commit tracked files in one step
git commit -a -m "Update documentation"

# Amend the last commit
git commit --amend -m "New commit message"
```

**Scenario:** You've committed changes but forgot to include a file:

```bash
git add forgotten-file.txt
git commit --amend --no-edit  # Keeps the same commit message
```

**Viewing history:**
```bash
# Basic log
git log

# Compact log
git log --oneline

# Graph view
git log --graph --oneline --decorate

# Show changes
git log -p

# Show stats
git log --stat

# Filter by author
git log --author="John Doe"

# Filter by date
git log --after="2023-01-01" --before="2023-01-31"

# Filter by commit message content
git log --grep="bug fix"

# Filter by file
git log -- filename.txt

# Show only the last N commits
git log -n 5
```

**Scenario:** You need to find when a specific bug was introduced:

```bash
# Search for commits that added or removed the string "buggy-code"
git log -S "buggy-code" -p
```

### Understanding Git States

Files in a Git repository can exist in three states:
- **Modified**: Changes that have not been staged
- **Staged**: Modified files that are marked for the next commit
- **Committed**: Data safely stored in the local repository

**Viewing differences:**
```bash
# Between working directory and staging area
git diff

# Between staging area and last commit
git diff --staged

# Between commits
git diff commit1 commit2

# For a specific file
git diff -- path/to/file
```

**Scenario:** Before committing, you want to double-check what changes are about to be committed:

```bash
git diff --staged
```

## Branching and Merging

Branching is one of Git's most powerful features, allowing developers to work on different parts of a project simultaneously without interfering with each other. Think of branches as independent lines of development within a repository.

### Working with Branches

**Understanding branches conceptually:**

A branch in Git is simply a lightweight, movable pointer to a commit. When you create a new branch, you're creating a new pointer to the current commit—not copying any files. The default branch name in Git is `main` (previously `master`).

**The HEAD pointer:** Git maintains a special pointer called HEAD that points to the current branch you're working on. When you make a commit, the branch that HEAD points to moves forward automatically.

**List branches:**
```bash
# List local branches (* marks current branch)
git branch

# List remote branches
git branch -r

# List all branches (local and remote)
git branch -a

# Show branches with more info (last commit message)
git branch -v

# Show merged or unmerged branches
git branch --merged    # Branches merged into HEAD
git branch --no-merged # Branches not merged into HEAD

# Show branches that contain a specific commit
git branch --contains commit-hash
```

**Create a new branch:**
```bash
# Create branch (doesn't switch to it)
git branch feature-login

# Create and switch to branch
git checkout -b feature-login

# Alternative with newer Git versions (2.23+)
git switch -c feature-login

# Create branch based on specific commit/tag
git checkout -b hotfix-bug commit-hash

# Create branch based on a remote branch
git checkout -b feature origin/feature
```

**Branch visualization:**

When you create a new branch, both branches point to the same commit initially:
```
           HEAD
            │
            v
main ────→ C1
            ↑
      feature-branch
```

After committing on the feature branch:
```
                   HEAD
                    │
                    v
                   C2
                  /
main ────→ C1────┘
```

**Branch naming conventions and strategies:**

Well-organized branch names help teams understand:
- The purpose of the branch
- Who's working on it
- What issue it relates to

**Scenario:** Your team follows a naming convention for branches to organize work:

```bash
# For new features
git checkout -b feature/user-authentication

# For bug fixes
git checkout -b fix/login-error

# For hotfixes on production
git checkout -b hotfix/security-issue main

# For experimental work
git checkout -b experimental/new-ui-framework

# Including ticket numbers
git checkout -b feature/AUTH-123-user-roles
```

**Common branching models:**

- **GitHub Flow**: Simple flow with main branch and feature branches
- **GitFlow**: More structured with develop, feature, release, hotfix, and main branches
- **Trunk-based development**: Short-lived feature branches merged to main frequently

**Switching branches:**
```bash
# Traditional way
git checkout branch-name

# With Git 2.23+
git switch branch-name

# Create and switch in one command
git checkout -b new-branch-name
git switch -c new-branch-name  # With newer Git

# Switch to previous branch
git checkout -
git switch -    # With newer Git
```

**What happens when switching branches:**
- Git updates files in your working directory to match the snapshot of the target branch
- If changes in your working directory conflict with the target branch, Git won't allow the switch
- Uncommitted changes that don't conflict will be carried over to the new branch

**Renaming branches:**
```bash
# Rename current branch
git branch -m new-name

# Rename specific branch
git branch -m old-name new-name

# Rename and push to remote (after local rename)
git push origin :old-name new-name   # Delete old, push new
git push origin -u new-name          # Set upstream tracking
```

**Deleting branches:**
```bash
# Delete merged branch (safe)
git branch -d branch-name

# Force delete unmerged branch
git branch -D branch-name

# Delete remote branch
git push origin --delete branch-name
# Alternative syntax
git push origin :branch-name
```

**Scenario:** After completing and merging a feature, clean up your local branches:

```bash
git checkout main
git pull
git branch -d feature-login  # Will only delete if merged
# If Git warns about unmerged changes, verify it's safe to delete
# then use -D to force deletion

# Delete all merged branches at once (except main)
git branch --merged | grep -v "\*\|main\|master" | xargs -n 1 git branch -d
```

### Merging Strategies

Merging combines changes from different branches. Git offers several strategies for merging, each with its own benefits.

#### Types of Merges

**Basic merge:**
The standard way to integrate changes from one branch into another:
```bash
# Switch to target branch (the one you want to merge into)
git checkout main

# Merge changes from another branch
git merge feature-login
```

**Fast-forward merge:**
When no new commits have been made on the target branch since the feature branch was created, Git performs a "fast-forward" merge by default. This simply moves the branch pointer forward.

Before:
```
      main
       |
C1----C2
       \
        C3----C4
              |
         feature-login
```

After:
```
              main
               |
C1----C2----C3----C4
                   |
            feature-login
```

```bash
git checkout main
git merge feature
# "Fast-forward" message appears
```

**No-fast-forward merge (--no-ff):**
Forces Git to always create a merge commit, even when a fast-forward would be possible. This preserves the record of a feature branch in the history.

Before:
```
      main
       |
C1----C2
       \
        C3----C4
              |
         feature-login
```

After:
```
      main
       |
C1----C2--------C5
       \        /
        C3----C4
              |
         feature-login
```

```bash
git checkout main
git merge --no-ff feature
```

**Scenario:** You want to maintain a clear project history, especially for feature branches:

```bash
# Always create a merge commit, even when a fast-forward is possible
git merge --no-ff feature/user-authentication -m "Merge feature: User Authentication"
```

This practice makes it easier to:
- Identify which commits were part of which feature
- Revert entire features if needed
- Understand the branch structure in visual Git tools

**Squash merge:**
Combines all changes from the source branch into a single commit on the target branch, simplifying history.

Before:
```
      main
       |
C1----C2
       \
        C3----C4----C5
                    |
              feature-login
```

After:
```
      main
       |
C1----C2----C6
       
        C3----C4----C5
                    |
              feature-login
```

```bash
git checkout main
git merge --squash feature
git commit -m "Implement feature-login"
# Creates one new commit with all changes
```

**Comparing merge strategies visually:**
1. **Fast-forward**: Move pointer forward, linear history
2. **No-fast-forward**: Create merge commit, preserve branch structure
3. **Squash**: Create single commit with all changes, simplify history

**When to use each strategy:**
- **Fast-forward**: Simple feature branches that don't need to be tracked separately
- **No-fast-forward**: When branch history is important to preserve
- **Squash**: When the implementation details of a feature are less important than the final result

**Scenario:** A feature branch has many small, incremental commits that would clutter the main branch's history:

```bash
git checkout main
git merge --squash feature/complex-feature
git commit -m "Add complex feature with all components"
```

**Advanced merge options:**
```bash
# Specify merge strategy
git merge -s recursive feature

# Specify strategy options
git merge -X ignore-space-change feature

# Automatically resolve conflicts favoring one side
git merge -X ours feature     # Keep target branch version
git merge -X theirs feature   # Use incoming branch version
```

### Handling Merge Conflicts

Merge conflicts occur when Git can't automatically resolve differences between branches. This typically happens when the same lines of code have been modified differently in each branch.

**Understanding conflict markers:**
When a conflict occurs, Git modifies the affected files to show both versions:

```
<<<<<<< HEAD
current branch content (the branch you're merging into)
=======
incoming branch content (the branch you're merging from)
>>>>>>> feature
```

**Strategies for resolving conflicts:**

1. **Manual resolution:**
```bash
# After a merge conflict occurs
git status  # See which files have conflicts

# Edit files to resolve conflicts manually
# Remove conflict markers and keep what you want

# After resolving
git add resolved-file.txt
git commit  # Complete the merge
```

2. **Using visual merge tools:**
```bash
# Configure your preferred merge tool
git config --global merge.tool vscode

# Open the configured tool
git mergetool

# After resolving all conflicts
git commit
```

Popular merge tools include:
- VS Code
- Kdiff3
- P4Merge
- Beyond Compare
- Meld

3. **Accepting one version entirely:**
```bash
# Accept target branch (your) version for a specific file
git checkout --ours conflicted_file.js
git add conflicted_file.js

# Accept incoming branch (their) version
git checkout --theirs conflicted_file.js
git add conflicted_file.js
```

**Aborting a merge:**
If you want to back out of a merge and try a different approach:
```bash
git merge --abort
```

**Previewing merges:**
Check for potential conflicts before performing the merge:
```bash
# Dry run - don't actually merge, just check for conflicts
git merge --no-commit --no-ff feature
# If there are conflicts, resolve them
# If you want to cancel: git merge --abort
# If you want to continue: git merge --continue
```

**Scenario:** You and a colleague both modified the same part of a configuration file:

```bash
git checkout main
git merge feature/new-config
# CONFLICT in config.json

# Open the file and see:
# <<<<<<< HEAD
# "timeout": 30,
# =======
# "timeout": 60,
# >>>>>>> feature/new-config

# After discussing with your team, keep the new value:
# Then:
git add config.json
git commit -m "Merge: Update timeout to 60 seconds"
```

**Best practices for preventing merge conflicts:**
1. Pull/rebase frequently to stay up to date with the target branch
2. Break large changes into smaller, focused commits
3. Communicate with team members working on the same files
4. Use feature toggles for long-running features
5. Structure your codebase to minimize overlap between features

## Rebasing

Rebasing is a powerful Git technique that rewrites history by moving or combining commits. It's like saying, "I want to pretend my changes were based on the latest code, not the code from when I started working."

### Basic Rebasing

At its core, rebasing takes your commits from one branch and replays them on top of another branch, creating a cleaner, linear history.

**Under the hood:** When you rebase, Git:
1. Identifies the common ancestor of your branch and the target branch
2. Gets the diff (changes) introduced by each commit on your branch
3. Temporarily saves those changes
4. Moves your branch pointer to the tip of the target branch
5. Applies each saved change, one by one, creating new commits

**Rebasing a branch:**
```bash
# While on feature branch
git rebase main

# Rebase with a specific base
git rebase --onto new-base old-base feature-branch
```

The `--onto` option is particularly powerful for more complex rebases:
- `new-base`: The commit where you want to place your changes
- `old-base`: The commit before your changes start 
- `feature-branch`: The branch containing your changes

**Scenario:** You've been working on a feature branch, but the main branch has moved forward with changes you need:

```bash
git checkout feature/login
git rebase main
# Now your feature branch changes appear as if they were made after the latest main
```

**Handling conflicts during rebase:**
```bash
# When conflicts occur, Git pauses the rebase
# After fixing conflicts in the files:
git add <resolved-files>
git rebase --continue

# If you want to abort the rebase entirely:
git rebase --abort

# If you want to skip the current commit:
git rebase --skip  # Be careful, this discards the changes in that commit
```

### Interactive Rebasing

Interactive rebasing gives you precise control over your commit history, allowing you to clean up your work before sharing it. Think of it as a powerful history editing tool.

```bash
# Rebase the last 3 commits
git rebase -i HEAD~3

# Rebase commits after a specific commit
git rebase -i commit-hash
```

**Interactive rebase command options explained:**
- `pick` (p): Use the commit as is (default)
- `reword` (r): Use the commit but edit its message
- `edit` (e): Pause the rebase at this commit to amend it (add or remove changes)
- `squash` (s): Combine with previous commit and merge commit messages
- `fixup` (f): Like squash, but discard this commit's message entirely
- `drop` (d): Remove the commit completely (delete its changes)
- `reorder`: Simply change the order of lines in the editor
- `exec` (x): Run a shell command after the commit is applied

When you save and close the editor, Git executes your instructions in order from top to bottom.

**Advanced interactive rebase example:**
```bash
git rebase -i HEAD~7

# Original list:
pick 2c6a45d Add user model
pick 9e78i65 Create database migration
pick f930e22 Fix linting errors in user model
pick 1f6ea91 Add authentication endpoints
pick 0c3k0d7 Add tests for auth endpoints
pick 87d2p1c Fix auth header bug
pick 2b3da56 Add documentation

# Changed to:
pick 2c6a45d Add user model
fixup f930e22 Fix linting errors in user model
pick 9e78i65 Create database migration
pick 1f6ea91 Add authentication endpoints
pick 0c3k0d7 Add tests for auth endpoints
fixup 87d2p1c Fix auth header bug
reword 2b3da56 Add documentation
```

This will:
1. Combine the linting fix with the user model commit
2. Reorder to put database migration after the user model
3. Combine the header bug fix with the auth endpoints commit
4. Prompt you to edit the documentation commit message

**Scenario:** You want to clean up your branch before creating a pull request:

```bash
git rebase -i HEAD~5
# In editor, change:
pick e2dc1a3 Add login feature
pick 91b35ce Fix typo in login form
pick 43f08d0 Update login UI
pick a5c4682 Another typo fix
pick fb78291 Add remember me checkbox

# To:
pick e2dc1a3 Add login feature
fixup 91b35ce Fix typo in login form
pick 43f08d0 Update login UI
fixup a5c4682 Another typo fix
pick fb78291 Add remember me checkbox

# This combines the typo fixes with their related commits
```

### Rebase vs. Merge

Understanding when to rebase and when to merge is crucial for effective Git workflow.

**Conceptual difference:**
- **Merge:** Creates a new "merge commit" that ties two branches together, preserving the complete history
- **Rebase:** Rewrites history by moving your branch's base to the tip of another branch, resulting in a linear history

**Visual representation:**

Before either operation:
```
      A---B---C (feature)
     /
D---E---F---G (main)
```

After merge:
```
      A---B---C
     /         \
D---E---F---G---H (merge commit)
```

After rebase:
```
              A'--B'--C' (feature)
             /
D---E---F---G (main)
```

Note that after rebasing, A, B, C are replaced with new commits A', B', C' with different hashes but same content changes.

**When to rebase:**
- Keeping feature branches updated with the main branch
- Cleaning up your branch before sharing
- When you want a linear history without merge commits
- For incorporating upstream changes into your feature branch
- When your branch's history doesn't need to reflect exactly when you did the work

**When to merge:**
- Integrating completed features into the main branch
- Preserving the full history of work, including when branches diverged
- When collaborating on a shared branch
- When you want to clearly see when and how a feature was integrated
- For long-running branches where preserving context is important

**Real-world scenario:** You're working on a long-running feature branch and need to incorporate new changes from main:

```bash
# While on feature branch
git fetch origin
git rebase origin/main
# Resolve any conflicts
git push --force-with-lease  # If branch is already pushed to remote
```

**Understanding `--force-with-lease`**:
- Safer than `--force` as it checks if the remote branch has been updated by someone else
- Prevents accidentally overwriting others' work
- Aborts if remote has new commits you haven't seen

**Rebase golden rule:** Never rebase a branch that others are working on. Doing so will cause serious problems when they try to merge their changes or pull your changes.

## Working with Remotes

### Remote Management

**Adding a remote:**
```bash
# Standard syntax
git remote add origin https://github.com/username/repo.git

# Adding multiple remotes
git remote add upstream https://github.com/original-owner/repo.git
```

**Viewing remotes:**
```bash
# List remotes
git remote -v

# View details about a specific remote
git remote show origin
```

**Renaming and removing remotes:**
```bash
# Rename a remote
git remote rename origin upstream

# Remove a remote
git remote remove upstream
```

**Scenario:** You're contributing to an open source project by working with your own fork and the original repository:

```bash
# Clone your fork
git clone https://github.com/your-username/project.git

# Add the original repository as 'upstream'
git remote add upstream https://github.com/original-owner/project.git

# Now you can fetch updates from the original repo
git fetch upstream

# And merge them into your local branch
git merge upstream/main
```

### Fetching and Pulling

**Fetching from a remote:**
```bash
# Fetch from default remote (origin)
git fetch

# Fetch from specific remote
git fetch upstream

# Fetch a specific branch
git fetch origin feature-branch

# Fetch all branches from all remotes
git fetch --all
```

**Pulling changes:**
```bash
# Pull from current branch's upstream
git pull

# Pull with rebase instead of merge
git pull --rebase

# Pull from specific remote and branch
git pull origin main
```

**Scenario:** You're working on a team project and need to get the latest changes before starting your workday:

```bash
# Update your local repo with all remote changes
git fetch --all

# See what's changed
git log --oneline main..origin/main

# Update your main branch
git checkout main
git merge origin/main  # Or git pull

# Create a new branch for today's work based on the latest main
git checkout -b feature/todays-task
```

### Pushing Changes

**Basic push:**
```bash
# Push current branch to its upstream
git push

# Push to specific remote and branch
git push origin feature-branch

# Push all local branches
git push --all origin

# Push tags
git push --tags
```

**Setting upstream tracking:**
```bash
# Push and set upstream
git push -u origin feature-branch

# Set upstream for current branch
git branch --set-upstream-to=origin/feature-branch
```

**Force pushing:**
```bash
# Force push (use with caution)
git push --force

# Safer force push
git push --force-with-lease
```

**Scenario:** After rebasing your branch to clean up history, you need to update the remote:

```bash
# First make sure you're on the right branch
git checkout feature/login

# Check if anyone else pushed changes
git fetch origin

# If branch head has changed, reconsider force pushing
# If it's safe to proceed:
git push --force-with-lease origin feature/login
```

## Tags and Releases

Tags in Git provide a way to mark specific points in history as important, typically used for marking release versions. Releases build upon tags by adding additional metadata, release notes, and binary assets.

### Working with Tags

Git offers two types of tags, each serving different purposes:

1. **Lightweight tags:** Simple pointers to specific commits (like a branch that doesn't move)
2. **Annotated tags:** Full Git objects containing tagger information, date, message, and can be signed and verified with GPG

**Creating tags:**
```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended for releases)
git tag -a v1.0.0 -m "Version 1.0.0 - Initial release"

# Tagging a specific commit
git tag -a v0.9.0 -m "Beta release" commit-hash

# GPG-signed tag for enhanced security
git tag -s v1.1.0 -m "Signed release v1.1.0"
```

**Understanding tag types:**
- **Lightweight tags**: Simply store a reference to a commit - useful for temporary or personal markers
- **Annotated tags**: Store complete objects in the Git database - preferred for public releases since they include:
  - Tagger name and email
  - Creation date
  - A tagging message
  - Optional GPG signature for verification

**Listing and filtering tags:**
```bash
# List all tags
git tag

# List tags matching a pattern
git tag -l "v1.*"

# Show tag details (for annotated tags)
git show v1.0.0

# Sort tags by version number (useful for semantic versioning)
git tag --sort=v:refname

# List tags with their commit messages
git tag -n

# List tags with date information
git for-each-ref --sort=taggerdate --format '%(refname:short) %(taggerdate:short) %(subject)' refs/tags
```

**Sharing tags:**
By default, `git push` doesn't transfer tags to remote servers. You need to explicitly push them:

```bash
# Push a specific tag
git push origin v1.0.0

# Push all tags at once
git push origin --tags

# Push tags matching a pattern
git push origin --tags --follow-tags 'v1.*'
```

**Deleting tags:**
```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Delete multiple tags at once
git tag -d v0.1.0 v0.2.0 v0.3.0-beta
git push origin --delete v0.1.0 v0.2.0 v0.3.0-beta
```

**Checking out tags:**
Tags aren't branches, but you can view the code at a tagged point:
```bash
# View code at a tag point (detached HEAD state)
git checkout v1.0.0

# Create a new branch from a tag to make changes
git checkout -b hotfix/critical-bug v1.0.0
```

**Semantic versioning explained:**
Semantic Versioning (SemVer) follows the format MAJOR.MINOR.PATCH:
- MAJOR: Incompatible API changes
- MINOR: Added functionality (backward-compatible)
- PATCH: Bug fixes (backward-compatible)

Additional labels for pre-release and build metadata: v1.0.0-alpha.1, v1.0.0-beta.2, v1.0.0+build.123

**Scenario:** Your team uses semantic versioning for software releases:

```bash
# After completing all features for a release
git checkout main
git pull
git tag -a v2.3.0 -m "Version 2.3.0 - Add user authentication and reporting"
git push origin v2.3.0

# For a hotfix on production
git checkout main
git checkout -b hotfix/security-fix
# Make changes
git commit -m "Fix security vulnerability in login"
git checkout main
git merge hotfix/security-fix
git tag -a v2.3.1 -m "Version 2.3.1 - Security patch"
git push origin main --tags
```

### Creating Releases

Releases expand on tags by providing a way to package and deliver software to users with release notes and binary assets.

**GitHub releases:**
1. Navigate to your repository on GitHub
2. Click on "Releases"
3. Click "Create a new release"
4. Select the tag (existing or create new)
5. Add title and description (supports markdown)
6. Optionally mark as pre-release or latest release
7. Attach binaries, installer packages, or other assets
8. Publish the release

**Release best practices:**
- Use consistent naming conventions for tags and releases
- Include clear, concise release notes with sections for:
  - New features
  - Bug fixes
  - Breaking changes
  - Upgrade instructions if needed
- Attach compiled binaries for easy download
- Link to relevant issues and pull requests
- Include screenshots for UI changes

**Release automation with GitHub Actions:**
```yaml
name: Create Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build project
        run: |
          npm install
          npm run build
          
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
          
      - name: Upload Release Asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./dist/app.zip
          asset_name: app.zip
          asset_content_type: application/zip
```

### Release Management

Release management involves planning, scheduling, and controlling software builds through different environments.

**Release workflow example:**
1. Create a release branch from development: `git checkout -b release/v2.0.0 develop`
2. Stabilize the release branch (only bug fixes, no new features)
3. Finalize version numbers in code and documentation
4. Merge to main when ready: `git checkout main && git merge --no-ff release/v2.0.0`
5. Tag the release: `git tag -a v2.0.0 -m "Version 2.0.0"`
6. Merge back to development: `git checkout develop && git merge --no-ff release/v2.0.0`
7. Delete the release branch: `git branch -d release/v2.0.0`
8. Push changes and tags: `git push origin main develop --tags`

**Git-flow release process:**
Git-flow is a branching model that formalizes the release process:
```bash
# Initialize git-flow
git flow init

# Start a release
git flow release start v1.2.0

# Finish a release (merges to main, tags, merges back to develop)
git flow release finish v1.2.0

# Push tags and branches
git push origin --tags
git push origin main develop
```
6. Attach binaries if needed
7. Publish release

**Using Git to create a release:**
```bash
# Create an annotated tag
git tag -a v1.0.0 -m "Version 1.0.0"

# Push the tag
git push origin v1.0.0

# Then create a release on GitHub based on this tag
```

**Scenario:** You've completed version 2.0 with major improvements and want to create a proper release:

Create a release branch:
```bash
git checkout -b release/2.0.0
```
Update version numbers and documentation, then merge:
```bash
git checkout main
git merge --no-ff release/2.0.0
git tag -a v2.0.0 -m "Version 2.0.0 - Major update with new features"
git push origin main --tags
```

## Release Management

**Tags vs. Releases:**
- Tags: Git feature that marks specific points in history (like a bookmark)
- Releases: GitHub feature built on tags that provides a way to package and deliver software

**Release branches:**
```bash
# Create a release branch
git checkout -b release/v1.0 main

# Make release-specific changes like version bumps
git commit -m "Bump version to 1.0.0"

# When ready, merge to main
git checkout main
git merge --no-ff release/v1.0
git tag -a v1.0.0
git push origin main --tags

# Also merge into develop for future work
git checkout develop
git merge --no-ff release/v1.0
git push origin develop
```

**Scenario:** Your team follows GitFlow and is preparing a new release:

```bash
# Create release branch from develop
git checkout develop
git pull
git checkout -b release/2.5.0

# Update version numbers
# Fix last minute small bugs

# When ready, finalize release
git checkout main
git merge --no-ff release/2.5.0 -m "Release 2.5.0"
git tag -a v2.5.0 -m "Version 2.5.0"

# Update develop branch with any changes made in the release branch
git checkout develop
git merge --no-ff release/2.5.0 -m "Merge release 2.5.0 back into develop"

# Push everything
git push origin develop main --tags
```

## Advanced Git Techniques

As you become more comfortable with Git, you'll discover powerful features that can help you solve complex version control problems. This section covers techniques that go beyond the basics but can dramatically improve your workflow efficiency.

### Stashing Changes

Git stash is like a clipboard for your working directory, allowing you to temporarily store uncommitted changes so you can work on something else and come back to your changes later.

**What happens when you stash:**
When you run `git stash`, Git:
1. Takes all modified tracked files and stages them
2. Saves them on a stack of unfinished changes
3. Reverts the working directory to match the HEAD commit
4. Lets you work on something else without committing half-done work

**Basic stashing operations:**
```bash
# Stash current changes (modified tracked files)
git stash

# Stash with a descriptive message (recommended)
git stash push -m "WIP: Half-implemented login feature"

# Stash including untracked files
git stash -u  # or --include-untracked

# Stash including ignored files
git stash -a  # or --all (includes both untracked and ignored)

# Stash specific files
git stash push path/to/file1.js path/to/file2.js -m "Specific files"

# Interactively choose which changes to stash
git stash -p  # or --patch
```

**Working with multiple stashes:**
Stashes are stored on a stack, with the most recent stash at position 0.

```bash
# List all stashes
git stash list
# Example output:
# stash@{0}: WIP on feature-branch: 5fc0dd9 Add user profile page
# stash@{1}: On main: WIP: Half-implemented login feature

# Show stash contents
git stash show stash@{0}  # Summary of changes
git stash show -p stash@{0}  # With full diff

# Apply stash (keeps the stash in the stack)
git stash apply  # Applies the most recent stash
git stash apply stash@{2}  # Applies a specific stash

# Pop stash (applies and removes it from the stack)
git stash pop  # Pops the most recent stash
git stash pop stash@{2}  # Pops a specific stash

# Create a branch from a stash
git stash branch new-branch stash@{1}

# Remove stashes
git stash drop stash@{1}  # Remove a specific stash
git stash clear  # Remove all stashes
```

**Stash visualization:**

Before stashing:
```
Working Directory: [Modified Files]
Staging Area: [Staged Changes]
Repository: [Last Commit]
```

After stashing:
```
Working Directory: [Clean]
Staging Area: [Clean] 
Repository: [Last Commit]
Stash Stack: [Stashed Changes]
```

**Handling conflicts when applying stashes:**
```bash
# If stash application causes conflicts
git status  # Check which files have conflicts
# Resolve conflicts manually
git add resolved-files
git reset --hard  # If you want to abort
```

**Scenario:** You're working on a feature but need to switch branches to fix an urgent bug:

```bash
# Save your current work
git stash push -m "WIP: User profile feature"

# Switch to main and create bug fix branch
git checkout main
git checkout -b hotfix/login-bug

# Fix the bug
# ...
git commit -m "Fix login authentication bug"
git push origin hotfix/login-bug

# Go back to your feature
git checkout feature/user-profile
git stash pop  # Resume your work
```

**Advanced stash techniques:**
```bash
# Keep staged changes in working directory but stash other changes
git stash --keep-index

# Stash but keep working directory unchanged
git stash --no-keep-index

# Create a patch file from stash
git stash show -p stash@{0} > patch.diff

# Apply a stash without using git stash
git apply patch.diff
```

### Cherry-Picking

Cherry-picking allows you to select specific commits from one branch and apply them to another. Unlike merging or rebasing, which typically apply a range of commits, cherry-picking gives you precise control over which individual changes to include.

**Conceptual understanding:**
Cherry-picking works by:
1. Extracting the changes introduced in a commit
2. Creating a new commit with these changes on your current branch
3. Generating a new commit hash (since commit metadata changes)

**When to use cherry-picking:**
- Applying a specific bugfix to multiple branches
- Backporting features to maintenance branches
- Recovering specific work from an abandoned branch
- Selectively applying changes from a colleague's work

**Basic cherry-picking:**
```bash
# Apply a specific commit to current branch
git cherry-pick commit-hash

# Cherry-pick without committing (changes staged only)
git cherry-pick -n commit-hash  # or --no-commit

# Cherry-pick a range of commits (exclusive of start-hash)
git cherry-pick start-hash..end-hash

# Cherry-pick a range of commits (inclusive of start-hash)
git cherry-pick start-hash^..end-hash

# Cherry-pick keeping original committer info
git cherry-pick -x commit-hash

# Cherry-pick with custom commit message
git cherry-pick -e commit-hash  # Opens editor
```

**Handling cherry-pick conflicts:**
```bash
# If conflicts occur during cherry-pick:
git status  # Check conflicted files

# After resolving conflicts:
git add resolved-files
git cherry-pick --continue

# To cancel the cherry-pick:
git cherry-pick --abort

# To skip this commit and move to next (if cherry-picking multiple):
git cherry-pick --skip
```

**Advanced cherry-pick options:**
```bash
# Cherry-pick a merge commit (specify parent)
git cherry-pick -m 1 merge-commit-hash
# The -m flag specifies which parent to consider as mainline
# Use 1 for the first parent (the branch being merged into)
# Use 2 for the second parent (the branch being merged)

# Cherry-pick only if it applies cleanly, abort if conflicts
git cherry-pick --ff commit-hash

# Cherry-pick a commit but record the fact it was cherry-picked
git cherry-pick -x commit-hash
# Adds: "(cherry picked from commit abc123)" to commit message
```

**Real-world scenarios:**

1. **Security patch across multiple versions:**
```bash
# Find the security fix commit
git log --grep="CVE-2023" --oneline

# Apply to release branches
git checkout release/2.0
git cherry-pick abc1234
git push origin release/2.0

git checkout release/1.5
git cherry-pick abc1234
git push origin release/1.5
```

2. **Feature backporting:**
```bash
# Identify the commits implementing the feature
git log --oneline feature/payments

# Switch to the maintenance branch
git checkout release/1.0

# Cherry-pick the feature commits
git cherry-pick abc1234 def5678 ghi9101

# If there are conflicts, resolve them and:
git add fixed-files.js
git cherry-pick --continue

# Push the changes
git push origin release/1.0
```

3. **Selective code review feedback:**
```bash
# Create a fixup branch
git checkout -b fix-feedback main

# Cherry-pick just the commits that need changes
git cherry-pick -n abc1234 def5678

# Make the requested changes
# ...

# Commit with new message
git commit -m "Implement feedback from code review"
```

**Cherry-picking best practices:**
- Use sparingly, as it creates duplicate commits with different hashes
- Document cherry-picked commits in the message (use `-x` flag)
- Be careful with dependent commits (cherry-pick them in order)
- Consider rebasing or merging for applying many sequential commits

### Reflog

The Git reference log (reflog) is your safety net, tracking all changes to branch tips and other references in your local repository. While the regular log shows the commit history, reflog shows the history of your actions, making it invaluable for recovering from mistakes.

**Understanding reflog:**
- Each time you commit, rebase, merge, reset, checkout, etc., Git records the action in the reflog
- Reflog entries are local to your repository and not pushed to remotes
- Entries expire after 90 days by default (configurable with `gc.reflogExpire`)
- Each entry is identified by `reference@{position}` where position is a time-based index

**Basic reflog commands:**
```bash
# View reflog for HEAD
git reflog

# View reflog with more detailed information
git reflog --date=iso

# View reflog for a specific reference
git reflog show main
git reflog show HEAD
git reflog show refs/heads/feature

# View reflog entries for a specific action
git reflog --grep-reflog="checkout"
git reflog --grep-reflog="merge"
```

**Reflog vs. Log:**
- `git log` shows the commit history of a branch
- `git reflog` shows the history of operations in your local repository
- Reflog includes "orphaned" commits that are no longer part of any branch

**Recovering from common mistakes:**

1. **Recover from a bad reset:**
```bash
# Check the reflog to find the state before reset
git reflog
# Example output:
# abc1234 HEAD@{0}: reset --hard HEAD~3: updating HEAD
# def5678 HEAD@{1}: commit: Important feature completed

# Recover to the state before the reset
git reset --hard def5678  # or HEAD@{1}
```

2. **Recover a deleted branch:**
```bash
# Check reflog to find the tip of the deleted branch
git reflog
# Look for the last commit on that branch before deletion
# Example: def5678 HEAD@{5}: commit: Last commit on deleted-branch

# Create new branch pointing to that commit
git checkout -b recovered-branch def5678
```

3. **Recover from a bad rebase or merge:**
```bash
# Find the state before the operation
git reflog
# Example for rebase:
# abc1234 HEAD@{0}: rebase finished: refs/heads/feature
# def5678 HEAD@{1}: checkout: moving from main to feature

# Reset to the state before rebase
git reset --hard def5678  # or HEAD@{1}
```

**Advanced reflog techniques:**
```bash
# See what the main branch pointed to 2 days ago
git show main@{2.days.ago}

# Compare current state with where HEAD was 5 operations ago
git diff HEAD HEAD@{5}

# See commits made in the last hour
git log --since="1 hour ago" --all
```

**Scenario:** You accidentally reset your branch and lost commits:

```bash
# Check the reflog to find the commit before the reset
git reflog
# Example output:
# abc1234 HEAD@{0}: reset --hard HEAD~3: updating HEAD
# def5678 HEAD@{1}: commit: Important feature completed
# ...

# Recover the lost work
git checkout -b recovery-branch def5678
# or
git reset --hard def5678
```

### Submodules

Submodules allow you to include other Git repositories within your repository, enabling you to keep a dependency as a separate project while still integrating it into your main project.

**Submodule concepts:**
- Each submodule is a completely separate Git repository
- The main repository tracks the specific commit of the submodule
- Changes in submodules don't automatically affect the main repository

**Adding and initializing submodules:**
```bash
# Add a submodule to your repository
git submodule add https://github.com/username/repo.git path/to/submodule
# This creates a .gitmodules file and the submodule directory

# Initialize submodules after cloning a repository
git submodule init  # Register submodules in .gitmodules
git submodule update  # Clone/checkout submodules to the committed state

# Clone a repository including all submodules
git clone --recurse-submodules https://github.com/username/repo.git
```

**Working with submodules:**
```bash
# View status of submodules
git submodule status

# Update all submodules to latest committed state in the main repo
git submodule update

# Update all submodules to their latest remote version
git submodule update --remote

# Update specific submodule to latest remote version
git submodule update --remote path/to/submodule

# Execute command in each submodule
git submodule foreach 'git checkout main'

# Removing a submodule (Git >= 1.8.3)
git submodule deinit path/to/submodule
git rm path/to/submodule
git commit -m "Remove submodule"
```

**Making changes in submodules:**
```bash
# Enter the submodule directory
cd path/to/submodule

# Make changes as in any Git repository
git checkout -b feature
# Edit files
git add .
git commit -m "Make changes in submodule"

# Push changes to submodule's remote
git push origin feature

# Return to main project and record the new submodule state
cd ../..
git add path/to/submodule
git commit -m "Update submodule to include new feature"
git push
```

**Submodule best practices:**
- Always use absolute URLs for submodules (https:// or git@)
- Consider using tags or specific commits for stability
- Document submodule use in your README
- Be careful when switching branches with different submodule versions

**Scenario:** You're developing a website that uses a shared UI component library:

```bash
# Add UI library as submodule
git submodule add https://github.com/company/ui-components.git lib/ui

# Later, update to the latest version
cd lib/ui
git fetch origin
git checkout v2.5.0  # Switch to a specific tag
cd ../..
git add lib/ui
git commit -m "Update UI library to v2.5.0"
```

### Hooks

Git hooks are scripts that run automatically when specific Git events occur, allowing you to customize Git's behavior and enforce workflows.

**Understanding Git hooks:**
- Stored in the `.git/hooks` directory of your repository
- Named according to the event they handle (e.g., `pre-commit`, `post-merge`)
- Must be executable files (chmod +x)
- Can be written in any language (bash, Python, Ruby, etc.)
- Can be bypassed with `--no-verify` (except server-side hooks)

**Common client-side hooks:**
- `pre-commit`: Runs before a commit is created (linting, testing)
- `prepare-commit-msg`: Runs before the commit message editor is launched
- `commit-msg`: Validates commit messages
- `post-commit`: Notification after commit is complete
- `pre-push`: Final check before pushes
- `post-checkout`: Environment setup after checkout
- `pre-rebase`: Verification before rebase

**Common server-side hooks:**
- `pre-receive`: Runs when receiving a push before any refs are updated
- `update`: Similar to pre-receive but runs once per pushed branch
- `post-receive`: Notification after push is completed (CI/CD, deployments)

**Creating a basic hook:**
```bash
# Create a pre-commit hook for lint checking
cat << 'EOF' > .git/hooks/pre-commit
#!/bin/bash
echo "Running linting..."
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed! Commit aborted."
    exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
```

**Sharing hooks with the team:**
Git doesn't track the hooks directory, but you can:
1. Store hooks in a project directory
2. Create symbolic links or a setup script
3. Use tools like Husky (for npm projects)

```bash
# Store hooks in project dir and create symbolic links
mkdir -p .githooks
cp .git/hooks/pre-commit .githooks/
ln -sf ../../.githooks/pre-commit .git/hooks/pre-commit
```

**Using Husky for Node.js projects:**
```bash
# Install Husky
npm install husky --save-dev

# In package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test && npm run lint",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```

**Real-world examples:**

1. **Enforcing commit message format:**
```bash
#!/bin/bash
# .git/hooks/commit-msg

MESSAGE=$(cat $1)
PATTERN="^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"

if ! [[ $MESSAGE =~ $PATTERN ]]; then
  echo "ERROR: Invalid commit message format."
  echo "Must start with type(scope): description"
  echo "Types: feat, fix, docs, style, refactor, test, chore"
  exit 1
fi
```

2. **Preventing direct commits to main:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

BRANCH=$(git symbolic-ref HEAD)
if [[ $BRANCH == "refs/heads/main" || $BRANCH == "refs/heads/master" ]]; then
  echo "ERROR: Direct commits to main/master are not allowed."
  echo "Please create a feature branch instead."
  exit 1
fi
```

3. **Automated version bumping:**
```bash
#!/bin/bash
# .git/hooks/post-commit

LAST_COMMIT_MSG=$(git log -1 HEAD --pretty=format:%s)

if [[ $LAST_COMMIT_MSG == *"RELEASE"* ]]; then
  echo "Release commit detected, bumping version..."
  npm version patch -m "Bump version to %s [skip ci]"
fi
```
git submodule update --remote lib/ui
git add lib/ui
git commit -m "Update UI components to latest version"
```

### Hooks

Git hooks are scripts that run automatically before or after Git commands.

**Common hooks:**
- `pre-commit`: Runs before commit is created
- `prepare-commit-msg`: Modifies default commit message
- `commit-msg`: Validates commit message
- `post-commit`: Runs after commit is created
- `pre-push`: Runs before push is executed
- `post-checkout`: Runs after checkout
- `pre-rebase`: Runs before rebase

**Creating a hook:**
```bash
# Navigate to hooks directory
cd .git/hooks

# Create a hook script (remove .sample extension)
cp pre-commit.sample pre-commit

# Edit the script and make it executable
chmod +x pre-commit
```

**Scenario:** You want to ensure code standards before each commit:

```bash
# Create a pre-commit hook (.git/hooks/pre-commit)
#!/bin/bash
echo "Running code linting..."
npm run lint

# If linting fails, prevent the commit
if [ $? -ne 0 ]; then
  echo "Linting failed! Aborting commit."
  exit 1
fi
```

## GitHub Specific Features

### Pull Requests

**Creating a pull request:**
1. Push your branch to GitHub
   ```bash
   git push -u origin feature/awesome-feature
   ```
2. Visit your repository on GitHub
3. Click "Pull requests" > "New pull request"
4. Select base branch and compare branch
5. Add title, description, and create the PR

**Pull request workflow:**
```bash
# Start with an up-to-date main branch
git checkout main
git pull

# Create a feature branch
git checkout -b feature/new-feature

# Make changes and commit
git add .
git commit -m "Implement new feature"

# Push to GitHub
git push -u origin feature/new-feature
```

**Updating a pull request:**
```bash
# After receiving feedback, make changes
git add .
git commit -m "Address PR feedback"
git push origin feature/new-feature
```

**Scenario:** You need to incorporate feedback from a code review:

```bash
# Make requested changes
git add updated-files.js
git commit -m "Address feedback: Fix error handling"
git push origin feature/login

# If you need to make additional changes:
git add src/components/Login.js
git commit --amend --no-edit  # If minor change to previous commit
git push --force-with-lease origin feature/login
```

### GitHub Actions

**Basic workflow file:**
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 16
    - name: Install dependencies
      run: npm ci
    - name: Run tests
      run: npm test
```

**Scenario:** You want to automate testing and deployment of your application:

Create `.github/workflows/ci-cd.yml`:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 16
      - run: npm ci
      - run: npm test
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Deployment commands
```

### Issues and Projects

**Creating issue templates:**
Create `.github/ISSUE_TEMPLATE/bug_report.md`:
```markdown
---
name: Bug Report
about: Create a report to help us improve
title: '[BUG] '
labels: bug
assignees: ''
---

**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected behavior**
A clear and concise description of what you expected to happen.
```

**Linking issues and pull requests:**
```bash
# In commit message
git commit -m "Fix login bug, closes #42"

# In PR description:
# "This PR fixes #42"
```

**Scenario:** Using GitHub to manage a sprint:

1. Create a Project board with columns: To Do, In Progress, Review, Done
2. Create issues for each task
3. Link issues to pull requests
4. Use labels to categorize (bug, enhancement, documentation)
5. Track progress using milestones

### GitHub Pages

**Setting up GitHub Pages:**
```bash
# Create a branch for GitHub Pages
git checkout -b gh-pages

# Add static website files
git add .
git commit -m "Initial GitHub Pages site"
git push -u origin gh-pages
```

**Using a docs folder:**
```bash
# Create docs directory on main branch
mkdir docs
# Add website files to docs
git add docs
git commit -m "Add GitHub Pages site in docs folder"
git push origin main
```

**Scenario:** Publishing project documentation:

```bash
# Generate docs with a tool like JSDoc
npm run generate-docs

# Move generated files to docs folder
mv out/ docs/

# Commit and push
git add docs
git commit -m "Update project documentation"
git push origin main
```

## Best Practices

### Commit Messages

Well-crafted commit messages are crucial for maintainability, collaboration, and project history. Good commit messages help others (and your future self) understand why changes were made, not just what changed.

**The seven rules of a great commit message:**
1. Separate subject from body with a blank line
2. Limit the subject line to 50 characters
3. Capitalize the subject line
4. Do not end the subject line with a period
5. Use the imperative mood in the subject line ("Add feature" not "Added feature")
6. Wrap the body at 72 characters
7. Use the body to explain what and why vs. how

**Structure of a good commit message (Conventional Commits):**
```
<type>(<scope>): <short summary>

<body>

<footer>
```

**Example of a well-structured commit:**
```
feat(auth): implement OAuth2 login

Add support for OAuth2 authentication with Google and GitHub providers.
This includes:
- New OAuth routes
- Token validation
- User profile mapping

This change improves security by removing the need to store passwords
locally and provides a better user experience with single sign-on.

Fixes #123
Related to #100
```

**Common commit types in Conventional Commits:**
- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect code meaning (formatting, white-space)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Code change that improves performance
- `test`: Adding missing tests or correcting existing tests
- `build`: Changes that affect the build system or external dependencies
- `ci`: Changes to CI configuration files and scripts
- `chore`: Other changes that don't modify src or test files

**Benefits of structured commit messages:**
- **Automated versioning**: Tools can determine version bumps automatically
- **Automatic changelog generation**: Categorize changes by type
- **Filtering history**: Find all bug fixes or features easily
- **Better collaboration**: Make code reviews and navigation easier
- **Clear history**: Understand why changes were made years later

**Commit message tools:**
- **commitlint**: Enforces commit message conventions
- **commitizen**: Interactive commit message generation
- **standard-version**: Release management using commit messages

**Setting up commit templates:**
```bash
# Create a template file
cat << 'EOF' > ~/.gitmessage
# <type>(<scope>): <short summary>
# |<---- Using a maximum of 50 characters ---->|
#
# Explain why this change is being made
# |<---- Try to limit each line to 72 characters ---->|
#
# Provide links or keys to any relevant tickets, articles or other resources
# Example: Fixes #123, Relates to #456, Implements #789
#
# --- COMMIT END ---
EOF

# Configure Git to use it
git config --global commit.template ~/.gitmessage
```

**Scenario:** Following conventional commits in a team project:

```bash
# Adding a feature
git commit -m "feat(user): add password reset functionality

Implement a secure password reset flow including:
- Reset request form
- Secure token generation
- Email notifications
- Password change form with validation

This implements our security requirement to provide users with
a self-service way to recover access to their accounts.

Closes #45"
```

**Enforcing commit message standards with a Git hook:**
```bash
#!/bin/bash
# .git/hooks/commit-msg

commit_msg=$(cat "$1")
pattern="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore)(\(.+\))?: .{1,50}$"

if ! [[ $(head -1 "$1") =~ $pattern ]]; then
  echo "ERROR: Invalid commit message format."
  echo "Must match: type(scope): description"
  exit 1
fi
```

### Workflow Patterns

Git workflows are structured approaches to using Git for collaboration. Different workflows suit different team sizes, release frequencies, and project complexities. Understanding these patterns helps teams establish consistent practices.

#### GitFlow

GitFlow is a branching model designed for projects with scheduled releases. It defines specific branch roles and how they interact.

**Branch structure:**
- `main`: Production-ready code, always stable
- `develop`: Latest development changes, integration branch
- `feature/*`: New features (branch from develop, merge back to develop)
- `release/*`: Release preparation (branch from develop, merge to main and develop)
- `hotfix/*`: Production emergency fixes (branch from main, merge to main and develop)

**Visualization of GitFlow:**
```
    main       ----------o----------o-----------------o------>
                          \          \               /
    release    -----------\----------\-------------o---------->
                           \          \           /
    develop    ---o----o---o---o------o---------o---o-------->
                  \      /     \              /     \
    feature       -o----o      --o----o-----o       -o----o-->
                                 \                   
    hotfix                       -o---o---------------o------>
                                      \               \
                                       \               \
                                        ∙               ∙
                                    merge back      merge back
                                   to main/dev     to main/dev
```

**Strengths:**
- Well-defined roles for branches
- Clear separation between in-progress work and stable code
- Good for teams with scheduled release cycles
- Handles hotfixes cleanly

**Weaknesses:**
- Complex for small teams or continuous delivery
- Can lead to long-lived branches and difficult merges
- Overhead for simple projects

**GitFlow example:**
```bash
# Initialize GitFlow
git flow init

# Start a new feature
git flow feature start user-authentication

# Make changes, commit
git add .
git commit -m "Implement user signup"

# Complete feature (merges to develop)
git flow feature finish user-authentication

# Start a release
git flow release start v1.0.0

# Make release preparations
git add .
git commit -m "Update version numbers"

# Complete release (merges to main and develop, creates tag)
git flow release finish v1.0.0

# Push changes
git push origin develop
git push origin main --tags
```

#### GitHub Flow

GitHub Flow is a simpler workflow focused on continuous delivery, centered around the main branch and pull requests.

**Branch structure:**
- `main`: Always deployable, represents production
- Feature branches: Created for all changes (named descriptively)

**Process:**
1. Create a branch from main
2. Make changes and commit
3. Open a pull request
4. Discussion and review
5. Deploy and test (optional)
6. Merge to main

**Visualization of GitHub Flow:**
```
                             PR & Review
                             ↓ 
    main       ---o----------o-----------o------o----------->
                   \                     /      \
    features       -o----o----o---------o        -o----o--->
                    
                    New       Collaborate      New feature
                  feature    & make changes
```

**Strengths:**
- Simple to understand and implement
- Great for continuous delivery
- Built around code review
- Low overhead

**Weaknesses:**
- Less structure for releases
- Requires excellent test automation
- May need feature flags for partial features

**GitHub Flow example:**
```bash
# Start from latest main
git checkout main
git pull

# Create a feature branch
git checkout -b feature/payment-integration

# Make changes, commit often
git add .
git commit -m "Add Stripe API client"

# Push branch to share
git push -u origin feature/payment-integration

# Create pull request (on GitHub)
# Review process happens...

# After approval, merge (can be done on GitHub)
git checkout main
git merge feature/payment-integration
git push
```

#### Trunk-Based Development

Trunk-Based Development centers on keeping everyone integrating to a single branch (trunk) frequently, typically at least daily.

**Branch structure:**
- `main` (or trunk): Primary integration branch
- Short-lived feature branches (1-2 days max)
- Release branches (optional): For release preparation only

**Key practices:**
- Developers integrate to main at least daily
- Feature flags for incomplete functionality
- Excellent automated testing
- Small, incremental changes

**Visualization of Trunk-Based Development:**
```
    main       --o---o---o---o---o---o---o---o---o---o----->
                  \   \       \   \               \
    features      -o   -o---o--o   -o---o         -o---o-->
                  (1d)   (2d)  (1d)   (2d)         (1d)
```

**Strengths:**
- Minimizes merge conflicts
- Continuous integration in truest form
- Reduces integration debt
- Great for continuous delivery

**Weaknesses:**
- Requires excellent testing practices
- Needs feature toggles for incomplete work
- Cultural shift for some teams

**Scenario:** Team adopting Trunk-Based Development:
```bash
# Start the day by pulling latest
git checkout main
git pull

# Create a small feature branch
git checkout -b feature/add-search-filter

# Make small, focused changes
git add .
git commit -m "Add basic search filter UI"

# Pull latest main before integrating
git checkout main
git pull

# Integrate your changes
git checkout feature/add-search-filter
git rebase main
git checkout main
git merge feature/add-search-filter
git push

# Delete the short-lived branch
git branch -d feature/add-search-filter
```

#### Choosing the Right Workflow

**Consider these factors:**
- Team size and distribution
- Release frequency and strategy
- Project complexity
- CI/CD infrastructure
- Team experience with Git

**Decision matrix:**
- **GitFlow**: Larger teams, scheduled releases, complex projects
- **GitHub Flow**: Small-medium teams, continuous delivery, web applications
- **Trunk-Based**: CI/CD focused teams, microservices, experienced teams

**Hybrid approaches:**
Many teams adapt these patterns to their specific needs, creating hybrid workflows that combine elements from different approaches.

# Complete feature
git checkout develop
git pull
git merge --no-ff feature/user-profiles
git push origin develop

# Prepare release
git checkout -b release/1.2.0 develop
# Version bumps, final fixes
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Version 1.2.0 - Major update with new features"
git checkout develop
git merge --no-ff release/1.2.0
git branch -d release/1.2.0
```

### Code Review

Code review is a critical quality control process where team members examine each other's code changes before they're merged into the main codebase. Git and GitHub provide tools that streamline the code review process.

#### Pull Request/Merge Request Workflow

Pull Requests (GitHub) or Merge Requests (GitLab) serve as the central hub for code review, providing a structured way to discuss and review changes.

**Anatomy of a good pull request:**
1. **Clear title**: Concisely describes the change
2. **Detailed description**: Explains what, why, and how
3. **References**: Links to related issues or documentation
4. **Test results**: Evidence the code works as expected
5. **Screenshots/videos**: For visual changes
6. **Checklist**: Items to verify before merging

**Creating a pull request:**
```bash
# Make sure your branch has all changes committed
git add .
git commit -m "Complete feature implementation"

# Push to GitHub
git push origin feature/user-profiles

# Create PR through GitHub UI or GitHub CLI
gh pr create --title "Implement user profiles" --body "This PR adds user profile pages with avatars and bios."
```

#### Preparing Code for Review

**Before requesting review:**
```bash
# Update from the target branch to avoid conflicts
git fetch origin
git rebase origin/main

# Ensure tests pass
npm test

# Check linting and formatting
npm run lint
npm run format

# Review your own changes first
git diff main..HEAD

# Push to GitHub
git push origin feature/login

# For an existing PR that needed force-push after rebasing
git push --force-with-lease origin feature/login
```

**Making your PR reviewer-friendly:**
- Break large changes into smaller, logical commits
- Use descriptive commit messages
- Include tests that verify the changes
- Add comments to explain complex parts
- Clean up debug code and comments

#### Effective Code Review Techniques

**For authors:**
- Keep changes focused on a single purpose
- Respond to all feedback (even with just acknowledgment)
- Don't take criticism personally
- Consider alternatives suggested by reviewers
- Use the opportunity to learn and improve

**For reviewers:**
- Be specific and constructive
- Provide the reasoning behind suggestions
- Distinguish between "must fix" and "nice to have"
- Ask questions instead of making accusations
- Use GitHub's suggestion feature for simple fixes
- Approve once requirements are met

**GitHub review features:**
- Line comments for specific feedback
- Suggestions for proposing exact changes
- Review summaries (Approve, Request Changes, Comment)
- Required reviewers and approval gates
- Status checks for automation
- Auto-assignment for code owners

#### Handling Code Review Feedback

**Scenario:** Addressing code review feedback:

```bash
# After receiving review feedback:
git checkout feature/login

# Make the requested changes
# ...editing files...

# Commit the changes 
git add .
git commit -m "Address code review feedback"

# Consider using fixup for cleaner history
git commit --fixup=[original-commit-hash]

# Push the updates
git push origin feature/login

# Respond to comments in GitHub UI
```

**When to use fixup vs. new commits:**
- **Fixup**: For corrections to existing work (typos, bugs, requested changes)
- **New commits**: For new functionality or significant changes requested in review

**Squash before merging:**
Many teams configure GitHub to squash commits when merging a PR to maintain a clean main branch history.

#### Continuous Integration in Code Review

Integrating automated checks provides objective feedback:
- Unit and integration tests
- Code coverage reports
- Static analysis and linting
- Security scanning
- Performance benchmarking

**GitHub Actions example:**
```yaml
name: PR Checks

on:
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
    - run: npm ci
    - run: npm test
    - run: npm run lint
```

#### Code Review Culture

Effective code review depends on team culture:
- Normalize code review for everyone (including senior engineers)
- Set clear expectations for turnaround time
- Value both technical correctness and knowledge sharing
- Create a psychologically safe environment
- Recognize thorough reviews as valuable contributions
- Use pair programming for complex changes

**Typical review workflow:**
1. Author creates PR with proper description
2. CI system runs automated checks
3. Reviewers provide feedback
4. Author addresses feedback
5. Reviewers approve
6. PR is merged (manually or automatically)
7. Branch is deleted
# Make requested changes
git add updated-files.js
git commit -m "Address feedback: Fix error handling"
git push origin feature/login

# If you need to make additional changes:
git add src/components/Login.js
git commit --amend --no-edit  # If minor change to previous commit
git push --force-with-lease origin feature/login
```

## Troubleshooting

Even experienced developers encounter Git issues. Understanding how to diagnose and resolve common problems saves time and prevents data loss.

### Recovering Lost Work

Git's design makes it difficult to truly lose work once it's been committed, and even uncommitted changes can often be recovered.

**Understanding Git's safety mechanisms:**
- Every commit is stored in the Git database until garbage collection runs
- Reflog tracks reference changes for 90 days by default
- Stashes are preserved until explicitly deleted
- Working directory changes may be recoverable with `git fsck`

**Recover uncommitted changes:**
```bash
# Check for stashed changes
git stash list

# Check if changes were auto-stashed during a failed operation
git stash list | grep "autostash"

# Use fsck to find dangling blobs (uncommitted content)
git fsck --lost-found
```

**Recover deleted commits:**
```bash
# Use reflog to see HEAD movement history
git reflog
# Example output:
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: Add important feature
# ...

# Reset to the commit before the deletion
git reset --hard def5678
```

**Recover deleted branch:**
```bash
# Find the last commit of the deleted branch
git reflog | grep "deleted-branch"
# or check for checkout operations
git reflog | grep "checkout: moving from deleted-branch"

# Recreate branch at that commit
git checkout -b restored-branch commit-hash
```

**Recovering during an interrupted workflow:**
```bash
# After a crash, Git leaves behind state files
# Look for instructions in:
ls -la .git/

# For interrupted merges
cat .git/MERGE_HEAD  # The commit being merged
git merge --abort  # Cancel the merge
git merge --continue  # Resume the merge

# For interrupted rebases
cat .git/rebase-merge/head-name  # Branch being rebased
git rebase --abort  # Cancel the rebase
git rebase --continue  # Resume the rebase
```

**Scenario:** You accidentally deleted a branch with important work:

```bash
# Find the lost commit
git reflog
# Example output:
# abc1234 HEAD@{5}: checkout: moving from deleted-branch to main

# Restore the branch
git checkout -b restored-branch abc1234
```

### Fixing Mistakes

Git offers several ways to correct mistakes, depending on whether the changes have been committed, staged, or pushed.

**For uncommitted changes:**
```bash
# Discard all uncommitted changes
git restore .  # Git 2.23+
git checkout -- .  # Older Git versions

# Discard changes for specific files
git restore file1.js file2.js  # Git 2.23+
git checkout -- file1.js file2.js  # Older Git versions

# Unstage files (but keep changes)
git restore --staged file1.js  # Git 2.23+
git reset HEAD file1.js  # Older Git versions
```

**For committed but not pushed changes:**
```bash
# Undo the last commit (keep changes staged)
git reset --soft HEAD~1

# Modify the last commit message
git commit --amend -m "New message"

# Add forgotten files to the last commit
git add forgotten-file.js
git commit --amend --no-edit

# Discard the last commit completely (lose changes)
git reset --hard HEAD~1

# Discard multiple commits
git reset --hard HEAD~3
```

**For pushed commits:**
```bash
# Undo a commit by creating a new commit that reverses changes
git revert commit-hash

# Revert multiple commits
git revert older-hash..newer-hash

# Force push after rewriting history (caution!)
git push --force-with-lease origin branch-name
```

**Understanding different reset modes:**
- `--soft`: Moves HEAD but preserves staged changes and working directory
- `--mixed` (default): Moves HEAD and unstages changes, preserves working directory
- `--hard`: Moves HEAD, unstages changes, and resets working directory (destructive)

**Scenario:** You committed sensitive information like an API key:

```bash
# Remove the sensitive file from git tracking but keep it locally
git rm --cached config.js
echo "config.js" >> .gitignore
git add .gitignore
git commit -m "Remove sensitive file from repo and add to .gitignore"

# If the sensitive data was already pushed
# 1. Change your API keys immediately
# 2. Use BFG Repo Cleaner for large binaries or passwords
git clone --mirror git://example.com/repo.git
bfg --replace-text passwords.txt repo.git
cd repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force

# Alternative: git-filter-repo (modern) or git-filter-branch (legacy)
git filter-repo --path config.js --invert-paths
```

### Common Errors

Understanding and resolving common Git error messages saves time and frustration.

**"Your local changes would be overwritten by checkout/merge":**
```bash
# Option 1: Stash your changes
git stash
git checkout branch-name
git stash pop

# Option 2: Commit your changes
git commit -m "WIP: Save changes before switching"

# Option 3: Discard your changes (if unwanted)
git restore .  # Git 2.23+
git checkout -- .  # Older Git
```

**"Failed to push some refs":**
```bash
# The remote has new changes you don't have locally
git fetch origin
git merge origin/branch-name
# or
git pull --rebase origin branch-name
git push origin branch-name
```

**"Pathspec is in submodule":**
```bash
# Initialize and update submodules
git submodule update --init --recursive

# Then try your operation again
```

**"Merge conflict in [file]":**
```bash
# Identify all conflicted files
git status

# Open the files and resolve markers
# <<<<<<< HEAD
# your changes
# =======
# incoming changes
# >>>>>>> branch-name

# After resolving
git add resolved-file.js
git commit  # or git rebase --continue / git merge --continue
```

**"detached HEAD" state:**
```bash
# Check where you are
git log --oneline -1

# Create a branch to save your work
git branch temp-branch

# Go back to a named branch
git checkout main
```

**"Cannot rebase onto multiple branches":**
```bash
# Be more specific about the branch to rebase onto
git rebase origin/main main
```

**"*** Please tell me who you are":**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Scenario:** Resolving merge conflicts during a rebase:

```bash
git checkout feature/login
git rebase main

# CONFLICT: Merge conflict in src/auth.js
# After fixing conflicts:
git add src/auth.js
git rebase --continue

# If you want to abort instead:
git rebase --abort
```

### Diagnosing Problems

For more complex issues, Git provides diagnostic tools:

```bash
# Get verbose output for any command
git checkout -v branch-name

# Debug remote operations
GIT_TRACE=1 git pull origin main

# Check repository integrity
git fsck

# View detailed object information
git cat-file -p commit-hash

# Check the git log
git log --graph --oneline --all
```

### Preventative Measures

**Avoiding common pitfalls:**
1. Make small, focused commits
2. Pull/fetch before starting work
3. Use separate branches for separate features
4. Write descriptive commit messages
5. Use `git status` frequently
6. Keep `.gitignore` updated
7. Be careful with force push
8. Back up important work

**Setting up helpful aliases:**
```bash
git config --global alias.hist "log --graph --decorate --oneline --all"
git config --global alias.oops "commit --amend --no-edit"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD --stat"
```

```bash
git checkout feature/login
git rebase main
# CONFLICT: Merge conflict in src/auth.js

# Edit src/auth.js to fix conflicts
git add src/auth.js
git rebase --continue

# If you want to abort the rebase
git rebase --abort
```

---

This guide covers Git and GitHub from basics to advanced topics, with practical scenarios to illustrate when and how to use different commands. Remember that while Git is powerful, it's also complex, so always think before executing commands, especially ones that modify history or force-push changes.

For more information, refer to:
- [Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com/en)
- [Pro Git Book](https://git-scm.com/book/en/v2)
