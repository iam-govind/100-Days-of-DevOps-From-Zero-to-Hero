Day 06/100 — Why Linux Is So Important in DevOps

You’ve completed the DevOps fundamentals section. Today we begin the next major part of the journey:

Linux for DevOps

Linux is one of the most important foundations for anyone working with DevOps, cloud, containers, servers, and automation.

🎯 Today's Objective

By the end of Day 6, you should understand:

What Linux is
Why Linux is widely used in DevOps
Linux vs Windows from a DevOps perspective
Basic Linux terminology
How to navigate a Linux system
Your first essential Linux commands
How to create, modify, and inspect files
🧠 First: What Exactly Is Linux?

Linux is an open-source operating system kernel.

You will commonly hear names such as:

Ubuntu
Debian
Red Hat Enterprise Linux
Rocky Linux
Fedora
Amazon Linux

These are Linux distributions built around the Linux kernel.

A simplified view:

Linux Distribution
        │
        ├── Linux Kernel
        ├── System Utilities
        ├── Package Manager
        └── Applications

For your DevOps journey, Ubuntu is a great distribution to start with because it is widely used and beginner-friendly.

🤔 Why Does DevOps Care So Much About Linux?

Imagine you eventually deploy an application to the cloud.

You may have:

Developer Laptop
       ↓
GitHub
       ↓
CI/CD Pipeline
       ↓
Cloud Server
       ↓
Docker Container
       ↓
Kubernetes

A large part of this environment commonly runs on Linux.

Linux gives DevOps engineers powerful tools for:

Server administration
Networking
Automation
Scripting
Containers
Cloud infrastructure
Application deployment
Monitoring
Troubleshooting

That's why Linux isn't just another topic in this journey.

Linux is one of the foundations on which many DevOps tools and workflows are built.

🆚 Linux vs Windows

This isn't about saying one operating system is universally better.

They are designed with different strengths and ecosystems.

For DevOps, Linux is particularly valuable because of its:

🔹 Command-line environment

You can perform many tasks without a graphical interface.

🔹 Automation capabilities

Shell scripts can automate repetitive operations.

🔹 Server usage

Linux is extremely common in cloud and server environments.

🔹 Developer ecosystem

Many DevOps tools provide strong Linux support.

🔹 Containers

Linux concepts are deeply connected to technologies such as Docker and Kubernetes.

🖥️ Your First Linux Concept: The Terminal

Instead of clicking through folders, Linux lets you interact with the system through commands.

For example:

pwd

means:

Print Working Directory

And:

ls

means:

List files and directories

This may look simple.

But eventually you'll use the same command-line mindset to manage:

Servers
Containers
Cloud VMs
Kubernetes environments
CI/CD agents
💻 Hands-on Lab — Your First Linux Commands
Step 1: Open a Linux terminal

If you already have Ubuntu/Linux installed, open your terminal.

If you're using Windows and don't have Linux installed yet, WSL (Windows Subsystem for Linux) with Ubuntu is a convenient way to start practicing Linux without replacing Windows.

Once your Linux terminal is open, continue.

Step 2: Check where you are

Run:

pwd

Example:

/home/govind

Your username/path may be different.

Step 3: List files
ls

For more details:

ls -l

To include hidden files:

ls -la
📁 Step 4: Create Your DevOps Learning Directory

Run:

mkdir -p ~/devops-linux/day06

Move into it:

cd ~/devops-linux/day06

Verify:

pwd

You should see something similar to:

/home/govind/devops-linux/day06
📄 Step 5: Create Your First File

Run:

touch devops.txt

Check:

ls -l

You should see:

devops.txt
✍️ Step 6: Write Something Into the File

Run:

echo "Day 06 - Learning Linux for DevOps" > devops.txt

Now read the file:

cat devops.txt

Expected:

Day 06 - Learning Linux for DevOps
➕ Step 7: Add More Content

Use:

echo "Linux is a foundation for DevOps." >> devops.txt

Notice the difference:

>   → Replace file contents
>>  → Append to file

Check:

cat devops.txt

Expected:

Day 06 - Learning Linux for DevOps
Linux is a foundation for DevOps.
📂 Step 8: Create Directories

Run:

mkdir notes scripts logs

Check:

ls

Expected:

devops.txt
logs
notes
scripts
🔄 Step 9: Move Into a Directory
cd notes

Check:

pwd

Go back:

cd ..

The .. means:

Parent directory

📋 Step 10: Copy a File

Run:

