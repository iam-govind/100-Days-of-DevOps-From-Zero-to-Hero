Day 22/100 — Git Basics: init, add, commit and status

Date: September 13, 2026
Yesterday: Day 21 — Introduction to Git and GitHub
Today: Day 22 — Git Basics
Tomorrow: Day 23 — Understanding Git Branches

Today we'll go deeper into the four fundamental Git commands:

git init
git status
git add
git commit

These commands form the foundation of almost every Git workflow.

1. Understanding the Git Workflow

Before running commands, understand this flow:

             Git Workflow

       Working Directory
              │
              │ git add
              ▼
        Staging Area
              │
              │ git commit
              ▼
       Local Repository

Think of it like preparing a package:

Working Directory

You make changes to your files.

Staging Area

You select which changes should go into the next commit.

Commit

You permanently record those staged changes in Git history.

2. git init

git init initializes a directory as a Git repository.

Example:

git init

Git creates a hidden directory:

.git/

This directory contains Git's internal information and history.

Important

Don't manually modify .git.

Git manages it for you.

3. git status

This is probably one of the most useful Git commands.

git status

It tells you:

Current branch
Modified files
Untracked files
Staged files
Changes waiting to be committed

A good Git habit is:

Run git status frequently.

4. git add

Suppose you create:

app.py
README.md
config.txt

You don't necessarily want all three in your next commit.

You can stage only one:

git add app.py

Or stage everything:

git add .

The important concept is:

Working Directory
       ↓
   git add
       ↓
Staging Area
5. git commit

A commit creates a snapshot of the staged changes.

git commit -m "Add application"

The -m specifies the commit message.

A good commit message should describe the change.

Good
Add login functionality
Fix database connection
Update deployment configuration
Poor
changes
update
test
asdf
🧪 Day 22 Hands-On Lab

We'll create a small Git project and deliberately move files through the Git workflow.

Step 1 — Create today's project
mkdir -p ~/100-days-devops/day22-git-basics
cd ~/100-days-devops/day22-git-basics

Verify:

pwd
Step 2 — Initialize Git
git init

Expected:

Initialized empty Git repository in ...

Now:

ls -la

You should see:

.git
Step 3 — Check Git status
git status

You should see something similar to:

On branch main

No commits yet

nothing to commit

Your default branch name may be master depending on your Git configuration.

Step 4 — Create a README
cat > README.md <<'EOF'
# Day 22 - Git Basics

Today I am learning:

- git init
- git status
- git add
- git commit
EOF

Check it:

cat README.md
Step 5 — Check status again
git status

Now Git should show:

Untracked files:

    README.md
What does "untracked" mean?

Git sees the file, but Git isn't tracking it yet.

Step 6 — Stage the file

Run:

git add README.md

Now:

git status

The output should show something similar to:

Changes to be committed:

    new file: README.md

The file has moved from:

Working Directory
       ↓
Staging Area
Step 7 — Commit the file
git commit -m "Add Day 22 Git basics README"

Expected output will be similar to:

[main abc1234] Add Day 22 Git basics README
 1 file changed

Congratulations—you've created your first Day 22 commit. 🎉

Step 8 — Check status

Run:

git status

You should now see:

nothing to commit, working tree clean

This is an important Git state.

It means:

Your working directory has no changes waiting to be committed.

Step 9 — Make a change

Add another section:

cat >> README.md <<'EOF'

## Git Workflow

Working Directory → Staging Area → Commit
EOF

Now check:

git status

Git should report that README.md has been modified.

Step 10 — See what changed

Run:

git diff

You'll see the change that hasn't been staged yet.

This is extremely useful before committing.

Step 11 — Stage the change
git add README.md

Check:

git status

Now the modification should appear under:

Changes to be committed
Step 12 — Commit again
git commit -m "Document Git workflow"
Step 13 — View your history

Run:

git log --oneline

You should now have at least two commits:

abc1234 Document Git workflow
def5678 Add Day 22 Git basics README

The commit IDs will be different on your machine.

🔄 Understand the Complete Flow

You've just performed:

Create File
    │
    ▼
git status
    │
    ▼
Untracked
    │
    │ git add
    ▼
Staged
    │
    │ git commit
    ▼
Committed
    │
    ▼
Clean Working Tree

Then:

Modify File
    │
    ▼
git status
    │
    ▼
git diff
    │
    │ git add
    ▼
Staged
    │
    │ git commit
    ▼
New Snapshot

This is the Git workflow you should become comfortable with.

🧪 Bonus Exercise

Create another file:

echo "Day 22 Git practice" > notes.txt

Check:

git status

Stage it:

git add notes.txt

Commit it:

git commit -m "Add Git practice notes"

Then:

git log --oneline

You should now have three commits.

📊 Expected Final Structure

Your project should look like:

day22-git-basics/
│
├── .git/
├── README.md
└── notes.txt

And:

git status

should eventually show:

nothing to commit, working tree clean
🛠️ Troubleshooting
git: command not found

On Ubuntu/Debian:

sudo apt update
sudo apt install git

Verify:

git --version
Author identity unknown

Configure Git:

git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"

Then retry your commit.

nothing to commit

That's not necessarily an error.

Run:

git status

If you see:

nothing to commit, working tree clean

your repository is already up to date.

File is still untracked

Run:

git status

Then:

git add filename

For example:

git add README.md
You accidentally staged a file

To remove it from staging without deleting the file:

git restore --staged filename

For example:

git restore --staged README.md

Then:

git status
📸 What Screenshot Should You Capture?

For LinkedIn, capture a clean terminal showing these three commands:

git status
git log --oneline
git status

Ideally the final part should show:

$ git log --oneline

abc1234 Add Git practice notes
def5678 Document Git workflow
ghi9012 Add Day 22 Git basics README

$ git status

On branch main
nothing to commit, working tree clean

This gives visual proof of:

multiple commits + clean working tree

🎯 Day 22 Key Takeaways

Remember this workflow:

git init
   ↓
git status
   ↓
git add
   ↓
git commit
   ↓
git status
git init

Creates a Git repository.

git status

Tells you what's happening in the repository.

git add

Moves changes into the staging area.

git commit

Records staged changes in Git history.

And one command worth developing as a habit:

git status

When in doubt, check your Git status.

💼 LinkedIn Post — Day 22/100
Day 22 LinkedIn Post

🚀 Day 22/100 — Git Basics: init, add, commit & status

Today I went one level deeper into Git.

Yesterday I learned what Git and GitHub are and how they fit into the DevOps workflow.

Today I focused on four commands that form the foundation of everyday Git usage:

git init
git status
git add
git commit

At first, the difference between the working directory, staging area and repository seemed a little abstract.

After actually practicing it, the flow became much clearer:

Working Directory
       ↓
   git add
       ↓
 Staging Area
       ↓
  git commit
       ↓
Git Repository

I created a small project, initialized Git, added files, staged changes, created multiple commits and checked the repository status after each step.

One command I'm starting to appreciate more is:

git status

It gives you a quick picture of what's happening:

➡️ What's untracked?
➡️ What's modified?
➡️ What's staged?
➡️ What's already committed?

I also practiced:

git diff
git log --oneline

git diff helped me see exactly what had changed before staging it, while git log --oneline gave me a simple view of the project's history.

The biggest lesson today:

Don't just run Git commands—understand where your changes are moving.

Working Directory
        ↓
     Staging
        ↓
      Commit

This simple workflow is going to become the foundation for everything that comes later in my DevOps journey, especially when we get into GitHub workflows and CI/CD.

Day 22/100 completed. 💪

Still learning. Still practicing. One command at a time.

➡️ Next: Day 23 — Understanding Git Branches

#100DaysOfDevOps #DevOps #Git #GitHub #VersionControl #CI_CD #DevOpsJourney #LearningInPublic #Linux #Automation #Cloud #TechLearning #100DaysOfCode

🔖 Tomorrow — Day 23

Understanding Git Branches

We'll learn why branches exist, how to create and switch between them, and how developers use branches to work on features without affecting the main codebase.
