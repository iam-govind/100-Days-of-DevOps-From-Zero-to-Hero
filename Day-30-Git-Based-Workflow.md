# 100 Days of DevOps — Day 30/100

## Git-Based Workflow

Today is Day 30 and the final day of the Git & GitHub section.

We will combine the concepts from Days 21–29 into one realistic workflow:

```text
Issue / Requirement
        ↓
Feature Branch
        ↓
Write Code
        ↓
Commit
        ↓
Push
        ↓
Pull Request
        ↓
Code Review + CI
        ↓
Approval
        ↓
Merge → main
        ↓
Tag
        ↓
Release
```

## Why Do We Need a Git Workflow?

Without a defined workflow:

```text
Developer → main
Developer → main
Developer → main
```

This can lead to broken builds, difficult reviews, confusing history, accidental changes, and production issues.

A controlled workflow is:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
Review + CI
   ↓
Merge
   ↓
Release
```

## Step 1 — Start With the Latest `main`

```bash
git switch main
git pull origin main
git status
```

Expected:

```text
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

## Step 2 — Create a Feature Branch

```bash
git switch -c feature/add-health-check
git branch
```

Expected:

```text
* feature/add-health-check
  main
```

## Step 3 — Make the Change

```bash
mkdir -p app
echo "Application is healthy" > app/health.txt
cat app/health.txt
```

Expected:

```text
Application is healthy
```

## Step 4 — Review Your Changes

```bash
git status
git diff
```

Stage the file:

```bash
git add app/health.txt
git diff --cached
```

## Step 5 — Create a Meaningful Commit

```bash
git commit -m "Add application health check"
git log --oneline -3
```

## Step 6 — Push the Feature Branch

```bash
git push -u origin feature/add-health-check
```

## Step 7 — Create a Pull Request

On GitHub, create:

```text
base: main
compare: feature/add-health-check
```

Example title:

```text
Add application health check
```

Example description:

```text
## What changed?

Added an application health-check file.

## Why?

Provides a simple health indicator for the application.

## Testing

Verified the health-check file locally.
```

## Step 8 — Code Review

Reviewers can check:

- Is the change required?
- Is the implementation correct?
- Is the code readable?
- Are there security issues?
- Are tests included?
- Does it follow project standards?

Possible review actions:

```text
Approve
Request Changes
Comment
```

## Step 9 — CI Checks

A real Pull Request may trigger:

```text
Pull Request
     │
     ├── Build
     ├── Unit Tests
     ├── Lint
     ├── Security Scan
     └── Code Quality
```

The exact checks depend on the repository's CI configuration.

## Step 10 — Merge the Pull Request

After required reviews and checks pass, merge the PR into `main`.

```text
feature/add-health-check
          ↓
     Pull Request
          ↓
      Approved
          ↓
        main
```

## Step 11 — Update Local `main`

```bash
git switch main
git pull origin main
git log --oneline --decorate -5
```

## Step 12 — Create a Release Tag

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git tag
git show v1.0.0
```

Expected tag:

```text
v1.0.0
```

## Step 13 — Push the Release Tag

```bash
git push origin v1.0.0
```

Or:

```bash
git push origin --tags
```

# Complete Workflow

```text
Issue
  ↓
Feature Branch
  ↓
Code
  ↓
Commit
  ↓
Push
  ↓
Pull Request
  │
  ├── Code Review
  └── CI Checks
          ↓
       Approval
          ↓
        Merge
          ↓
         main
          ↓
        Git Tag
          ↓
        v1.0.0
          ↓
       Release
```

# Day 30 Practical Lab

If you're using your DevOps repository:

### 1. Update `main`

```bash
git switch main
git pull origin main
```

### 2. Create your feature branch

```bash
git switch -c feature/day30-git-workflow
```

### 3. Create a file

```bash
mkdir -p day30
echo "# Git-Based Workflow" > day30/README.md
```

Add content:

```bash
cat >> day30/README.md <<EOF

## Workflow

Feature Branch
→ Commit
→ Push
→ Pull Request
→ Review
→ CI
→ Merge
→ Tag
→ Release
EOF
```

Check:

```bash
cat day30/README.md
```

### 4. Review

```bash
git status
git diff
```

### 5. Commit

```bash
git add day30/README.md
git diff --cached
git commit -m "Add Git-based workflow documentation"
```

### 6. Push

```bash
git push -u origin feature/day30-git-workflow
```

### 7. Create the Pull Request

On GitHub:

```text
feature/day30-git-workflow
              ↓
             main
```

Create the PR, review it, and merge it.

### 8. Update Local `main`

```bash
git switch main
git pull origin main
```

### 9. Create the release tag

```bash
git tag -a v1.0.0 -m "Day 30 Git workflow release"
```

Check:

```bash
git tag
```

### 10. Push the tag

```bash
git push origin v1.0.0
```

# Expected Result

Your repository should have:

```text
main
 │
 └── merged feature
       │
       ▼
     v1.0.0
