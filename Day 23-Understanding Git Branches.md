Day 23/100 — Understanding Git Branches

Date: September 14, 2026
Previous: Day 22 — Git Basics
Today: Day 23 — Understanding Git Branches
Next: Day 24 — Git Merge

Today we're taking the next step in Git: branches.

Branches are one of the most important concepts you'll use when working with Git in a real development or DevOps team.

🎯 Today's Objectives

By the end of Day 23, you should understand:

What a Git branch is
Why branches are needed
What main represents
How to create a branch
How to switch branches
How to create and switch in one command
How changes behave across branches
How to delete a branch
How branches support team development

The basic idea:

                 main
                  │
                  ▼
              Commit A
                  │
              Commit B
                  │
           ┌──────┴──────┐
           ▼             ▼
      feature-login   feature-ui
           │             │
        Commit C       Commit D
1. What is a Git Branch?

A Git branch is essentially a separate line of development.

Imagine your stable application is on:

main

You want to develop a new feature.

Instead of modifying main directly, you create:

feature-login

Then:

main
 │
 ├── stable code
 │
 └── feature-login
       │
       ├── change 1
       ├── change 2
       └── change 3

You can work on the feature without disturbing the main branch.

2. Why Do We Need Branches?

Imagine a team of five developers.

Without branches:

Developer 1 ─┐
Developer 2 ─┤
Developer 3 ─┼──► main
Developer 4 ─┤
Developer 5 ─┘

Everyone is changing the same code.

That can become difficult very quickly.

With branches:

                 main
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     feature-A feature-B bug-fix
        │         │         │
        ▼         ▼         ▼
     Developer Developer Developer

Each developer or team can work independently.

Later, those changes can be merged into main.

We'll learn Git Merge on Day 24.

3. main Branch

Most modern Git repositories use:

main

as the primary branch.

Think of it as the branch containing the main version of the project.

A simplified workflow might be:

main
 │
 ├── feature/login
 │
 ├── feature/payment
 │
 └── bugfix/database

The exact branching strategy depends on the organization.

4. Creating a Branch

To create a branch:

git branch feature-login

This creates the branch but does not switch to it.

Check your branches:

git branch

Example:

* main
  feature-login

The * indicates the branch you're currently on.

5. Switching Branches

You can switch to the new branch with:

git switch feature-login

Now:

git branch

should show:

  main
* feature-login

The * has moved.

6. Create + Switch in One Command

Instead of:

git branch feature-login
git switch feature-login

you can use:

git switch -c feature-login

This means:

Create the branch and switch to it.

You'll use this command frequently.

🧪 Day 23 Hands-On Lab

Let's create a realistic branching example.

Step 1 — Create today's project
mkdir -p ~/100-days-devops/day23-git-branches
cd ~/100-days-devops/day23-git-branches

Verify:

pwd
Step 2 — Initialize Git
git init
Step 3 — Create the initial application
cat > app.txt <<'EOF'
DevOps Application

Version: 1.0

Status: Stable
EOF

Check:

cat app.txt
Step 4 — Create the first commit
git add app.txt

Then:

git commit -m "Add initial application"
Step 5 — Check your branch
git branch

Depending on your Git configuration, you might see:

* master

or:

* main

For consistency, let's rename the branch to main:

git branch -M main

Check:

git branch

Expected:

* main
Step 6 — Create a feature branch

Run:

git switch -c feature-login

Expected:

Switched to a new branch 'feature-login'

Check:

git branch

You should see:

* feature-login
  main
Step 7 — Make a Change on the Feature Branch

Add login functionality to the example:

cat >> app.txt <<'EOF'

Feature: Login
Status: In Development
EOF

Check:

cat app.txt

You should see:

DevOps Application

Version: 1.0

Status: Stable

Feature: Login
Status: In Development
Step 8 — Commit the Feature
git add app.txt

Then:

git commit -m "Add login feature"
Step 9 — Check the History
git log --oneline --all

You should see something similar to:

abc1234 Add login feature
def5678 Add initial application
Step 10 — Switch Back to Main
git switch main

Now:

cat app.txt

You should see:

DevOps Application

Version: 1.0

Status: Stable

Notice something important:

The login feature isn't present on main.

Why?

Because the login change was committed on:

feature-login

not on:

main

This is exactly why branches are useful.

🔍 Step 11 — Switch Back to the Feature
git switch feature-login

Now:

cat app.txt

You should see:

DevOps Application

Version: 1.0

Status: Stable

Feature: Login
Status: In Development

The feature is back.

🌿 Visualizing What Happened

Our repository now looks like:

                 main
                  │
                  ▼
              Commit A
                  │
                  └──────────────┐
                                 │
                                 ▼
                          feature-login
                                 │
                                 ▼
                              Commit B

Where:

Commit A = Initial application
Commit B = Login feature

main currently points to Commit A.

feature-login points to Commit B.

🧪 Bonus Exercise — Create Another Branch

Switch back to main:

git switch main

Create another branch:

git switch -c feature-monitoring

Create a monitoring note:

echo "Monitoring enabled" >> app.txt

Commit it:

git add app.txt
git commit -m "Add monitoring configuration"

Check:

git log --oneline --all --graph

You should see a graphical representation of your branches.

Something similar to:

* abc1234 Add monitoring configuration
| * def5678 Add login feature
|/
* ghi9012 Add initial application

The exact commit IDs will be different.

🗑️ Step 12 — Delete a Branch

Once a branch has been merged and is no longer needed, it can be deleted.

For demonstration, first switch to main:

git switch main

Then delete the monitoring branch:

git branch -d feature-monitoring

Git may refuse if the branch contains commits that haven't been merged.

That's intentional protection against accidentally losing work.

Forcing deletion is possible with:

git branch -D feature-monitoring

⚠️ Don't use -D casually. It can delete a branch containing unmerged work.

🔑 Important Commands Today
Command	Purpose
git branch	List branches
git branch name	Create branch
git switch name	Switch branch
git switch -c name	Create + switch
git branch -M main	Rename current branch
git branch -d name	Delete merged branch
git log --oneline --all	View all history
git log --oneline --all --graph	Visualize branches
🛠️ Troubleshooting
git switch is not available

Older Git versions may not support git switch.

You can use:

git checkout feature-login

To create and switch:

git checkout -b feature-login
"Your local changes would be overwritten"

Git is protecting your changes.

Check:

git status

If you don't need the changes, you can discard them carefully:

git restore filename

If you want to keep them, commit them or temporarily stash them.

We'll learn more about handling changes as the Git section progresses.

Branch doesn't appear

Run:

git branch

If you're expecting remote branches, use:

git branch -a
Can't delete a branch

Git may report that the branch isn't fully merged.

First inspect:

git log --oneline --all --graph

If you're certain the work isn't needed:

git branch -D branch-name
📸 What Screenshot Should You Capture?

For LinkedIn, today's strongest screenshot is:

git log --oneline --all --graph

Your terminal should show something resembling:

* abc1234 Add monitoring configuration
| * def5678 Add login feature
|/
* ghi9012 Add initial application

Then show:

git branch

with something like:

* main
  feature-login
📌 Why this screenshot?

It visually proves that you actually created separate lines of development.

That's much stronger than simply posting:

git branch
🧠 Day 23 Key Takeaways
Branch = separate line of development
main
 │
 └── feature-login
Create a branch
git branch feature-login
Create + switch
git switch -c feature-login
Switch
git switch main
List branches
git branch
Visualize branches
git log --oneline --all --graph

The biggest concept to remember:

Branches allow you to develop changes independently without immediately changing the main codebase.

💼 LinkedIn Post — Day 23/100
Day 23 LinkedIn Post

🌿 Day 23/100 — Understanding Git Branches

Today I moved from basic Git commands into one of the concepts that makes Git really powerful:

Branches.

Before practicing it, I understood a branch as "a copy of the code."

But today's hands-on exercise helped me understand it differently.

A branch is better thought of as a separate line of development.

I started with:

main
  │
  ▼
Initial Commit

Then created a feature branch:

git switch -c feature-login

I made changes and committed them on that branch.

When I switched back:

git switch main

the feature changes weren't there.

Switching back:

git switch feature-login

brought the changes back.

That simple exercise made the purpose of branching much clearer.

The workflow now looks like:

                 main
                  │
              Commit A
                  │
          ┌───────┴────────┐
          ↓                ↓
   feature-login      feature-monitoring
          ↓                ↓
       Commit B          Commit C

This is powerful in a team environment because developers can work on different features without constantly modifying the main codebase.

Some commands I practiced today:

git branch
git switch -c feature-login
git switch main
git log --oneline --all --graph

I'm also starting to see how Git branches will eventually connect with:

🔀 Pull Requests
🤝 Team collaboration
🔄 CI/CD pipelines
🚀 Automated deployments

The next step is where things get even more interesting:

How do we bring the changes from a feature branch back into the main branch?

That's tomorrow's topic.

Day 23/100 completed. 💪

One more Git concept understood through hands-on practice.

➡️ Next: Day 24 — Git Merge

#100DaysOfDevOps #DevOps #Git #GitHub #GitBranches #VersionControl #DevOpsJourney #LearningInPublic #CICD #Automation #Cloud #TechLearning #100DaysOfCode

🔖 Tomorrow — Day 24
🔀 Git Merge

We'll take the feature-login branch we created today and learn how to merge its changes into main, understand fast-forward merges, and see what happens when two branches contain different changes.
