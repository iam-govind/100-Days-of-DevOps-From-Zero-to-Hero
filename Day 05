Day 05/100 — CI vs Continuous Delivery vs Continuous Deployment

Today, you’ll learn one of the most important concepts in modern DevOps:

What is the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?

These three concepts are closely connected, but they are not the same.

🎯 Today's Objective

By the end of today, you should understand:

What Continuous Integration (CI) means
What Continuous Delivery means
What Continuous Deployment means
The key difference between them
How they work together in a DevOps workflow
A simple hands-on simulation of CI/CD
🧠 The Problem

Imagine a team where developers work on the same application.

Developer A makes changes.

Developer B makes changes.

Developer C makes changes.

But instead of integrating their changes regularly, everyone waits until the end of the week.

Then suddenly:

Developer A's Code
        +
Developer B's Code
        +
Developer C's Code
        ↓
      MERGE 😰
        ↓
    Errors Found ❌

Now the team must figure out:

Which change caused the issue?
Why did the code fail?
Why does it work on one machine but not another?
Can the application still be released?

The longer teams wait to integrate changes, the more difficult problems can become.

So, what is the potential solution?

Integrate changes frequently, test them automatically, and make releases repeatable.

This leads us to CI/CD.

🔄 What Is CI/CD?

CI/CD generally refers to:

Continuous Integration
        ↓
Continuous Delivery
        ↓
Continuous Deployment

Let's understand each one.

1️⃣ Continuous Integration (CI)

Continuous Integration means developers frequently integrate their code changes into a shared codebase.

Each change can trigger automated processes such as:

Code Change
    ↓
Build
    ↓
Test
    ↓
Validation
    ↓
Feedback

If something fails:

Code Change
    ↓
Build/Test ❌
    ↓
Developer Gets Feedback
    ↓
Fix
    ↓
Run Again
🎯 Main Goal of CI

Find problems early.

Instead of waiting days or weeks to discover integration problems, CI helps identify them soon after changes are introduced.

2️⃣ Continuous Delivery

Continuous Delivery takes CI further.

After code successfully passes the build and test stages, the application is kept in a release-ready state.

The workflow may look like:

Code
 ↓
Build
 ↓
Test
 ↓
Ready for Release ✅
 ↓
Manual Approval
 ↓
Production Deployment

The important point is:

The application is ready to be deployed, but production deployment may still require a human decision.

🎯 Main Goal of Continuous Delivery

Make releases reliable and repeatable.

3️⃣ Continuous Deployment

Continuous Deployment goes one step further.

If all required checks pass successfully, the application is automatically deployed.

Code
 ↓
Build
 ↓
Test
 ↓
All Checks Passed ✅
 ↓
Automatically Deploy
 ↓
Production 🚀

There is no manual approval required between successful validation and deployment.

🎯 Main Goal of Continuous Deployment

Deliver validated changes automatically.

🔥 The Key Difference

Here is the simplest way to remember it:

Practice	What Happens?
Continuous Integration	Code is frequently integrated, built, and tested
Continuous Delivery	Validated code is kept ready for release
Continuous Deployment	Validated code is automatically deployed

Or visually:

CI
Code → Build → Test

Continuous Delivery
Code → Build → Test → Ready for Release → Manual Decision → Deploy

Continuous Deployment
Code → Build → Test → Automatic Deploy
💡 Real-Life Example

Imagine you update an application from Version 1 to Version 2.

Without CI/CD
Developer changes code
        ↓
Developer manually builds
        ↓
Someone manually tests
        ↓
Someone manually prepares the release
        ↓
Someone manually deploys

Now compare it with CI/CD:

Developer pushes code
        ↓
Automated Build
        ↓
Automated Test
        ↓
Validation
        ↓
Ready to Release / Automatically Deploy

The major advantage is not simply speed.

It is also:

Repeatability
Consistency
Faster feedback
Early issue detection
Reduced manual effort
💻 Hands-on Lab — Simulating a Simple CI/CD Pipeline

Today, you will create a small Bash-based pipeline.

Step 1: Go to your repository
cd ~/devops-100-days

If your repository has a different name, use the correct path.

Check your location:

pwd
Step 2: Create the Day 5 lab
mkdir -p labs/day05
cd labs/day05
Step 3: Create a simple application
echo "Welcome to My DevOps CI/CD Journey - Version 1" > app.txt

Verify:

cat app.txt

Expected:

Welcome to My DevOps CI/CD Journey - Version 1
Step 4: Create the pipeline directories
mkdir -p build test release production

Check:

ls

Expected output should include:

app.txt
build
production
release
test
⚙️ Step 5: Create Your CI/CD Script

Create a file:

nano pipeline.sh

Add:

#!/bin/bash

echo "===== CI/CD PIPELINE STARTED ====="

echo ""
echo "[1/4] BUILD STAGE"
cp app.txt build/app.txt
echo "Build completed successfully."

echo ""
echo "[2/4] TEST STAGE"

if grep -q "DevOps" build/app.txt
then
    echo "Test passed successfully."
else
    echo "Test failed!"
    exit 1
fi

