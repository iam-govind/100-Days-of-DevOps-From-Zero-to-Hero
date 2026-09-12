Day 21 — Introduction to Git and GitHub

Date: September 13, 2026
Progress: 21/100

Today we move from networking into Git and GitHub, which form a major foundation for modern DevOps workflows.

1. What is Git?

Git is a distributed version control system.

In simple terms, Git helps you:

Track changes in your code
Go back to previous versions
Work on multiple features safely
Collaborate with other developers
Maintain a history of your project

Imagine you are working on a project:

Project
   │
   ├── Version 1
   ├── Version 2
   ├── Version 3
   └── Version 4

Without Git, managing these versions manually can become difficult.

With Git, the history is tracked automatically.

2. Why is Git important for DevOps?

DevOps depends heavily on automation.

A typical DevOps workflow looks like:

Developer
    │
    ▼
   Git
    │
    ▼
 GitHub
    │
    ▼
 CI/CD Pipeline
    │
    ▼
 Build
    │
    ▼
 Test
    │
    ▼
 Deploy

Git becomes the source of truth for application code and often for:

Infrastructure code
Dockerfiles
Kubernetes manifests
Terraform configurations
Ansible playbooks
CI/CD workflows

That's why Git is one of the first tools you should become comfortable with in DevOps.

3. Git vs GitHub

This is one of the most important concepts to understand.

Git

Git is the version control tool running on your computer.

Your Laptop
     │
     ▼
    Git
GitHub

GitHub is a cloud-based platform for hosting Git repositories and collaborating around them.

Your Laptop
     │
     │ Git
     ▼
   GitHub
     │
     ├── Collaboration
     ├── Pull Requests
     ├── Issues
     ├── Actions
     └── Releases
Simple comparison
Git	GitHub
Version control system	Git hosting/collaboration platform
Runs locally	Cloud platform
Tracks changes	Hosts repositories
Works offline	Requires network for remote operations
Command-line tool	Web + Git-based platform

Git is not GitHub.

Other platforms can also host Git repositories, such as GitLab and Bitbucket.

4. What is a Repository?

A repository, or repo, is where Git tracks your project.

For example:

my-devops-project/
│
├── README.md
├── app/
├── Dockerfile
├── terraform/
└── .git/

The .git directory contains Git's internal information and history.

When you initialize Git:

git init

Git creates this .git directory.

5. Git Working Areas

A very important concept is understanding how changes move through Git.

Working Directory
       │
       │ git add
       ▼
Staging Area
       │
       │ git commit
       ▼
Local Repository
       │
       │ git push
       ▼
Remote Repository
       │
       ▼
     GitHub
Working Directory

Where you actually create or modify files.

Staging Area

Files you've selected to include in your next commit.

Local Repository

Your local Git history.

Remote Repository

A repository hosted somewhere such as GitHub.

6. What is a Commit?

A commit is a snapshot of your project at a particular point in time.

For example:

Commit 1
"Create README"

      ↓

Commit 2
"Add Linux notes"

      ↓

Commit 3
"Add networking notes"

A commit normally contains:

Changes
Author information
Timestamp
Commit message
Unique commit ID

A good commit message explains what changed.

Example:

git commit -m "Add Day 21 Git notes"
🧪 Day 21 Hands-On Lab

Today we'll create a small Git project and connect it to GitHub.

Step 1 — Check whether Git is installed

Run:

git --version

Expected output:

git version 2.x.x

The exact version will depend on your system.

Step 2 — Configure your Git identity

Set your name:

git config --global user.name "Your Name"

Set your email:

git config --global user.email "your-email@example.com"

Check the configuration:

git config --global --list

You should see something similar to:

user.name=Your Name
user.email=your-email@example.com
Important

Use the email associated with your GitHub account if you want your commits to be properly associated with your GitHub profile.

Step 3 — Create a project directory

Run:

mkdir day21-git-introduction

Move into it:

cd day21-git-introduction

Check your location:

pwd
Step 4 — Initialize Git

Run:

git init

Expected output will look similar to:

Initialized empty Git repository in ...

Now check the directory:

ls -la

You should see:

.git

This means Git has been initialized.

Step 5 — Create your first file

Create a README:

echo "# Day 21 - Introduction to Git and GitHub" > README.md

Check the file:

cat README.md

Expected:

# Day 21 - Introduction to Git and GitHub
Step 6 — Check Git status

Run:

git status

You should see something similar to:

Untracked files:

    README.md

Git is telling you:

I see this file, but you haven't asked me to track it yet.

Step 7 — Add the file to staging

Run:

git add README.md

Or add everything in the current directory:

git add .

Now check:

git status

You should see something similar to:

Changes to be committed:

    new file: README.md

The file is now in the staging area.

Step 8 — Create your first commit

Run:

git commit -m "Add Day 21 Git introduction"

Expected output will be similar to:

[main xxxxxxx] Add Day 21 Git introduction
 1 file changed
 1 insertion(+)

Congratulations! 🎉

You have created your first Git commit for this lab.

Step 9 — View commit history

Run:

git log

You should see information about your commit.

For a cleaner view:

git log --oneline

Example:

a1b2c3d Add Day 21 Git introduction

The first part is the shortened commit ID.

Step 10 — Make another change

Add another line:

echo "Learning Git as part of my DevOps journey." >> README.md

Check the difference:

git diff

You should see the newly added line.

This is a very useful command because it shows you what changed before you commit it.

