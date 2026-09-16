Today's topic: Understanding Pull Requests (PRs)

We are continuing from Day 25 — Git Rebase. No restart.

1. What is a Pull Request?

A Pull Request is a way to propose changes from one Git branch to another branch on a platform such as GitHub.

A typical workflow looks like:

Developer
   │
   ▼
Create Feature Branch
   │
   ▼
Write Code
   │
   ▼
Commit Changes
   │
   ▼
Push Branch to GitHub
   │
   ▼
Create Pull Request
   │
   ▼
Code Review
   │
   ├── Changes Requested
   │       │
   │       └── Developer updates code
   │
   ▼
Approval
   │
   ▼
Merge
   │
   ▼
CI/CD Pipeline
Important distinction

A Git branch is a Git concept.

A Pull Request is a collaboration/review mechanism provided by platforms such as GitHub.

So:

Git
 └── Branches
      └── Commits

GitHub
 └── Pull Requests
      ├── Review
      ├── Discussion
      ├── Approval
      └── Merge
2. Why Do We Need Pull Requests?

Imagine 10 developers are working on the same project.

If everyone directly changes main:

Developer A ──┐
Developer B ──┤
Developer C ──┼──> main
Developer D ──┤
Developer E ──┘

This can become difficult to manage.

Instead:

              feature-login
                   │
Developer ────────┤
                   │
              Pull Request
                   │
              Code Review
                   │
                   ▼
                  main

A PR creates a controlled point where the team can:

Review code
Discuss changes
Run automated tests
Run security checks
Request modifications
Approve changes
Merge the work
3. Pull Request vs git merge

This is important.

You can merge locally using:

git merge feature-login

A Pull Request is a collaborative workflow around that integration.

For example:

Local Git:

feature-login
     ↓
git merge
     ↓
main

GitHub workflow:

feature-login
     ↓
git push
     ↓
GitHub
     ↓
Pull Request
     ↓
Review + CI
     ↓
Merge
     ↓
main

In professional DevOps environments, the second workflow is extremely common.

🧪 4. Hands-On Lab — Create Your First Pull Request

We'll use GitHub for today's practical exercise.

You can use your existing GitHub repository or create a small practice repository.

Since you already have your 100 Days of DevOps repository, you can create a dedicated branch for today's exercise.

100 Days of DevOps GitHub repository

Recommendation: Don't directly experiment on main if the repository contains important work. Use a separate practice repository if you want zero risk.

Step 1 — Clone the Repository

If you haven't cloned your practice repository:

git clone <YOUR-REPOSITORY-URL>
cd <YOUR-REPOSITORY-DIRECTORY>

Check:

git remote -v

Expected:

origin  <repository-url> (fetch)
origin  <repository-url> (push)
🌿 5. Create a Feature Branch

First check your current branch:

git branch

Switch to main:

git switch main

Get the latest changes:

git pull origin main

Create a new branch:

git switch -c feature/day26-pull-request

Verify:

git branch

Expected:

* feature/day26-pull-request
  main
✏️ 6. Create a Change

Let's create a small file.

Linux/macOS:

mkdir -p day26
echo "# Day 26 - Pull Requests" > day26/README.md

PowerShell:

New-Item -ItemType Directory -Force day26
"# Day 26 - Pull Requests" | Set-Content day26/README.md

Check:

git status

Expected:

Untracked files:
    day26/
📦 7. Stage and Commit
git add day26/README.md

Check:

git status

Then:

git commit -m "Add Day 26 pull request notes"

Check history:

git log --oneline -3

Expected:

abc1234 Add Day 26 pull request notes
...
...
☁️ 8. Push the Branch to GitHub
git push -u origin feature/day26-pull-request

Expected output will look similar to:

[new branch] feature/day26-pull-request -> feature/day26-pull-request

Your branch now exists on GitHub.

🔀 9. Create the Pull Request

Open your repository on GitHub.

GitHub may show a message such as:

Compare & pull request

Select it.

Set:

base:   main
compare: feature/day26-pull-request

Add a useful title:

Add Day 26 Pull Request Notes

Description example:

## What changed?

- Added Day 26 Pull Request notes
- Added basic PR workflow documentation

## Testing

Verified the file exists and committed successfully.

## Related

Day 26/100 — Understanding Pull Requests

Then create the Pull Request.

👀 10. What Happens After Creating the PR?

The PR page gives your team a central place to inspect the change.

You can see:

Conversation

Discussion about the change.

Commits

All commits included in the PR.

Checks

Automated CI checks.

Files changed

Exactly what was modified.

For example:

Files changed

day26/
   └── README.md

This is where Pull Requests become extremely useful in DevOps.

🔍 11. Understanding Code Review

A reviewer can inspect:

