Day 08/100 — Linux File Permissions: Who Can Access What?

Yesterday, you learned how Linux commands can help you inspect and troubleshoot application logs.

Today, we go one level deeper into a concept you'll encounter constantly on Linux servers:

File permissions.

🎯 Today's Objective

By the end of today, you'll understand:

Linux users and groups
Read, write, and execute permissions
How to read ls -l output
Numeric permissions such as 755 and 644
chmod
chown
Why permission errors happen
How permissions affect DevOps scripts and deployments
🧠 The Problem

Imagine you're working on a production Linux server.

You download a deployment script:

deploy.sh

You try:

./deploy.sh

And Linux responds:

Permission denied

The script exists.

The script contains valid commands.

You're in the correct directory.

So why can't it run?

Because Linux doesn't only ask:

"Does this file exist?"

It also asks:

"Who are you, and what are you allowed to do with this file?"

This becomes extremely important in DevOps.

Think about:

Deployment scripts
Application configuration
SSH keys
Log files
Docker-related files
CI/CD agents
Cloud servers
Kubernetes configuration
Secrets

Incorrect permissions can cause applications to fail—or expose sensitive information.

🔐 Linux Permission Model

Linux permissions are primarily based on three categories:

Owner
Group
Others

And three basic permissions:

r = Read
w = Write
x = Execute

For example:

-rwxr-xr--

Break it down:

- | rwx | r-x | r--
    │      │     │
  Owner   Group Others

So:

Owner  → rwx
Group  → r-x
Others → r--
🔢 Numeric Permissions

Linux also represents permissions using numbers:

r = 4
w = 2
x = 1

Therefore:

rwx = 4 + 2 + 1 = 7
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4

So:

755

means:

Owner  → 7 → rwx
Group  → 5 → r-x
Others → 5 → r-x

And:

644

means:

Owner  → 6 → rw-
Group  → 4 → r--
Others → 4 → r--

A useful rule to remember:

7 = rwx
6 = rw-
5 = r-x
4 = r--
0 = ---
💻 Hands-on Lab — Fixing a Permission Problem
Step 1: Create today's lab
cd ~/devops-linux
mkdir -p day08
cd day08

Verify:

pwd
Step 2: Create a deployment script
cat > deploy.sh <<'EOF'
#!/bin/bash

echo "Starting deployment..."
echo "Application deployed successfully!"
EOF

View it:

cat deploy.sh

Expected:

#!/bin/bash
echo "Starting deployment..."
echo "Application deployed successfully!"
🚨 Step 3: Try to Execute It

Run:

./deploy.sh

You may see:

Permission denied

That's intentional.

The file exists, but it isn't currently executable.

Check its permissions:

ls -l deploy.sh

You may see something similar to:

-rw-r--r-- 1 user user ... deploy.sh

Notice:

-rw-r--r--

There is no x.

🔧 Step 4: Make the Script Executable

Run:

chmod +x deploy.sh

Check again:

ls -l deploy.sh

Now you should see something similar to:

-rwxr-xr-x

The x permission has been added.

Run:

./deploy.sh

Expected:

Starting deployment...
Application deployed successfully!

🎉 You've just fixed a real Linux permission problem.

🔢 Step 5: Use Numeric Permissions

Instead of:

chmod +x deploy.sh

you can explicitly define permissions.

Run:

chmod 755 deploy.sh

Check:

ls -l deploy.sh

Expected permission pattern:

-rwxr-xr-x

Remember:

755

Owner  = rwx
Group  = r-x
Others = r-x
📄 Step 6: Create a Configuration File
echo "APP_ENV=development" > app.conf

Check:

ls -l app.conf

You may see:

-rw-r--r--

That's typically represented as:

644
🔒 Step 7: Protect the Configuration File

Suppose this file contains sensitive information.

For example:

DB_PASSWORD=secret
API_KEY=12345

You wouldn't want every user on the system to read it.

Change its permissions:

chmod 600 app.conf

Check:

ls -l app.conf

Expected:

-rw-------

Meaning:

Owner  → rw-
Group  → ---
Others → ---

This is why understanding permissions matters for DevOps and security.

👤 Step 8: Check Your User

Run:

whoami

Expected:

your-linux-username

Now check your groups:

groups

You'll see the groups associated with your user.

We'll explore Linux users and groups in more detail later.

🔍 Step 9: Inspect Ownership

Run:

ls -l

Example:

-rwxr-xr-x 1 govind govind 78 deploy.sh
-rw------- 1 govind govind 21 app.conf

The important part is:

owner group

For example:

govind govind
👑 Step 10: Change Ownership

First create a file:

touch application.log

Check:

ls -l application.log

Linux provides the chown command to change ownership.