echo ""
echo "[3/4] RELEASE STAGE"
cp build/app.txt release/app-v1.0.txt
echo "Release created successfully."

echo ""
echo "[4/4] DEPLOY STAGE"
cp release/app-v1.0.txt production/app.txt
echo "Deployment completed successfully."

echo ""
echo "===== PIPELINE COMPLETED SUCCESSFULLY ====="

Save and exit.

Make it executable:

chmod +x pipeline.sh
▶️ Step 6: Run the Pipeline
./pipeline.sh

Expected output:

===== CI/CD PIPELINE STARTED =====

[1/4] BUILD STAGE
Build completed successfully.

[2/4] TEST STAGE
Test passed successfully.

[3/4] RELEASE STAGE
Release created successfully.

[4/4] DEPLOY STAGE
Deployment completed successfully.

===== PIPELINE COMPLETED SUCCESSFULLY =====
🧪 Step 7: Verify Production

Run:

cat production/app.txt

Expected:

Welcome to My DevOps CI/CD Journey - Version 1

Your application has now moved through:

Code
 ↓
Build
 ↓
Test
 ↓
Release
 ↓
Deploy
❌ Step 8: Simulate a Failed Test

Now change the application:

echo "Welcome to My Learning Journey - Version 2" > app.txt

Run the pipeline again:

./pipeline.sh

Expected:

Test failed!

Why?

Because our test expects the word:

DevOps

But the updated file no longer contains it.

This is an important CI concept:

If validation fails, the pipeline should stop before deployment.

Your production environment should still contain the previous successful version.

Check:

cat production/app.txt

Expected:

Welcome to My DevOps CI/CD Journey - Version 1
🔧 Step 9: Fix the Issue

Update the application:

echo "Welcome to My DevOps CI/CD Journey - Version 2" > app.txt

Run:

./pipeline.sh

Verify production:

cat production/app.txt

Expected:

Welcome to My DevOps CI/CD Journey - Version 2

🎉 Your pipeline successfully detected a problem, stopped, and then completed after the issue was fixed.

📁 Expected Final Structure
labs/day05/
├── app.txt
├── pipeline.sh
├── build/
│   └── app.txt
├── test/
├── release/
│   └── app-v1.0.txt
└── production/
    └── app.txt

Check:

find . -type f
🔧 Troubleshooting
Problem: Permission denied

Run:

chmod +x pipeline.sh

Then:

./pipeline.sh
Problem: Test failed!

Check:

cat app.txt

Make sure it contains the word:

DevOps
Problem: No such file or directory

Create the required folders again:

mkdir -p build test release production

Then run:

./pipeline.sh
📸 What to Capture for LinkedIn

Take screenshots of:

Screenshot 1 — Successful Pipeline
./pipeline.sh

Show:

Build completed successfully.
Test passed successfully.
Release created successfully.
Deployment completed successfully.
Screenshot 2 — Failed Pipeline

Show the pipeline stopping with:

Test failed!
Screenshot 3 — Production Version

After fixing the issue:

cat production/app.txt

Show:

Welcome to My DevOps CI/CD Journey - Version 2
✍️ LinkedIn Post — Day 05/100

🚀 Day 05/100 — What Happens When Code Changes Faster Than Teams Can Validate It?

Imagine multiple developers working on the same application.

Everyone writes code.

Everyone makes changes.

And instead of validating those changes regularly, they are combined at the end of the week.

Then comes the real problem:

❌ Integration errors
❌ Broken builds
❌ Difficult troubleshooting
❌ Late discovery of issues
❌ Manual testing delays
❌ Uncertainty about whether the application is ready to release

The longer we wait to integrate and validate changes, the harder it can become to identify what went wrong.

So, what could be the solution?

What if every code change could be validated early?

What if builds and tests happened through a repeatable process?

What if successful changes were always kept ready for release?

This is where CI/CD comes in.

🔹 Continuous Integration helps teams frequently integrate, build, and test changes.

🔹 Continuous Delivery ensures validated applications remain ready for release.

🔹 Continuous Deployment takes it one step further by automatically deploying validated changes.

Today, I simulated a simple pipeline:

Code → Build → Test → Release → Deploy

I intentionally made a change that caused the test to fail.

The pipeline stopped before deployment. ❌

After fixing the issue, I ran it again, and the new version successfully reached production. ✅

My biggest takeaway:

The real strength of CI/CD is not just moving code faster. It is building confidence that every change follows a consistent validation process.

💭 A thought to leave with:

As software changes faster, should we rely more on people remembering every step—or build systems that can validate those steps consistently?

That's the mindset CI/CD encourages.

Day 05/100 completed ✅

Follow my profile for more practical insights and hands-on learning from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #CICD #ContinuousIntegration #ContinuousDelivery #ContinuousDeployment #Automation #LearningInPublic #DevOpsJourney

🔜 Day 06/100 Teaser

Tomorrow, you’ll begin the Linux section of the journey:

What is Linux, and why is it so important for DevOps?

You’ll learn why Linux is widely used in servers, cloud environments, containers, and DevOps workflows—and start your Linux hands-on practice.
