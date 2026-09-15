oday's Topic: Git Merge

Yesterday, we learned about Git branches. Today, we take the next step:

How do we bring changes from one branch into another?

That is where git merge comes in.

1. What is Git Merge?

Imagine your project has this structure:

main
  |
  A
  |
  B
  |
  C

You create a feature branch:

main
  |
  A
  |
  B
  |
  C
       \
        D
        |
        E
     feature-login

You finish your work on feature-login.

Now you want those changes in main.

You can merge the branch:

git switch main
git merge feature-login

After merging:

A---B---C---D---E
             ^
            main
            feature-login

Git has combined the changes from the feature branch into main.

2. Why is Git Merge Important in DevOps?

In a real DevOps workflow, developers generally don't make every change directly on main.

A simplified workflow looks like:

main
  |
  +---- feature-login
  |          |
  |       development
  |          |
  |       testing
  |          |
  +-------> merge
             |
            main
             |
           CI/CD
             |
          Deployment

This allows teams to:

Develop features independently
Test changes before merging
Review code
Resolve conflicts
Keep the main branch stable
Trigger CI/CD pipelines after merging
🧪 3. Hands-On Lab — Your First Git Merge

We'll create a small repository.

Step 1 — Create the lab directory

Linux/macOS:

mkdir -p ~/100-days-devops/day24-git-merge
cd ~/100-days-devops/day24-git-merge

PowerShell:

mkdir "$HOME\100-days-devops\day24-git-merge"
cd "$HOME\100-days-devops\day24-git-merge"
Step 2 — Initialize Git
git init

Check:

git status

Expected:

On branch master
No commits yet

If your Git configuration automatically uses main, that's perfectly fine.

Step 3 — Create the initial application file

Linux/macOS:

echo "# DevOps Application" > README.md

PowerShell:

"# DevOps Application" | Out-File README.md

Then:

git add README.md
git commit -m "Initial commit"

Expected:

[main ...] Initial commit
🌿 4. Create a Feature Branch

Create a branch called feature-login:

git switch -c feature-login

Verify:

git branch

You should see something similar to:

* feature-login
  main

The * tells us which branch we're currently on.

✏️ 5. Make a Change

Add a login feature:

Linux/macOS:

echo "Login feature added" >> README.md

PowerShell:

"Login feature added" | Add-Content README.md

Check the change:

git diff

You should see:

+# Login feature added

Now commit it:

git add README.md
git commit -m "Add login feature"
🔀 6. Switch Back to Main
git switch main

Check the README:

cat README.md

PowerShell:

Get-Content README.md

You should see only:

# DevOps Application

The login change isn't visible on main yet.

That's because the feature branch hasn't been merged.

🚀 7. Merge the Feature Branch

Now run:

git merge feature-login

You should see something similar to:

Updating ...
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)

Now check:

cat README.md

You should see:

# DevOps Application
Login feature added

🎉 Your feature branch has been merged into main.

📊 8. Visualize the Git History

Run:

git log --oneline --graph --all

You may see:

* abc1234 Add login feature
* def5678 Initial commit

Because this was a fast-forward merge, Git didn't need to create a separate merge commit.

⚡ 9. What is a Fast-Forward Merge?

Suppose we have:

A---B---C
         \
          D---E

main hasn't changed since the feature branch was created.

When we merge:

A---B---C---D---E

Git simply moves the main pointer forward.

This is called a:

Fast-forward merge

🔀 10. Understanding a True Merge Commit

Now let's create a situation where both branches have changed.

Start from your repository:

git switch main

Create another branch:

git switch -c feature-monitoring

Make a change:

echo "Monitoring enabled" >> README.md

Commit it:

git add README.md
git commit -m "Add monitoring"

Now switch to main:

git switch main

Make a different change:

echo "Production environment" >> README.md

Commit:

git add README.md
git commit -m "Add production environment"

Now both branches have different commits.

Merge:

git merge feature-monitoring

Git may create a merge commit:

       D---E  feature-monitoring
      /     \
A---B---C---M  main

M represents the merge commit.

🧩 11. Merge Conflicts

Sometimes Git cannot automatically combine changes.

For example:

main:

Environment = Development

and:

feature:

Environment = Production

Both branches changed the same part of the same file.

Git may report:

CONFLICT (content): Merge conflict in README.md
Automatic merge failed

Don't panic. This is normal in team development.

Step 1 — Check the status
git status

Git will tell you which files have conflicts.

Step 2 — Open the conflicted file

You may see:

<<<<<<< HEAD
Production environment
=======
Development environment
>>>>>>> feature-monitoring

Meaning:

<<<<<<< HEAD

starts the current branch's version.

=======

separates the two versions.

>>>>>>> feature-monitoring

shows the incoming branch's version.

Step 3 — Decide what the final content should be

For example:

Production environment
Monitoring enabled

Remove the conflict markers.

Step 4 — Stage the resolved file
git add README.md
Step 5 — Complete the merge
git commit

Or:

git commit -m "Resolve merge conflict"
🛑 12. How to Cancel a Merge

If you realize you don't want to continue:

git merge --abort

This attempts to return the repository to the state it was in before the merge started.

🧠 13. Useful Git Merge Commands
Command	Purpose
git merge branch-name	Merge a branch
git merge --no-ff branch-name	Force a merge commit
git status	Check merge/conflict status
git log --oneline --graph --all	Visualize history
git merge --abort	Cancel an in-progress merge
🔧 14. Troubleshooting
Problem: Already up to date

Example:

Already up to date.

This means Git doesn't see any new changes to merge.

Check:

git status
git log --oneline --graph --all
Problem: Wrong branch

Check:

git branch

Switch to the branch that should receive the changes:

git switch main

Then:

git merge feature-login
Problem: Merge conflict

Run:

git status

Resolve the conflicted files, then:

git add .
git commit
Problem: I want to cancel the merge

Run:

git merge --abort
📸 15. What Screenshot Should You Capture?

For today's LinkedIn post, I recommend capturing one terminal screenshot showing:

git branch
git log --oneline --graph --all
git status

A strong screenshot would show something like:

* main
  feature-login

* abc1234 Add login feature
* def5678 Initial commit

On branch main
nothing to commit, working tree clean

This demonstrates that you actually performed the merge rather than simply explaining it.

🎯 16. What You Learned Today

By the end of Day 24, you should understand:

What git merge does
How to merge a feature branch
Fast-forward merges
Merge commits
Merge conflicts
Conflict resolution
git merge --abort
Why merging is important in DevOps workflows

The key workflow to remember:

Create Branch
     ↓
Develop
     ↓
Commit
     ↓
Switch to main
     ↓
git merge feature
     ↓
Test
     ↓
CI/CD
💼 Day 24/100 — LinkedIn Post

Here’s a natural learning-in-public version you can post:

Day 24/100 — Git Merge 🔀

Yesterday I learned how Git branches allow us to work on different features independently.

Today I learned the next important step:

How do we bring those changes back into the main branch?

That's where git merge comes in.

I practiced a complete workflow today:

main
  ↓
create feature branch
  ↓
make changes
  ↓
commit
  ↓
switch to main
  ↓
git merge feature-login

I also learned that Git can perform different types of merges.

🔹 Fast-forward merge — when the main branch hasn't changed since the feature branch was created.

🔹 Merge commit — when both branches have developed independently and Git needs to combine their histories.

And probably one of the most important things for real-world teamwork:

Merge conflicts are normal.

When two branches modify the same part of a file, Git may not know which version to keep. We then need to review the changes, resolve the conflict, stage the file, and complete the merge.

Some commands I practiced today:

git switch main
git merge feature-login

git log --oneline --graph --all

git status

git merge --abort

I'm starting to see how Git fits into a real DevOps workflow:

Develop → Branch → Commit → Merge → Test → CI/CD → Deploy

Still learning, still practicing, and continuing one day at a time. 🚀

Day 24/100 complete.

#100DaysOfDevOps #DevOps #Git #GitHub #VersionControl #SoftwareDevelopment #CI_CD #DevOpsJourney #LearningInPublic

🔜 Tomorrow — Day 25

Git Rebase

We'll learn how git rebase works, how it differs from git merge, and why rebase can create a cleaner project history.
