Today's Topic: Git Rebase

Yesterday we learned Git Merge. Today we're learning another important way to integrate changes between branches:

git rebase

Rebase is especially useful when you want to keep Git history clean and linear.

1. What is Git Rebase?

Suppose your history looks like this:

A---B---C  main
     \
      D---E  feature

While you were working on the feature, main continued moving forward.

With a merge, you might get:

A---B---C-------M  main
     \         /
      D---E---/

With rebase, Git takes your feature commits and replays them on top of the latest main:

A---B---C---D'---E'  feature

The result is a cleaner, linear history.

2. Merge vs Rebase

This is one of the most important concepts to understand.

Git Merge
git switch main
git merge feature

History can look like:

A---B---C-------M
     \         /
      D---E---/
Git Rebase
git switch feature
git rebase main

History becomes:

A---B---C---D'---E'
Simple difference
Git Merge	Git Rebase
Combines histories	Replays commits
Can create merge commit	Usually creates linear history
Preserves original branch structure	Rewrites commit history
Safer for shared branches	Best used carefully on private branches
🧠 3. What Actually Happens During Rebase?

Imagine:

main:     A---B---C
               \
feature:         D---E

Git does roughly this:

1. Find commits D and E
2. Temporarily remove them
3. Move feature to C
4. Replay D
5. Replay E

Result:

A---B---C---D'---E'

Notice the '.

That's because the commits are recreated with new commit IDs.

This is why rebase is considered a history-rewriting operation.

🧪 4. Hands-On Lab — Your First Rebase

We'll create a clean repository specifically for today's lesson.

Step 1 — Create the lab directory

Linux/macOS:

mkdir -p ~/100-days-devops/day25-git-rebase
cd ~/100-days-devops/day25-git-rebase

PowerShell:

mkdir "$HOME\100-days-devops\day25-git-rebase"
cd "$HOME\100-days-devops\day25-git-rebase"
Step 2 — Initialize Git
git init

Check:

git status

If Git created master instead of main, rename it:

git branch -M main
📝 5. Create the Initial Commit

Linux/macOS:

echo "# DevOps Application" > README.md

PowerShell:

"# DevOps Application" | Out-File README.md

Then:

git add README.md
git commit -m "Initial commit"
🌿 6. Create a Feature Branch
git switch -c feature-monitoring

Verify:

git branch

Expected:

* feature-monitoring
  main
✏️ 7. Make a Feature Commit

Linux/macOS:

echo "Monitoring configuration" >> README.md

PowerShell:

"Monitoring configuration" | Add-Content README.md

Commit it:

git add README.md
git commit -m "Add monitoring configuration"
🔀 8. Move Back to Main
git switch main

Now make a change on main.

Linux/macOS:

echo "Production environment" >> README.md

PowerShell:

"Production environment" | Add-Content README.md

Commit:

git add README.md
git commit -m "Add production environment"

Now our history looks approximately like:

A---B  main
 \
  C    feature-monitoring

More precisely:

A
├── B  main
└── C  feature-monitoring

Both branches have progressed independently.

🔄 9. Rebase the Feature Branch

Switch to the feature branch:

git switch feature-monitoring

Now run:

git rebase main

Git should replay the feature commit on top of the latest main.

The history now becomes:

A---B---C'
         ^
         feature-monitoring

The original feature commit was recreated as a new commit.

📊 10. Visualize the Result

Run:

git log --oneline --graph --all

You should see a linear history, similar to:

* 91abcde Add monitoring configuration
* 72def45 Add production environment
* 34abc12 Initial commit

This is one of the main reasons developers use rebase.

🔍 11. Check the Difference Between Branches

Run:

git status

Then:

git branch -vv

You can also inspect the complete history:

git log --oneline --decorate --graph --all

This is an excellent command to understand what Git actually did.

⚔️ 12. Rebase Conflict

Just like merge, rebase can produce conflicts.

For example:

main:
Environment = Production

Feature branch:

Environment = Development

If both branches modify the same line, Git may stop and report:

CONFLICT (content): Merge conflict

Check:

git status
Resolve the Conflict

Open the conflicted file.

You may see:

<<<<<<< HEAD
Environment = Production
=======
Environment = Development
>>>>>>> feature-monitoring

Choose the correct final content.

For example:

Environment = Production
Monitoring = Enabled

Then stage it:

git add README.md

Continue the rebase:

git rebase --continue