Step 11 — Stage and commit the change
git add README.md

Then:

git commit -m "Update Day 21 README"

Check history:

git log --oneline

You should now have two commits:

xxxxxxx Update Day 21 README
xxxxxxx Add Day 21 Git introduction
7. Connect the Local Repository to GitHub

Now we'll move from:

Local Git Repository

to:

GitHub Remote Repository

First, create a new empty repository on GitHub.

For this lab, call it something like:

day21-git-introduction

Avoid adding a README from GitHub if you've already created one locally.

After creating the repository, GitHub will provide commands similar to:

git remote add origin <YOUR-GITHUB-REPOSITORY-URL>

Then set the main branch:

git branch -M main

Then push:

git push -u origin main

Replace <YOUR-GITHUB-REPOSITORY-URL> with your actual repository URL.

8. Verify the Remote

Run:

git remote -v

You should see:

origin  <repository-url> (fetch)
origin  <repository-url> (push)

This tells Git where your remote repository is located.

9. Verify on GitHub

Open your repository in GitHub.

You should see:

README.md

and your commit history.

Your workflow is now:

Create/Edit File
      │
      ▼
git add
      │
      ▼
Staging Area
      │
      ▼
git commit
      │
      ▼
Local Repository
      │
      ▼
git push
      │
      ▼
GitHub
🔥 Important Git Commands Learned Today
Command	Purpose
git --version	Check Git installation
git config	Configure Git
git init	Initialize repository
git status	Check repository status
git add	Stage changes
git commit	Save changes to Git history
git log	View commit history
git log --oneline	Compact history
git diff	View changes
git remote -v	View remote repository
git push	Send commits to remote
git pull	Get changes from remote

Don't try to memorize everything today.

Focus on this basic workflow:

git status
     ↓
git add
     ↓
git commit
     ↓
git push
🛠️ Troubleshooting
git: command not found

Git isn't installed or isn't available in your PATH.

On Ubuntu/Debian:

sudo apt update
sudo apt install git

Then:

git --version
Git says your identity is unknown

Configure:

git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
nothing to commit

Run:

git status

If there are no changes, Git has nothing new to commit.

remote origin already exists

Check:

git remote -v

If you need to change it:

git remote set-url origin <YOUR-GITHUB-REPOSITORY-URL>
Push authentication problem

If GitHub asks for authentication, use an appropriate GitHub authentication method such as HTTPS credentials/token or SSH rather than your normal GitHub account password.

We will go deeper into GitHub workflows later in the series.

📸 What Screenshot Should You Capture?

For LinkedIn, I recommend one clean terminal screenshot showing:

git log --oneline

and:

git status

Ideally your screenshot should show something like:

$ git log --oneline

a1b2c3d Update Day 21 README
e4f5g6h Add Day 21 Git introduction

$ git status

On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean

This demonstrates that you've actually:

created → committed → pushed → verified

your Git repository.

🎯 Day 21 Key Takeaways

If you remember only five things from today:

1️⃣ Git tracks changes
Git = Version Control
2️⃣ GitHub hosts Git repositories
Git → Local
GitHub → Remote
3️⃣ Commit = snapshot
git commit

saves a point in your project's history.

4️⃣ Staging comes before committing
Working Directory
       ↓
   git add
       ↓
 Staging Area
       ↓
 git commit
       ↓
Repository
5️⃣ Push sends your commits to GitHub
git push
💼 LinkedIn Post — Day 21/100

Here's a human, learning-in-public version you can post with your terminal screenshot:

Day 21 LinkedIn Post

🚀 Day 21/100 — Introduction to Git & GitHub

Today I started one of the most important parts of my DevOps journey:

Git and GitHub.

Before starting this 100-day journey, I knew Git was used for version control and GitHub was used to store repositories.

But today I wanted to understand what actually happens when I work with Git.

I started with a simple project and went through the complete basic workflow:

Working Directory
      ↓
   git add
      ↓
 Staging Area
      ↓
 git commit
      ↓
Local Repository
      ↓
  git push
      ↓
    GitHub

I practiced commands like:

git init
git status
git add .
git commit -m "Add Day 21 Git introduction"
git log --oneline
git diff
git remote -v
git push

One thing that became much clearer today is the difference between Git and GitHub.

🔹 Git → Version control system
🔹 GitHub → Platform for hosting and collaborating on Git repositories

I also learned that a commit isn't just "saving a file."

It's a snapshot in the project's history that allows us to understand what changed and when.

That becomes especially important in DevOps because our repositories can contain much more than application code:

🐳 Dockerfiles
☸️ Kubernetes manifests
🏗️ Terraform code
⚙️ Ansible playbooks
🔄 CI/CD workflows
💻 Application source code

So Git isn't just a developer tool.

It's one of the foundations of a DevOps workflow.

Today's hands-on exercise was simple, but I think understanding these fundamentals properly will make the advanced Git and CI/CD topics much easier later.

Day 21/100 completed. 💪

One step closer to understanding the complete DevOps lifecycle.

➡️ Next: Day 22 — Git Basics: init, add, commit and status

#100DaysOfDevOps #DevOps #Git #GitHub #VersionControl #Linux #CI_CD #DevOpsJourney #LearningInPublic #Cloud #TechLearning #100DaysOfCode

🔖 Tomorrow

Day 22 — Git Basics: init, add, commit and status

We'll go deeper into the four fundamental Git commands and understand exactly how changes move through a Git repository.
