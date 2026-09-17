# Day 27 — Resolving Git Merge Conflicts

## Topic

**Resolving Merge Conflicts**

A merge conflict happens when Git finds two different changes to the same part of a file and cannot automatically decide which version should be kept.

A conflict is not necessarily a Git failure. It means Git needs a developer to decide what the correct final version should be.

## Minimal Diagram

```text
Feature Branch
      |
      | changes same line
      v
   Commit A
      |
      +--------+
               v
            MERGE
               |
          CONFLICT
               |
               v
       Resolve manually
               |
               v
         git add + commit
               |
               v
             main
```

## What Does Git Put Inside a Conflicted File?

Git may modify the file like this:

```text
<<<<<<< HEAD
Environment = Staging
=======
Environment = Production
>>>>>>> feature-production
```

- `<<<<<<< HEAD` — current branch's version
- `=======` — separator
- `>>>>>>> feature-production` — incoming branch's version

Your job is to decide what the final file should contain.

# Hands-On Lab

We will intentionally create a merge conflict and resolve it.

## Step 1 — Create a Practice Repository

### Linux/macOS

```bash
mkdir -p ~/100-days-devops/day27-merge-conflict
cd ~/100-days-devops/day27-merge-conflict
```

### PowerShell

```powershell
New-Item -ItemType Directory -Force "$HOME\100-days-devops\day27-merge-conflict"
cd "$HOME\100-days-devops\day27-merge-conflict"
```

## Step 2 — Initialize Git

```bash
git init
git branch -M main
git status
```

Expected:

```text
On branch main
No commits yet
```

## Step 3 — Create the Initial File

### Linux/macOS

```bash
echo "Environment = Development" > config.txt
cat config.txt
```

### PowerShell

```powershell
"Environment = Development" | Set-Content config.txt
Get-Content config.txt
```

Expected:

```text
Environment = Development
```

Commit:

```bash
git add config.txt
git commit -m "Add environment configuration"
```

## Step 4 — Create the Feature Branch

```bash
git switch -c feature-production
git branch
```

Expected:

```text
* feature-production
  main
```

## Step 5 — Modify the File on the Feature Branch

### Linux/macOS

```bash
echo "Environment = Production" > config.txt
```

### PowerShell

```powershell
"Environment = Production" | Set-Content config.txt
```

Commit:

```bash
git add config.txt
git commit -m "Configure production environment"
```

## Step 6 — Go Back to Main

```bash
git switch main
```

Make a different change to the same line.

### Linux/macOS

```bash
echo "Environment = Staging" > config.txt
```

### PowerShell

```powershell
"Environment = Staging" | Set-Content config.txt
```

Commit:

```bash
git add config.txt
git commit -m "Configure staging environment"
```

The branches now contain conflicting changes:

```text
             feature-production
                    |
                    v
Initial -------- Production

Initial -------- Staging
                    ^
                    |
                   main
```

## Step 7 — Create the Conflict

Make sure you're on `main`:

```bash
git branch
```

Then:

```bash
git merge feature-production
```

You should see something similar to:

```text
CONFLICT (content): Merge conflict in config.txt
Automatic merge failed; fix conflicts and then commit the result.
```

## Step 8 — Check the Conflict

```bash
git status
```

You should see:

```text
You have unmerged paths.

both modified:   config.txt
```

Inspect the file:

```bash
cat config.txt
```

PowerShell:

```powershell
Get-Content config.txt
```

Expected:

```text
<<<<<<< HEAD
Environment = Staging
=======
Environment = Production
>>>>>>> feature-production
```

## Step 9 — Resolve the Conflict

For this lab, choose the production version.

Edit `config.txt` so it contains only:

```text
Environment = Production
```

Remove all conflict markers:

```text
<<<<<<<
=======
>>>>>>>
```

## Step 10 — Tell Git the Conflict Is Resolved

```bash
git add config.txt
git status
```

Then complete the merge:

```bash
git commit -m "Resolve environment configuration conflict"
```

## Step 11 — Inspect the Final History

```bash
git log --oneline --graph --decorate --all
```

You should see a history similar to:

```text
*   abc1234 (HEAD -> main) Resolve environment configuration conflict
|| * def5678 Configure production environment
* | ghi9012 Configure staging environment
|/
* jkl3456 Add environment configuration
```

The merge conflict has now been successfully resolved.

# How to Cancel a Merge

If you want to cancel an in-progress merge:

```bash
git merge --abort
```

Then:

```bash
git status
```

Git should return the repository to its state before the merge started.

# Complete Conflict-Resolution Workflow