Git may open an editor for the commit message. Keep the existing message and save/close the editor.

🛑 13. Abort a Rebase

If something goes wrong and you want to return to the state before the rebase:

git rebase --abort

This is a very useful recovery command.

🔧 14. Common Troubleshooting
CONFLICT appears

Run:

git status

Resolve the conflicted files, then:

git add .
git rebase --continue
I want to cancel the rebase
git rebase --abort
Git says the branch is already up to date

Check:

git log --oneline --graph --all

Your feature branch may already contain the latest main commits.

Rebase opened an editor

Git may ask you to confirm a commit message.

If using Vim:

Esc
:wq
Enter

If using another editor, save the file and close it.

⚠️ 15. Important Rebase Rule

This is probably the most important rule from today's lesson:

Avoid rebasing commits that other people are already working on.

Why?

Because rebase rewrites commit history.

For example:

Before:

A---B---C
     \
      D---E

After rebase:

A---B---C---D'---E'

D and E have effectively been replaced by D' and E'.

If another developer already pulled the original commits, this can create unnecessary problems.

Good use:
Your private feature branch
        ↓
      rebase
        ↓
latest main
        ↓
Pull Request
Be careful with:
Shared production branch
        ↓
      rebase
🛠️ 16. Useful Rebase Commands
Command	Purpose
git rebase main	Rebase current branch onto main
git rebase --continue	Continue after resolving conflict
git rebase --abort	Cancel rebase
git rebase --skip	Skip the current commit
git log --graph --oneline --all	Visualize history
🎯 17. A Real DevOps Workflow

A simplified development workflow could look like:

             main
              │
              ▼
       Create feature branch
              │
              ▼
         Write code
              │
              ▼
           Commit
              │
              ▼
       Rebase with main
              │
              ▼
        Resolve conflicts
              │
              ▼
         Push / PR
              │
              ▼
       Code Review + CI
              │
              ▼
           Merge
              │
              ▼
          Deployment

Rebase is therefore not just a Git command—it becomes useful when maintaining a clean branch before code review and integration.

📸 18. What Screenshot Should You Capture?

For today's LinkedIn post, capture:

git log --oneline --graph --decorate --all

and:

git status

Ideally your screenshot should show something similar to:

* abc1234 Add monitoring configuration
* def5678 Add production environment
* ghi9012 Initial commit

On branch feature-monitoring
nothing to commit, working tree clean

This visually demonstrates the linear history created through rebase.

🧠 19. Day 25 Key Takeaways

Today you learned:

✅ What Git rebase is
✅ How rebase differs from merge
✅ How commits are replayed
✅ Why commit IDs change after rebase
✅ How to handle rebase conflicts
✅ git rebase --continue
✅ git rebase --abort
✅ When rebase is useful
✅ Why rebasing shared branches can be dangerous

The simplest way to remember it:

MERGE

Branch A ────────┐
                 ├── Merge Commit
Branch B ────────┘


REBASE

Branch B
   │
   ▼
Move on top of latest A
   │
   ▼
Clean linear history
💼 Day 25/100 — LinkedIn Post

Day 25/100 — Git Rebase 🔄

Yesterday I learned how git merge brings changes from one branch into another.

Today I learned another approach:

Git Rebase.

At first, merge and rebase looked almost identical to me.

The important difference became clear when I visualized the Git history.

With merge, we can end up with something like:

A---B---C-------M
     \         /
      D---E---/

With rebase:

A---B---C---D'---E'

Instead of creating a merge commit, Git replays the feature commits on top of the latest base.

I practiced this workflow today:

git switch feature-monitoring
git rebase main

git log --oneline --graph --all

I also learned something very important:

Rebase rewrites history.

That means I should be careful when rebasing branches that other developers are already using.

For me, the practical takeaway is:

Merge = combine histories

Rebase = replay commits on a new base

I'm beginning to understand why Git history management becomes so important when working with larger development teams and CI/CD pipelines.

Another day of learning and practicing DevOps. 🚀

Day 25/100 complete.

#100DaysOfDevOps #DevOps #Git #GitHub #VersionControl #CI_CD #SoftwareDevelopment #DevOpsJourney #LearningInPublic #GitRebase

🔜 Tomorrow — Day 26/100

Understanding Pull Requests

We'll move from local Git workflows into collaboration with GitHub and learn how developers propose, review, discuss, and integrate changes through Pull Requests (PRs).
