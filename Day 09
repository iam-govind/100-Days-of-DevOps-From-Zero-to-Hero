Day 09/100 — Linux Users, Groups & sudo

Welcome to Day 09/100 of your 100 Days of DevOps — From Zero to Hero journey.

So far:

Day 06: Linux fundamentals
Day 07: Essential Linux commands & log troubleshooting
Day 08: Linux file permissions
Day 09: Linux users, groups & sudo

Yesterday, we learned what permissions are.

Today, we're going to answer the next important question:

Who do those permissions actually apply to?

🎯 Today's Objective

By the end of today, you should understand:

Linux users
Linux groups
File ownership
User/group permissions
whoami
id
groups
useradd
groupadd
usermod
chown
sudo
Why DevOps engineers should avoid unnecessary root access
🧠 The Problem

Imagine you are managing a production Linux server.

There are multiple people and services using it:

Developer
Operations Engineer
CI/CD Pipeline
Application
Database
Monitoring Agent

Should everyone have access to everything?

Obviously not.

Imagine:

Developer
   ↓
Can modify application code

Application
   ↓
Can access application files

Monitoring Agent
   ↓
Can read monitoring data

Administrator
   ↓
Can manage the server

If every user has unrestricted access, one mistake could affect the entire server.

This creates two major concerns:

🔴 Security

Who can access sensitive files?

🔴 Reliability

Who can modify important system files?

This is why Linux uses users, groups, ownership, and permissions.

👤 Linux Users

Every Linux user has an identity.

Run:

whoami

Example:

govind

You can get more information using:

id

Example:

uid=1000(govind) gid=1000(govind) groups=1000(govind),27(sudo)

Don't worry about every number yet.

The important concepts are:

UID → User ID
GID → Group ID
Groups → Groups the user belongs to
👥 Linux Groups

Groups allow administrators to manage permissions for multiple users.

Instead of doing:

User A → Permission
User B → Permission
User C → Permission

you can create:

devops-team
     ↓
User A
User B
User C
     ↓
Shared Permission

This becomes extremely useful in teams and production environments.

🔐 Ownership + Permissions

Remember yesterday's example:

-rwxr-xr--

Linux separates access into:

Owner | Group | Others

For example:

-rwxr-xr--
   │    │   │
   │    │   └── Others
   │    └────── Group
   └─────────── Owner

So a file can be:

Owner → Read + Write + Execute
Group → Read + Execute
Others → Read

This gives you much more control than simply saying "allowed" or "not allowed."

🛠️ Hands-on Lab

Today we'll create a small simulated DevOps team.

Our goal:

devops-team
     │
     ├── developer
     └── tester

Then we'll use ownership and permissions to control access.

Step 1 — Create Today's Lab

Go to your Linux environment:

cd ~/devops-linux
mkdir -p day09
cd day09

Check:

pwd
Step 2 — Check Your Current User

Run:

whoami

Then:

id

And:

groups

Take a moment to look at the output.

You are already a Linux user, and you belong to one or more groups.

Step 3 — Create a Team Group

Creating users and groups generally requires administrator privileges.

Run:

sudo groupadd devops-team

Verify:

getent group devops-team

You should see something similar to:

devops-team:x:1001:

The numeric GID may be different on your system.

Step 4 — Create a Developer User

Run:

sudo useradd -m developer

Set a password:

sudo passwd developer

Linux will ask you to enter a password.

You don't need to use this account extensively today—the purpose is to understand how users are created and managed.

Step 5 — Create a Tester User

Run:

sudo useradd -m tester

Set a password:

sudo passwd tester

Now you have:

developer
tester
Step 6 — Add Users to the DevOps Group

Run:

sudo usermod -aG devops-team developer

Then:

sudo usermod -aG devops-team tester

Verify:

groups developer

You should see devops-team.

And:

groups tester

You should also see:

devops-team
🔑 Why -aG?

This is important.

usermod -aG group user

means:

Add the user to the supplementary group without removing their existing supplementary groups.

The -a is important.

Without it, you can accidentally replace existing supplementary group memberships.

📁 Step 7 — Create a Shared DevOps Directory

Create:

sudo mkdir -p /opt/devops-team

Check:

ls -ld /opt/devops-team

You may see:

drwxr-xr-x root root ...

Currently, root owns the directory.

👥 Step 8 — Change the Group Ownership

Change the group:

sudo chown root:devops-team /opt/devops-team

Check:

ls -ld /opt/devops-team

You should now see something similar to:

drwxr-xr-x root devops-team ...

The owner is still:

root

But the group is now:

devops-team
🔐 Step 9 — Give the Group Access

Run:

sudo chmod 770 /opt/devops-team

Check:

ls -ld /opt/devops-team

Expected permission pattern:

drwxrwx---

Meaning:

Owner  → rwx
Group  → rwx
Others → ---

Now:

root
  ↓
Full access

devops-team
  ↓
Full access

Everyone else
  ↓
No access

This is a simple example of least-privilege thinking.

📝 Step 10 — Create a Shared File

Create a file:

sudo touch /opt/devops-team/deployment-notes.txt

Change its group ownership:

sudo chown root:devops-team /opt/devops-team/deployment-notes.txt

Set permissions:

sudo chmod 660 /opt/devops-team/deployment-notes.txt

Check:

ls -l /opt/devops-team/deployment-notes.txt

Expected pattern:

-rw-rw----

This means:

Owner  → Read + Write
Group  → Read + Write
Others → No access
🔄 Step 11 — Test the Developer User

Switch to the developer account:

sudo su - developer

Check:

whoami

Expected:

developer

Check groups:

groups

You should see:

devops-team

Now try:

cd /opt/devops-team

Then:

echo "Deployment tested by developer" >> deployment-notes.txt

Read it:

cat deployment-notes.txt

Expected:

Deployment tested by developer

🎉 The group permissions are working.

🚪 Step 12 — Return to Your Original User

Run:

exit

Check:

whoami

You should be back to your original user.

🔎 Step 13 — Understand sudo

You have already used:

sudo

What does it mean?

Simply put:

sudo allows an authorized user to execute a command with elevated privileges.

For example:

sudo groupadd devops-team

Your normal user doesn't have permission to modify the system's group database.

sudo temporarily gives the command the required privileges.

👑 What About root?

root is the superuser on Linux.

Root can generally:

Modify system files
Create/delete users
Change permissions
Install software
Stop services
Change system configuration

That's powerful.

And that's exactly why it should be treated carefully.

A useful DevOps/security principle is:

Use the minimum privileges necessary to perform the task.

🧪 Mini Challenge

Try to answer these without looking back.

1. Who am I?
whoami
2. What groups do I belong to?
groups
3. What is my UID and GID?
id
4. What are the permissions on the shared directory?
ls -ld /opt/devops-team
5. What is the owner and group?
ls -ld /opt/devops-team

Look for:

OWNER GROUP
🧠 Important Commands Today
Command	Purpose
whoami	Show current user
id	Show UID, GID and groups
groups	Show group memberships
useradd	Create a user
passwd	Set/change password
groupadd	Create a group
usermod	Modify user settings
chown	Change ownership
chmod	Change permissions
sudo	Run command with elevated privileges
su	Switch user
getent group	Look up group information
🔧 Troubleshooting
groupadd: command not found

You're probably not in a standard Linux environment.

Check:

uname

You should see:

Linux
Permission denied

Check:

ls -ld /opt/devops-team

If you're modifying a system directory, you may need:

sudo
Developer cannot access /opt/devops-team

Check the group:

groups developer

Make sure devops-team appears.

Also check:

ls -ld /opt/devops-team

You should have something similar to:

drwxrwx--- root devops-team
Group membership doesn't appear immediately

If the user was already logged in before being added to the group, the current session may not have the updated group membership.

Exit the session and start a new one:

exit

Then log in again.

You accidentally removed group memberships

Be careful with:

usermod -G

Prefer:

usermod -aG

when adding a user to an additional group.

📸 What to Capture for LinkedIn

For your Day 09 post, capture a screenshot showing:

whoami
id
groups

Then show:

ls -ld /opt/devops-team
ls -l /opt/devops-team/deployment-notes.txt

Ideally, your screenshot should demonstrate:

devops-team
      ↓
Shared Directory
      ↓
Owner  → root
Group  → devops-team
      ↓
Permissions
      ↓
Developer can access

This gives your LinkedIn post a real hands-on story.

✍️ LinkedIn Post — Day 09/100

🚀 Day 09/100 — Who Should Have Access to Your Linux Server?

Imagine you're managing a production Linux server with multiple developers, operations engineers, applications, and automation tools.

Should everyone have access to everything?

Probably not.

Giving everyone unrestricted access creates two problems:

🔴 Security risk
🔴 Operational risk

A developer may need access to application files.

A deployment process may need access to deployment directories.

A monitoring agent may only need to read certain files.

And an administrator may need much broader access.

So the question becomes:

How do we control who can access what?

This is where Linux users, groups, ownership, and permissions come together.

Today I created a small devops-team group and two users:

devops-team
   ├── developer
   └── tester

I then created a shared directory and assigned the group ownership:

Owner → root
Group → devops-team

Finally, I applied permissions so the team could access the directory while others had no access.

I also practiced:

whoami → identify the current user
id → UID, GID and groups
groups → group membership
useradd → create users
groupadd → create groups
usermod → manage group membership
chown → change ownership
chmod → control permissions
sudo → perform authorized administrative tasks

One command I paid particular attention to was:

usermod -aG devops-team developer

The -a matters because we want to add the user to the group without unnecessarily replacing their existing group memberships.

My biggest takeaway today:

Good DevOps isn't only about making systems accessible and automated. It's also about making sure access is given to the right people and processes.

💭 A thought for today:

If everyone can access everything, is the system really automated—or just exposed?

Understanding Linux users, groups, and permissions is another small but important step toward building secure and reliable infrastructure.

Day 09/100 completed ✅

Follow my profile for more practical insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #DevOpsJourney #LinuxSecurity #Automation #CloudComputing #LearningInPublic

📌 GitHub File

Save today's lesson as:

day09-linux-users-groups-sudo.md

Recommended structure:

100-days-of-devops/
│
├── README.md
├── day01-devops-fundamentals.md
├── day02-why-devops.md
├── day03-traditional-vs-devops.md
├── day04-devops-lifecycle.md
├── day05-ci-cd.md
├── day06-linux-fundamentals.md
├── day07-linux-commands.md
├── day08-linux-file-permissions.md
├── day09-linux-users-groups-sudo.md
│
└── labs/
    ├── day06/
    ├── day07/
    ├── day08/
    └── day09/
🔜 Day 10/100 — Linux Processes & Services

Tomorrow we'll move from "Who can access the system?" to:

"What is actually running on the system?"

You'll learn:

Processes vs services
ps
top
htop
kill
systemctl
Starting/stopping services
Checking service status
Simulating a failed service
Basic Linux process troubleshooting

This will be your first step toward real server administration and troubleshooting. 🚀
