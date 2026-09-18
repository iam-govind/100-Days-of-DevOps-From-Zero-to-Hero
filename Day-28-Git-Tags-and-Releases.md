# 🚀 100 Days of DevOps — Day 28/100

## Git Tags and Releases

Today we're learning how to mark important points in our Git history using **Tags**, and how those tags can become **Releases** on GitHub.

---

## 1. What is a Git Tag?

A Git tag is a reference that points to a specific commit.

Think of it like putting a **label on an important version** of your project.

```text
Git History

A ── B ── C ── D ── E
              ↑
           v1.0.0
             TAG
```

For example:

```text
v1.0.0
v1.1.0
v2.0.0
```

Instead of remembering a commit ID such as `a83f91c`, you can refer to it as `v1.0.0`.

---

## 2. Why Are Tags Important in DevOps?

Imagine your application has been successfully deployed to production.

You want to remember exactly which code was deployed.

```text
v1.0.0 → Production release
v1.1.0 → New features
v1.2.0 → Bug fixes
v2.0.0 → Major changes
```

Tags are useful for:

- Production deployments
- Rollbacks
- Version tracking
- Release management
- CI/CD pipelines
- Auditing
- Identifying stable versions

---

## 3. Git Tag vs Git Branch

| Git Branch | Git Tag |
|---|---|
| Moves as new commits are added | Normally stays pointing to a specific commit |
| Used for development | Used to mark important versions |
| Can contain ongoing work | Represents a specific point |
| Example: `feature/login` | Example: `v1.0.0` |

Simple way to remember:

> **Branch = where development continues**  
> **Tag = a label for an important point in history**

---

## 4. Types of Git Tags

### Lightweight tag

A simple pointer to a commit:

```bash
git tag v1.0.0
```

### Annotated tag

Contains additional information such as tag message, tagger, date, and metadata:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

For production release management, annotated tags are generally more useful because they contain release metadata.

---

# 🧪 5. Hands-on Lab

We'll use a local Git repository so you can safely experiment.

## Step 1 — Create a practice repository

```bash
mkdir git-tags-lab
cd git-tags-lab
git init
```

Expected:

```text
Initialized empty Git repository
```

---

## Step 2 — Configure Git if necessary

Check:

```bash
git config --global user.name
git config --global user.email
```

If they're empty:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Use your own details.

---

## Step 3 — Create the application file

```bash
echo "# My DevOps Application" > README.md
cat README.md
```

Expected:

```text
# My DevOps Application
```

---

## Step 4 — Commit the application

```bash
git add README.md
git commit -m "Initial application version"
```

Check:

```bash
git log --oneline
```

You should see something similar to:

```text
abc1234 Initial application version
```

---

# 🏷️ 6. Create a Lightweight Tag

```bash
git tag v1.0.0
```

List tags:

```bash
git tag
```

Expected:

```text
v1.0.0
```

---

# 🔎 7. Inspect the Tag

```bash
git show v1.0.0
```

You should see the commit associated with the tag.

---

# 🏷️ 8. Create an Annotated Tag

Create another commit:

```bash
echo "Version 1.1 features" >> README.md
git add README.md
git commit -m "Add version 1.1 features"
```

Create an annotated tag:

```bash
git tag -a v1.1.0 -m "Release version 1.1.0"
```

Check:

```bash
git tag
```

Expected:

```text
v1.0.0
v1.1.0
```

---

# 🔍 9. Inspect the Annotated Tag

```bash
git show v1.1.0
```

Compare:

```bash
git show v1.0.0
git show v1.1.0
```

---

# 🌐 10. Push Tags to GitHub

Creating a tag locally does **not automatically push it to GitHub**.

Push one tag:

```bash
git push origin v1.1.0
```

Or push all local tags:

```bash
git push origin --tags
```

Verify:

```bash
git ls-remote --tags origin
```

---

# 🧹 11. Delete a Tag

Delete locally:

```bash
git tag -d v1.1.0
```

If it was already pushed to GitHub:

```bash
git push origin --delete v1.1.0
```

⚠️ Be careful deleting tags already used by production processes.

---

# 🔄 12. Checkout a Specific Version

Inspect an older version:

```bash
git checkout v1.0.0
```

This puts you at the code represented by that tag.

Return to your branch:

```bash
git switch main
```

If your branch is called `master`:

```bash
git switch master
```

---

# 🚀 13. What is a GitHub Release?

A **GitHub Release** is a way to package and communicate a particular version of your project.

```text
Code
 │
 ▼
Commit
 │
 ▼
Tag
 │
 ▼
GitHub Release
 │
 ├── Release Notes
 ├── Version Information
 └── Optional Assets
```

For example:

```text
v1.0.0
│
├── Application code
├── Release notes
└── Build artifacts
```

A GitHub Release is commonly associated with a Git tag.

---

# 🏭 14. Git Tags in a CI/CD Pipeline

A pipeline could be configured like:

```text
Developer
    │
    ▼
Git Commit
    │
    ▼
Pull Request
    │
    ▼
Code Review
    │
    ▼
Merge
    │
    ▼
Create Tag
    │
    ▼
v1.0.0
    │
    ▼
CI/CD Pipeline
    │
    ▼
Production
```