```text
git merge
    |
    v
CONFLICT
    |
    v
git status
    |
    v
Open conflicted file
    |
    v
Understand both changes
    |
    v
Choose the correct version
    |
    v
Remove conflict markers
    |
    v
git add
    |
    v
git commit
```

Commands:

```bash
git merge feature-production
git status

# Fix the file

git add config.txt
git commit -m "Resolve merge conflict"
```

# Merge Conflict vs Rebase Conflict

Conflicts can occur during both operations.

### Merge

```bash
git merge feature
```

After resolving:

```bash
git add .
git commit
```

### Rebase

```bash
git rebase main
```

After resolving:

```bash
git add .
git rebase --continue
```

To abort a rebase:

```bash
git rebase --abort
```

# Merge Conflicts in a Real DevOps Team

```text
Developer
    |
    v
Feature Branch
    |
    v
Pull Request
    |
    v
Code Review
    |
    v
Conflict?
   / \
 Yes  No
  |    |
  v    |
Resolve |
  |    |
  +----+
     |
     v
  CI Checks
     |
     v
   Merge
     |
     v
    main
```

## Troubleshooting

### `You have unmerged paths`

Run:

```bash
git status
```

Resolve each conflicted file and stage it:

```bash
git add <filename>
```

Then:

```bash
git commit
```

### Multiple files have conflicts

```bash
git status
```

You may see:

```text
both modified: app.py
both modified: config.txt
both modified: README.md
```

Resolve each file:

```bash
git add app.py
git add config.txt
git add README.md
```

Then:

```bash
git commit -m "Resolve merge conflicts"
```

### I accidentally started the wrong merge

```bash
git merge --abort
```

### Conflict markers are still in my file

Look for:

```text
<<<<<<<
=======
>>>>>>>
```

Remove them after deciding which content should remain.

Then:

```bash
git add .
git commit
```

# What Screenshot Should You Capture for LinkedIn?

Capture two screenshots.

### Screenshot 1 — Conflict

Show:

```bash
git status
```

and your conflicted `config.txt`.

The important part should be visible:

```text
<<<<<<< HEAD
Environment = Staging
=======
Environment = Production
>>>>>>> feature-production
```

### Screenshot 2 — Successful Resolution

Show:

```bash
git status
git log --oneline --graph --decorate --all
```

The merge commit and clean repository demonstrate that you successfully resolved the conflict.

> Never expose passwords, tokens, private keys, or other credentials in screenshots.

# Day 27 Key Takeaways

Today you learned:

- What a merge conflict is
- Why conflicts happen
- How Git identifies conflicting changes
- How to inspect conflicts
- How to manually resolve them
- How to stage a resolution
- How to complete a merge
- How to abort a merge
- How merge and rebase conflicts differ

The key workflow:

```text
Conflict
   |
   v
Understand
   |
   v
Resolve
   |
   v
git add
   |
   v
git commit
   |
   v
Clean Repository
```

# Day 27/100 — LinkedIn Post

**Day 27/100 — Git Merge Conflicts**

Today I intentionally created a Git merge conflict.

And this was one of those Git concepts that makes much more sense after actually breaking something and fixing it.

I created two branches where the **same line was changed differently**.

When I tried to merge them, Git stopped:

```text
CONFLICT (content): Merge conflict in config.txt
```

Git then showed me both versions:

```text
<<<<<<< HEAD
Environment = Staging
=======
Environment = Production
>>>>>>> feature-production
```

At that point, Git couldn't decide which version was correct.

And that's the important part:

**Git can detect the conflict, but the developer has to decide what the correct final code should be.**

I practiced the complete workflow:

```bash
git merge feature-production

git status

# Resolve the conflicting file

git add config.txt

git commit -m "Resolve merge conflict"
```

I also learned how to safely cancel an unfinished merge:

```bash
git merge --abort
```

The workflow is now much clearer to me:

**Branch → Commit → Pull Request → Review → Conflict Resolution → CI → Merge**

Today's biggest takeaway:

> **Merge conflicts are a normal part of collaborative development.**

The important skill isn't avoiding every conflict.

It's knowing how to **understand, resolve and verify one safely**.

Another practical Git lesson completed.

**Day 27/100 complete.**

#100DaysOfDevOps #DevOps #Git #GitHub #GitMerge #MergeConflicts #VersionControl #CodeReview #CICD #DevOpsJourney #LearningInPublic

## Next Day — Day 28/100

**Git Tags & Releases**

We'll learn how to mark important commits as versions such as `v1.0.0`, understand lightweight vs annotated tags, push tags to GitHub, and connect Git versions with software releases.