Old code
   ↓
New code

They can leave comments such as:

Can we improve this section?

or:

Please add a test for this change.

The developer then makes another commit:

git add .
git commit -m "Address PR review comments"
git push

The existing Pull Request automatically gets updated.

You don't normally create another PR for each review comment.

🤖 12. Pull Requests and CI/CD

This is where today's topic connects directly to DevOps.

A common workflow is:

Developer pushes code
        ↓
Pull Request
        ↓
CI Pipeline
        ↓
Build
        ↓
Unit Tests
        ↓
Security Scan
        ↓
Code Review
        ↓
Approval
        ↓
Merge
        ↓
Deployment Pipeline

For example, GitHub Actions can automatically run when a PR is opened or updated.

We'll study GitHub Actions later in the CI/CD section of your 100-day roadmap.

🧪 13. Optional: Make a Second Commit

To understand how PRs update automatically, modify the file.

Linux/macOS:

echo "Pull Requests enable collaborative code review." >> day26/README.md

PowerShell:

"Pull Requests enable collaborative code review." | Add-Content day26/README.md

Then:

git add .
git commit -m "Improve Day 26 notes"
git push

Go back to your Pull Request.

You'll see the new commit included automatically.

That's an important practical behavior to understand.

🔧 14. Troubleshooting
Problem: git push says permission denied

Check your remote:

git remote -v

If you're using HTTPS, authenticate with GitHub using your configured credentials/token.

If you're using SSH, test:

ssh -T git@github.com
Problem: Branch already exists

Check:

git branch

If you already created the branch:

git switch feature/day26-pull-request
Problem: Git says there are uncommitted changes

Run:

git status

Then either commit them:

git add .
git commit -m "Save changes"

or intentionally discard changes if you don't need them.

Problem: Push rejected

First update your local main if appropriate:

git switch main
git pull origin main

Then return to your feature branch.

If your feature branch needs the latest main, you can use the rebase knowledge from Day 25:

git switch feature/day26-pull-request
git rebase main
git push --force-with-lease

⚠️ Use --force-with-lease carefully, especially on shared branches.

📸 15. What Screenshot Should You Capture?

For LinkedIn, I'd recommend two screenshots at most.

Screenshot 1 — Terminal

Show:

git branch
git log --oneline --graph --all
git status

For example:

* feature/day26-pull-request
  main

abc1234 Add Day 26 pull request notes
def5678 Previous commit

On branch feature/day26-pull-request
nothing to commit, working tree clean
Screenshot 2 — GitHub PR

Show the Pull Request page containing:

feature/day26-pull-request
            ↓
           main

Open
Files changed
Commits
Checks

The PR screenshot is the more valuable one, because today's lesson is specifically about collaborative Pull Requests.

🧠 16. Day 26 Key Takeaways

Remember these five things:

1️⃣ Branch

A separate line of development.

2️⃣ Commit

A saved snapshot of changes.

3️⃣ Push

Upload your branch/commits to the remote repository.

4️⃣ Pull Request

A proposal to integrate your changes into another branch, usually with review and automated checks.

5️⃣ Merge

Actually integrates the changes into the target branch.

So:

Branch
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
Approval
  ↓
Merge

That's a workflow you'll see repeatedly in real DevOps environments.

📸 LinkedIn Evidence

Your strongest proof of today's practical work is:

GitHub Pull Request + terminal showing the feature branch and clean working tree.

Avoid posting screenshots containing personal access tokens, passwords, private repository information, or other credentials.

💼 Day 26/100 — LinkedIn Post

Day 26/100 — Understanding Pull Requests 🔀

Yesterday I learned about Git Rebase.

Today I moved one step beyond local Git commands and learned something that's central to team-based development:

Pull Requests.

Before today, I mostly thought of Git as:

Code
 ↓
git add
 ↓
git commit
 ↓
git push

But in a real team, pushing code isn't the end of the process.

A typical workflow looks more like:

Feature Branch
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
  Approval
      ↓
    Merge

Today I created a feature branch, made a change, pushed it to GitHub, and opened a Pull Request against main.

One thing that became much clearer to me:

A Pull Request is not just about merging code.

It's a place where teams can:

✅ Review code
✅ Discuss changes
✅ Run automated tests
✅ Perform security checks
✅ Request improvements
✅ Approve changes
✅ Eventually merge into the target branch

This is where Git starts becoming more than just version control.

It becomes part of a collaborative software delivery workflow.

And I'm starting to see how GitHub, CI/CD, testing, security and deployment will eventually connect together.

Day 26/100 complete. 🚀

#100DaysOfDevOps #DevOps #Git #GitHub #PullRequest #VersionControl #CodeReview #CICD #DevOpsJourney #LearningInPublic