A CI/CD system might be configured to deploy whenever a tag matching `v*` is pushed.

For example:

```bash
git push origin v1.0.0
```

could trigger a production release pipeline.

**Important:** the actual deployment behavior depends on how the CI/CD pipeline is configured.

---

# 🔢 15. Understanding Semantic Versioning

You'll often see versions such as:

```text
v1.4.2
```

This commonly follows **Semantic Versioning (SemVer)**:

```text
MAJOR.MINOR.PATCH
   │     │     │
   │     │     └── Bug fixes
   │     └──────── New backward-compatible features
   └────────────── Breaking changes
```

Examples:

```text
1.0.0 → First release
1.0.1 → Bug fix
1.1.0 → New backward-compatible feature
2.0.0 → Breaking change
```

---

# 🧪 Day 28 Challenge

Create three commits and tag them:

```text
v1.0.0
v1.1.0
v2.0.0
```

Conceptually:

```text
A ── B ── C
│    │    │
│    │    └── v2.0.0
│    └─────── v1.1.0
└──────────── v1.0.0
```

Then:

```bash
git tag
```

Expected:

```text
v1.0.0
v1.1.0
v2.0.0
```

Finally:

```bash
git log --oneline --decorate
```

You should see the tags associated with their commits.

---

# 🛠️ Troubleshooting

### `fatal: tag 'v1.0.0' already exists`

Check:

```bash
git tag
```

If you really need to recreate it:

```bash
git tag -d v1.0.0
git tag v1.0.0
```

### Tag doesn't appear on GitHub

Local tags are shown by:

```bash
git tag
```

Push the tag:

```bash
git push origin v1.0.0
```

Or:

```bash
git push origin --tags
```

### `src refspec v1.0.0 does not match any`

Check:

```bash
git tag
```

If the tag doesn't exist:

```bash
git tag v1.0.0
git push origin v1.0.0
```

### Detached HEAD after checking out a tag

That's expected.

Check:

```bash
git status
```

Return to your branch:

```bash
git switch main
```

---

# 📸 What to Capture for LinkedIn

Capture a clean terminal screenshot showing:

```bash
git tag
git log --oneline --decorate
```

For example:

```text
v1.0.0
v1.1.0
v2.0.0

c45ab12 (HEAD -> main, tag: v2.0.0) Release v2.0.0
91de321 (tag: v1.1.0) Add new feature
32ac789 (tag: v1.0.0) Initial release
```

If you've pushed the tags to GitHub, an even better screenshot is the **GitHub Releases/Tags page** showing your version numbers.

---

# 🧠 Day 28 Takeaways

Today you learned:

- What Git tags are
- Why tags are useful
- Lightweight vs annotated tags
- How to create tags
- How to inspect tags
- How to push tags to GitHub
- How to delete tags
- What GitHub Releases are
- Semantic Versioning
- How tags can connect Git to CI/CD

> **A tag gives a name to an important point in your Git history.**

---

# 💼 LinkedIn Post — Day 28/100

🚀 Day 28/100 — 100 Days of DevOps

Today I learned something simple in Git that becomes very important when we start talking about releases and production deployments:

🏷️ Git Tags

A Git tag is basically a label attached to a specific commit.

Instead of remembering a commit like:

`abc1234`

we can give that important version a meaningful name:

`v1.0.0`

For example:

`v1.0.0` → First release  
`v1.1.0` → New feature  
`v1.1.1` → Bug fix  
`v2.0.0` → Breaking change

One thing that helped me understand tags:

**Branch = where development continues**

**Tag = a name for an important point in history**

I also practiced the difference between:

👉 Lightweight tags  
👉 Annotated tags

I then pushed tags to GitHub and explored how they can be associated with GitHub Releases.

The DevOps connection was the most interesting part.

A production workflow can look like:

Code  
↓  
Commit  
↓  
Pull Request  
↓  
Code Review  
↓  
Merge  
↓  
Create Tag  
↓  
v1.0.0  
↓  
CI/CD Pipeline  
↓  
Production

This means version tags can become an important part of release and deployment automation.

I also started understanding Semantic Versioning:

**MAJOR.MINOR.PATCH**

For example:

`2.0.0` → Breaking change  
`1.5.0` → New feature  
`1.5.1` → Bug fix

Today's hands-on practice:

✔ Created Git tags  
✔ Created annotated tags  
✔ Inspected tagged commits  
✔ Pushed tags to GitHub  
✔ Practiced versioning  
✔ Connected tags with a real-world CI/CD workflow

Another small Git concept learned today, but one that has a big role in production workflows.

Day 28 complete. 🚀

Tomorrow: Git Best Practices

#100DaysOfDevOps #DevOps #Git #GitHub #GitTags #GitHubReleases #CI #CD #CICD #DevOpsLearning #VersionControl #SemanticVersioning

---

## 🔜 Day 29 Teaser

**Git Best Practices**

We'll move from learning individual Git commands to understanding how Git should actually be used in a professional team:

```text
Good Commits
     ↓
Good Branches
     ↓
Good PRs
     ↓
Clean History
     ↓
Reliable Releases
```

**Day 28 complete. 🚀**