cp devops.txt notes/

Check:

ls notes

Expected:

devops.txt
🔀 Step 11: Move a File

Create another file:

touch old-notes.txt

Move it into the notes directory:

mv old-notes.txt notes/

Check:

ls notes

You should now have:

devops.txt
old-notes.txt
🗑️ Step 12: Delete a File

Remove the file we just created:

rm notes/old-notes.txt

Verify:

ls notes

Only devops.txt should remain.

⚠️ Be careful with rm.

Unlike a graphical recycle bin, deleted files may not be easily recoverable.

🔍 Step 13: Find Your Files

Go back to your main directory:

cd ~/devops-linux/day06

Run:

find . -type f

Expected:

./devops.txt
./notes/devops.txt

This is particularly useful later when working with large server environments.

📊 Step 14: Check File Information

Run:

ls -lh

You will see information such as:

Permissions
Owner
File size
Date/time
File name

Example:

-rw-r--r-- 1 user user 65 Aug 29 09:45 devops.txt

Don't worry about understanding every column yet.

We'll cover Linux permissions later in the journey.

🧪 Mini Challenge

Try completing this without looking back at the commands.

Create this structure:

day06/
├── app/
│   └── application.txt
├── logs/
│   └── application.log
└── scripts/

The commands you need are things you've already learned.

Then verify:

find . -type f

Expected:

./app/application.txt
./logs/application.log
🔑 Commands Learned Today
Command	Purpose
pwd	Show current directory
ls	List files
ls -la	List all files with details
cd	Change directory
mkdir	Create directory
touch	Create empty file
echo	Print/write text
cat	Display file contents
cp	Copy files
mv	Move/rename files
rm	Remove files
find	Search for files
..	Parent directory
~	Home directory

Don't try to memorize everything immediately.

The goal is to use these commands repeatedly until they become natural.

🔧 Troubleshooting
command not found

Check that you're actually inside a Linux terminal:

uname

A Linux system should return something similar to:

Linux
No such file or directory

Check your current location:

pwd

Then:

ls

Use cd to navigate to the correct directory.

File already exists

That's usually fine.

Check it with:

ls -l
Permission denied

Don't immediately use sudo.

First inspect the file/directory:

ls -l

We'll properly learn Linux permissions later.

📸 What to Capture for LinkedIn

A good Day 6 screenshot would show your terminal with:

pwd
ls -la
find . -type f

And ideally the final structure:

day06/
├── app/
│   └── application.txt
├── logs/
│   └── application.log
└── scripts/

You can also show:

cat devops.txt

with:

Day 06 - Learning Linux for DevOps
Linux is a foundation for DevOps.

The important thing is to show hands-on work, not just a screenshot of commands copied from a tutorial.

✍️ LinkedIn Post — Day 06/100

🚀 Day 06/100 — Before Learning DevOps Tools, Learn Where They Run

When we hear "DevOps", it's easy to immediately think about:

Docker.
Kubernetes.
Terraform.
Jenkins.
Cloud.
CI/CD.

But there's a question we often overlook:

Where do many of these tools actually run?

Behind the dashboards, pipelines, containers, and cloud infrastructure, there is often something fundamental:

Linux.

And this is where my Linux journey begins.

Today I started with the basics:

pwd → ls → cd → mkdir → touch → cp → mv → rm → find

At first, these look like simple commands.

But I realized something important.

A DevOps engineer may eventually need to:

Navigate servers
Inspect files
Check logs
Manage processes
Troubleshoot applications
Automate repetitive tasks
Work with cloud VMs
Manage containers

And a lot of that happens from the command line.

So instead of jumping straight into advanced DevOps tools, I'm taking a step back and strengthening the foundation.

Today I created directories, created files, copied and moved them, inspected their contents, and searched through the directory structure.

My biggest takeaway:

You don't need to know every Linux command to start DevOps. But you need to become comfortable working with the Linux command line.

💭 A thought for today:

We often focus on learning the latest DevOps tools.

But how strong can our DevOps skills become if the foundation underneath them is weak?

Day 06/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and learnings from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #DevOpsJourney #LearningInPublic #CloudComputing #Automation #CICD

🔜 Day 07/100 — Essential Linux Commands

Tomorrow we'll go deeper into Linux commands and start working with:

File and directory navigation
grep
head
tail
less
wc
sort
history
Command chaining
Pipes |
Redirection > and >>

And we'll use them to inspect and troubleshoot a simulated application log—a much more realistic DevOps scenario.