```

Git history should look similar to:

```text
* abc1234 (HEAD -> main, tag: v1.0.0)
| Add Git-based workflow documentation
|
* def5678 Previous change
|
* 123abcd Initial commit
```

The commit IDs will be different on your machine.

# Troubleshooting

## `main` is behind remote

```bash
git switch main
git pull origin main
```

## Push is rejected

Check:

```bash
git status
git branch
git remote -v
```

If required:

```bash
git pull --rebase origin main
```

Resolve conflicts before pushing.

## Pull Request has conflicts

```bash
git switch main
git pull origin main
git switch feature/day30-git-workflow
git rebase main
```

If conflicts appear:

```bash
git status
```

Fix the files, then:

```bash
git add <resolved-file>
git rebase --continue
```

Cancel if necessary:

```bash
git rebase --abort
```

## Tag already exists

Check:

```bash
git tag
```

Don't blindly overwrite an existing release tag. If appropriate, use a new version:

```bash
git tag -a v1.0.1 -m "Release version 1.0.1"
```

## Tag isn't on GitHub

```bash
git tag
git ls-remote --tags origin
git push origin v1.0.0
```

# Screenshot Guidance for LinkedIn

### Screenshot 1 — Git history

```bash
git log --oneline --decorate --graph --all
```

Capture output showing the branch, merged commit, and tag.

### Screenshot 2 — GitHub Pull Request

Capture the PR page showing:

```text
feature/day30-git-workflow → main
```

with checks/review visible.

### Screenshot 3 — Release/Tag

Show:

```text
v1.0.0
```

on the GitHub tags or Releases page.

### Best single screenshot

A clean GitHub PR screenshot showing the PR successfully merged and its checks completed is ideal.

# Day 30 Key Takeaways

Today we connected:

- Git repository
- Branches
- Commits
- `git status`
- `git diff`
- Push
- Pull Requests
- Code review
- CI checks
- Merge
- Tags
- Releases

The complete workflow:

```text
Branch
  ↓
Code
  ↓
Commit
  ↓
Push
  ↓
Pull Request
  ↓
Review
  ↓
CI
  ↓
Merge
  ↓
Tag
  ↓
Release
```

> **Git is not just a collection of commands. It's a workflow for safely moving code from development toward production.**

# LinkedIn Post — Day 30/100

🚀 **Day 30/100 — 100 Days of DevOps**

Today is a special milestone in my DevOps journey.

**Day 30 — Build a Git-Based Workflow**

For the last few days, I've been learning Git concepts one by one.

Today I connected everything into one practical workflow.

```text
Issue
  ↓
Feature Branch
  ↓
Code
  ↓
Commit
  ↓
Push
  ↓
Pull Request
  ↓
Code Review
  ↓
CI Checks
  ↓
Merge
  ↓
Tag
  ↓
Release
```

I practiced the complete flow:

✅ Updated `main`  
✅ Created a feature branch  
✅ Made a change  
✅ Reviewed the changes with `git diff`  
✅ Created a meaningful commit  
✅ Pushed the branch to GitHub  
✅ Created a Pull Request  
✅ Reviewed the change  
✅ Merged it into `main`  
✅ Created a release tag  
✅ Pushed the tag to GitHub  

What really stood out to me is that Git becomes much more powerful when combined with a proper team workflow.

It's not simply:

```text
git add
git commit
git push
```

It's more like:

```text
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
Review + CI
    ↓
Merge
    ↓
Release
```

And this is where Git starts connecting directly with the rest of DevOps.

Later, we'll take this workflow further with:

→ CI/CD  
→ Docker  
→ Azure  
→ Kubernetes  
→ Terraform  
→ Monitoring  
→ DevSecOps

The Git & GitHub section is now complete.

**30 days down.  
70 days to go. 🚀**

Day 30 complete!

Next up: **Docker — Day 31: Introduction to Containers and Docker**

#100DaysOfDevOps #DevOps #Git #GitHub #GitWorkflow #PullRequest #CICD #ContinuousIntegration #ContinuousDeployment #Docker #DevOpsLearning #DevOpsEngineer

# Day 31 Teaser

## 🐳 Introduction to Containers and Docker

We're leaving the Git & GitHub section and moving into **Docker**.

We'll answer:

- What is a container?
- Why do we need containers?
- What problem does Docker solve?
- VM vs Container
- Docker architecture
- Images vs Containers
- Your first Docker command

The journey changes from:

```text
Source Code
    ↓
Git
    ↓
GitHub
```

to:

```text
Source Code
    ↓
Git
    ↓
Docker Image
    ↓
Container
    ↓
Application
```

**Day 30 complete. 🚀 Git & GitHub section complete.**

**Tomorrow: Docker begins. 🐳**