The general format is:

chown USER:GROUP FILE

For example:

sudo chown root:root application.log

⚠️ You may need sudo for this operation.

Verify:

ls -l application.log

The owner/group should now show:

root root

Don't worry if your environment behaves differently due to permissions or user configuration. The important concept today is understanding ownership vs permissions.

🧪 Step 11: Create a Permission Challenge

Create another script:

cat > test.sh <<'EOF'
#!/bin/bash
echo "Permission challenge solved!"
EOF

Remove execute permission:

chmod 644 test.sh

Try:

./test.sh

Expected:

Permission denied

Now fix it:

chmod 755 test.sh

Run:

./test.sh

Expected:

Permission challenge solved!
📊 Understand What Just Happened

Your workflow was:

Create Script
     ↓
No Execute Permission
     ↓
Permission Denied ❌
     ↓
chmod 755
     ↓
Execute Permission Added
     ↓
Script Runs Successfully ✅

This is a simple example, but the same concept appears when dealing with real deployment scripts, automation agents, and server configurations.

🧠 Commands Learned Today
Command	Purpose
ls -l	View permissions and ownership
chmod	Change permissions
chown	Change ownership
whoami	Show current user
groups	Show user's groups
sudo	Execute a command with elevated privileges

Important permission values:

4 = Read
2 = Write
1 = Execute

7 = rwx
6 = rw-
5 = r-x
4 = r--

Common examples:

644 → rw-r--r--
755 → rwxr-xr-x
600 → rw-------
🔧 Troubleshooting
Permission denied when running a script

Check:

ls -l script.sh

If there is no x, run:

chmod +x script.sh
chmod doesn't seem to work

Check the current permissions:

ls -l script.sh

Then explicitly set them:

chmod 755 script.sh
Operation not permitted

You may not own the file.

Check:

ls -l filename

If appropriate, you'll need elevated privileges:

sudo ...

Use sudo carefully—don't use it as a default fix for every permission problem.

./script.sh still doesn't run

Check the first line:

head -n 1 script.sh

It should be:

#!/bin/bash

Also check:

ls -l script.sh
📸 What to Capture for LinkedIn

A particularly good Day 08 screenshot is a before-and-after demonstration.

Show:

ls -l deploy.sh
./deploy.sh
chmod 755 deploy.sh
./deploy.sh

The screenshot should capture the story:

Permission denied ❌
        ↓
chmod 755
        ↓
Deployment successful ✅

You can also capture:

ls -l deploy.sh app.conf

to demonstrate the difference between:

755 → Executable script
600 → Protected configuration

That makes the post much more practical than simply sharing a list of commands.

✍️ LinkedIn Post — Day 08/100

🚀 Day 08/100 — Why Does Linux Say "Permission Denied"?

Imagine you're deploying an application to a Linux server.

You have the deployment script.

The script is correct.

The file exists.

You run:

./deploy.sh

And Linux responds:

Permission denied. ❌

Nothing appears to be wrong with the script.

So what happened?

The problem isn't the code.

It's the permissions.

Linux doesn't simply ask:

"Does this file exist?"

It also asks:

"Who are you, and what are you allowed to do with this file?"

This becomes extremely important in DevOps.

Think about:

🔹 Deployment scripts
🔹 Configuration files
🔹 SSH keys
🔹 Application logs
🔹 Cloud servers
🔹 CI/CD agents
🔹 Containers
🔹 Secrets

A small permission mistake can stop an application from deploying—or worse, expose information that shouldn't be accessible.

Today I practiced:

r → Read
w → Write
x → Execute

and learned how Linux separates permissions between:

Owner | Group | Others

I also deliberately created a Permission Denied error.

Then fixed it using:

chmod 755 deploy.sh

The result:

Permission denied ❌
        ↓
    chmod 755
        ↓
Script executed successfully ✅

I also explored why a configuration file might use:

chmod 600 app.conf

while an executable script might use:

chmod 755 deploy.sh

My biggest takeaway today:

Linux permissions aren't just a technical detail. They are part of reliability and security.

💭 A thought for today:

In DevOps, we often focus on making things accessible and automated.

But sometimes the more important question is:

"Who should NOT have access?"

Understanding Linux permissions is one small step toward answering that question correctly.

Day 08/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #DevOpsJourney #LearningInPublic #LinuxSecurity #Automation #CloudComputing

🔜 Day 09/100 — Linux Users, Groups & Sudo

Tomorrow we'll build on permissions by understanding who owns the files and why.

You'll learn:

User
 ↓
Group
 ↓
Permissions
 ↓
sudo
 ↓
Administrative Access

You'll create users and groups, assign permissions, and understand why DevOps engineers should avoid running everything as root.
