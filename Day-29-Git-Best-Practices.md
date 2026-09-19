# 100 Days of DevOps — Day 29/100

## Git Best Practices

Knowing Git commands is important, but using Git consistently in a team is what keeps a production repository healthy.

A good Git workflow looks like:

```text
Clean Commits
     ↓
Clear Branches
     ↓
Good Pull Requests
     ↓
Useful History
     ↓
Safer Releases
```

## 1. Meaningful Commit Messages

Avoid:

```bash
git commit -m "changes"
git commit -m "update"
git commit -m "fix"
```

Prefer:

```bash
git commit -m "Add health check endpoint"
git commit -m "Fix database connection timeout"
git commit -m "Update Docker base image"
```

A good commit message should answer:

> What changed?

## 2. Keep Commits Small

Avoid putting unrelated changes into one commit.

Prefer focused commits such as:

```text
Commit 1 → Fix login validation
Commit 2 → Update database configuration
Commit 3 → Update Dockerfile
Commit 4 → Update documentation
```

Small commits are easier to review, debug, revert, cherry-pick, and understand.

## 3. Never Commit Secrets

Never commit:

```text
passwords
API keys
private keys
cloud credentials
access tokens
.env files containing secrets
```

Use environment variables or appropriate secret-management systems such as GitHub Actions Secrets, Azure Key Vault, HashiCorp Vault, or Kubernetes Secrets.

If a credential is accidentally exposed, rotate/revoke it immediately.

## 4. Use `.gitignore`

Create:

```bash
touch .gitignore
```

Example:

```text
.env
*.log
*.tmp
*.pem
node_modules/
.terraform/
__pycache__/
```

Check it:

```bash
cat .gitignore
```

`.gitignore` helps prevent accidental commits of logs, temporary files, credentials, dependencies, local configuration, and generated files.

## 5. Use Feature Branches

Avoid doing development directly on `main`.

```bash
git switch -c feature/health-check
```

Typical workflow:

```text
feature/health-check
        ↓
      commit
        ↓
       push
        ↓
Pull Request
        ↓
      review
        ↓
      merge
        ↓
       main
```

## 6. Keep `main` Stable

A common production workflow is:

```text
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
CI Tests
    ↓
Code Review
    ↓
Approval
    ↓
main
```

Avoid direct pushes to a protected production branch whenever your team's workflow allows it.

## 7. Pull Before Starting Work

A common workflow is:

```bash
git switch main
git pull origin main
git switch -c feature/my-change
```

This starts your work from the latest version of `main`.

## 8. Understand `git pull`

A normal pull integrates fetched remote changes into the current branch.

Another option is:

```bash
git pull --rebase
```

Conceptually:

```text
Before:

A ── B ── C
           D ── E

After rebase:

A ── B ── C ── D' ── E'
```

Use the approach that matches your team's Git workflow.

## 9. Don't Rewrite Shared History Carelessly

Commands such as:

```bash
git rebase
git reset
git push --force
```

can rewrite history.

When a force push is genuinely necessary, prefer:

```bash
git push --force-with-lease
```

over an unconditional `--force`.

Be particularly careful with shared branches.

## 10. Use Pull Requests for Team Changes

A good PR should explain:

### What changed?

```text
Added application health-check endpoint.
```

### Why?

```text
Required for application health monitoring.
```

### How was it tested?

```text
Unit tests passed.
Manual endpoint test completed.
```

## 11. Keep Pull Requests Focused

Large PRs are harder to review.

Whenever practical, keep PRs focused and reasonably small.

```text
Small PR
  ↓
Easier Review
  ↓
Faster Feedback
  ↓
Safer Merge
```

## 12. Review Changes Before Commit

Use:

```bash
git status
git diff
```

After staging:

```bash
git diff --cached
```

Useful workflow:

```text
Working Directory
       ↓
    git diff
       ↓
     Review
       ↓
    git add
       ↓
git diff --cached
       ↓
     Review
       ↓
    commit
```

## 13. Don't Blindly Use `git add .`

Instead of always doing:

```bash
git add .
```

review first:

```bash
git status
```

Then stage the intended files:

```bash
git add README.md
git add .gitignore
```

## 14. Use `git status` Frequently

When unsure about the state of your repository:

```bash
git status
```

It shows useful information about:

- Current branch
- Modified files
- Staged files
- Untracked files

> When unsure, run `git status`.

# Hands-on Lab

## Step 1 — Create the repository

```bash
mkdir git-best-practices-lab
cd git-best-practices-lab
git init
```

## Step 2 — Create a README

```bash
echo "# Git Best Practices Lab" > README.md
git status
```

## Step 3 — Stage and review

```bash
git add README.md
git diff --cached
```

## Step 4 — Make a meaningful commit

```bash
git commit -m "Add Git best practices lab README"
```

Check:

```bash
git log --oneline
```

Expected:

```text
abc1234 Add Git best practices lab README
```

## Step 5 — Create `.gitignore`

```bash
cat > .gitignore <<EOF
.env
*.log
*.tmp
*.pem
node_modules/
.terraform/
EOF
```

Check:

```bash
cat .gitignore
```

## Step 6 — Test `.gitignore`

```bash
touch application.log
touch test.tmp
touch .env
git status
```

The ignored files should not appear as untracked files.

## Step 7 — Create a feature branch

```bash
git switch -c feature/documentation
git branch
```

Expected:

```text
* feature/documentation
  main
```

## Step 8 — Make a change

```bash
echo "## Git Workflow" >> README.md
git diff
```

## Step 9 — Stage intentionally

```bash
git add README.md .gitignore
git diff --cached
```

## Step 10 — Commit

```bash
git commit -m "Document Git workflow best practices"
```

## Step 11 — Review history

```bash
git log --oneline --decorate --graph --all
```

You should see a history similar to:

```text
* 91de321 (HEAD -> feature/documentation)
| Document Git workflow best practices
|
* 32ac789 (main)
  Add Git best practices lab README
```

# Bonus Challenge

Create another feature branch:

```bash
git switch main
git switch -c feature/security
```

Add security documentation:

```bash
echo "## Security" >> README.md
echo "Never commit secrets." >> README.md
```

Review:

```bash
git diff
```

Commit:

```bash
git add README.md
git commit -m "Document Git security best practices"
```

View the history:

```bash
git log --oneline --graph --decorate --all
```

# Expected Workflow

```text
              main
               │
               ├──────────────┐
               │              │
               ▼              ▼
     feature/documentation  feature/security
               │              │
             changes        changes
               │              │
             review         review
               │              │
             commit         commit
               │              │
               └──────┬───────┘
                      ▼
                    PR
                      │
                    Review
                      │
                    Merge
                      │
                      ▼
                    main
```

# Troubleshooting

## `.gitignore` isn't ignoring a file

If the file was already tracked, `.gitignore` will not automatically remove it from tracking.

For example:

```bash
git rm --cached .env
git commit -m "Stop tracking environment file"
```

## Accidentally staged a file

```bash
git restore --staged filename
git status
```

## Accidentally committed a secret

Deleting the file is not enough if the secret exists in Git history.

Immediately rotate/revoke the credential, then remove the secret from history using an appropriate history-rewriting procedure.

## Don't know what changed

Run:

```bash
git status
git diff
git diff --cached
```

## Worried about losing local changes

Check:

```bash
git status
git diff
```

Don't run destructive commands such as:

```bash
git reset --hard
```

until you understand what will be removed.

# Screenshot Guidance for LinkedIn

Capture a clean terminal screenshot containing:

```bash
git status
git log --oneline --graph --decorate --all
```

A good screenshot should demonstrate:

```text
feature/documentation
        ↓
meaningful commits
        ↓
clean Git history
```

You can also capture:

```bash
cat .gitignore
```

A strong screenshot is one showing the branch and readable commit history.

# Day 29 Key Takeaways

Today we learned:

- Write meaningful commit messages
- Keep commits focused
- Use feature branches
- Protect `main`
- Use Pull Requests
- Review changes before committing
- Use `.gitignore`
- Never commit secrets
- Keep PRs manageable
- Pull before starting work
- Be careful when rewriting history
- Prefer `--force-with-lease` over `--force` when force pushing is necessary
- Keep Git history understandable

> **Git is not just version control. It's part of your team's engineering workflow.**

# LinkedIn Post — Day 29/100

🚀 **Day 29/100 — 100 Days of DevOps**

Today I moved from learning individual Git commands to something more important:

**Git Best Practices.**

Knowing Git commands is one thing.

Using Git properly in a real development team is another.

Some practices I worked on today:

✅ Meaningful commit messages  
✅ Small and focused commits  
✅ Feature branches  
✅ Pull Requests  
✅ Reviewing changes before committing  
✅ Using `.gitignore`  
✅ Keeping `main` stable  
✅ Avoiding secrets in Git  
✅ Keeping Pull Requests manageable  
✅ Maintaining a clean Git history

One habit I especially want to keep:

Before committing, don't blindly run:

`git add .`

Instead:

```text
git status
     ↓
git diff
     ↓
git add
     ↓
git diff --cached
     ↓
git commit
```

This gives me a chance to understand exactly what I'm about to commit.

Another important lesson:

**Never commit secrets.**

Passwords, API keys, cloud credentials, private keys and other sensitive information should be handled through proper secret-management mechanisms rather than stored in Git.

I also practiced using `.gitignore` to keep files such as:

`.env`  
`*.log`  
`*.pem`  
`node_modules/`  
`.terraform/`

out of version control.

The bigger picture is becoming clearer:

```text
Clean Commits
      ↓
Feature Branches
      ↓
Pull Requests
      ↓
Code Review
      ↓
CI Checks
      ↓
Stable main
      ↓
Reliable Releases
```

Git isn't just about saving code.

It's about creating a workflow that makes software development safer, more predictable and easier to collaborate on.

**Day 29 complete. 🚀**

Tomorrow: **Build a Git-Based Workflow**

#100DaysOfDevOps #DevOps #Git #GitHub #GitBestPractices #VersionControl #CodeReview #PullRequest #CICD #DevOpsLearning #DevOpsEngineer #GitWorkflow

# Day 30 Teaser

**Build a Git-Based Workflow**

We'll bring everything together:

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
Review
  ↓
CI Checks
  ↓
Merge
  ↓
Tag
  ↓
Release
```

**Day 29 complete. One day at a time. 🚀**
