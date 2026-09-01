ay 10/100 — Linux Processes & Services

Welcome to Day 10/100 of your restarted 100 Days of DevOps — From Zero to Hero journey.

We've built a solid Linux foundation:

Day 06: Linux fundamentals
Day 07: Linux commands & log troubleshooting
Day 08: File permissions
Day 09: Users, groups & sudo

Today we're moving from:

"Who can access the system?"

to:

"What is actually running on the system?"

This is a critical DevOps skill because when an application is slow, unavailable, or consuming too many resources, one of the first things you'll need to investigate is processes and services.

🎯 Today's Objective

By the end of Day 10, you should understand:

What a Linux process is
Process IDs (PIDs)
Foreground vs background processes
ps
top
pgrep
kill
jobs
bg
fg
What a Linux service is
systemctl
How to check service status
How to start and stop services
Basic process troubleshooting
🧠 The Problem

Imagine you receive this alert:

🚨 Application is not responding

You log into the Linux server.

What do you check first?

You need to determine:

Is the application running?
        ↓
Is the process consuming too much CPU?
        ↓
Is it consuming too much memory?
        ↓
Is the service stopped?
        ↓
Is the process stuck?
        ↓
Do we need to restart it?

This is where Linux process and service management becomes essential.

🔄 Process vs Service

These two terms are related, but they're not identical.

Process

A process is a running instance of a program.

For example:

bash
python
nginx
java
docker

Each running process normally has a PID.

Example:

PID    COMMAND
1250   bash
1422   python
1601   nginx
Service

A service is typically a long-running background application managed by the operating system's service manager.

Examples:

SSH
Web server
Database
Docker
Monitoring agent

On many modern Linux distributions, services are managed through systemd and systemctl.

🆔 What Is a PID?

PID means:

Process ID

Every running process gets an identifier.

Run:

ps

Example:

PID TTY          TIME CMD
1234 pts/0    00:00:00 bash

The number:

1234

is the PID.

💻 Hands-on Lab
Step 1 — Create Today's Lab
cd ~/devops-linux
mkdir -p day10
cd day10

Verify:

pwd
Step 2 — See Your Current Processes

Run:

ps

You may see something like:

PID TTY          TIME CMD
1234 pts/0    00:00:00 bash
1456 pts/0    00:00:00 ps

You're seeing processes associated with your current terminal session.

🔎 Step 3 — See More Processes

Run:

ps aux

This provides much more information.

You may see:

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  ...   ... ?        Ss   ...      ... /sbin/init
root       ...  0.0  0.2  ...   ... ?        ...         ... sshd
user       ...  0.0  0.1  ...   ... pts/0    ...         ... bash

Important columns include:

Column	Meaning
USER	Process owner
PID	Process ID
%CPU	CPU usage
%MEM	Memory usage
STAT	Process state
COMMAND	Command/program
🔍 Step 4 — Search for a Process

Instead of manually searching the entire process list:

ps aux | grep bash

You can also use:

pgrep bash

This returns the PID(s) of matching processes.

📊 Step 5 — Use top

Run:

top

You'll see a continuously updating view of the system.

Look for:

%CPU
%MEM
PID
COMMAND

This is useful when investigating:

"Why is my server slow?"

You can identify processes consuming significant CPU or memory.

Exit with:

q
🚀 Step 6 — Create a Background Process

Let's create a simple process that runs continuously.

Run:

sleep 300 &

The & means:

Run the command in the background.

You should see something similar to:

[1] 2345

Here:

[1] → Job number
2345 → PID

Your PID will be different.

🔎 Step 7 — Check Background Jobs

Run:

jobs

Expected:

[1]+  Running    sleep 300 &

Now find the process:

pgrep sleep

You should get its PID.

🛑 Step 8 — Stop the Process

Suppose the PID is:

2345

Run:

kill 2345

Replace 2345 with the PID from your system.

Check:

jobs

The job should now be terminated.

You can also verify:

pgrep sleep

If there is no output, the process is no longer running.

⚠️ What Does kill Actually Mean?

Despite its name, kill doesn't always mean "forcefully terminate."

It sends a signal to a process.

The default:

kill PID

sends:

SIGTERM

This politely asks the process to terminate.

A more forceful signal is:

kill -9 PID

which sends:

SIGKILL
Important:

Don't immediately use:

kill -9

Use normal termination first:

kill PID

Forceful termination should generally be reserved for situations where the process doesn't respond appropriately.

🔄 Step 9 — Foreground vs Background

Run:

sleep 60

Your terminal waits until the command finishes.

That's a foreground process.

Press:

Ctrl + C

to stop it.

Now:

sleep 60 &

The command runs in the background and your terminal remains available.

Check:

jobs
🔙 Step 10 — Bring a Job to the Foreground

If you have:

[1]+ Running sleep 60 &

run:

fg %1

The process comes back to the foreground.

You can stop it with:

Ctrl + C
🖥️ Step 11 — Understanding Linux Services

Now let's look at services.

Run:

systemctl --type=service --state=running

You'll see services currently running on your system.

Depending on your Linux environment, the list will vary.

🔍 Step 12 — Check SSH Service

On many Ubuntu installations, the SSH service is called:

ssh

Check:

systemctl status ssh

If SSH is installed and managed by systemd, you may see:

Active: active (running)

That's what you want for a running SSH service.

If SSH isn't installed or your environment doesn't use systemd, that's okay—your system may simply differ.

▶️ Step 13 — Start and Stop a Service

For a service such as SSH, the general commands are:

sudo systemctl stop ssh

Start it:

sudo systemctl start ssh

Check status:

systemctl status ssh

⚠️ Important: If you're connected to a remote server through SSH, don't casually stop the SSH service. You could terminate your access to the machine.

For today's practice, use a service you know is safe to stop in your own lab environment.

🔄 Step 14 — Restart a Service

General syntax:

sudo systemctl restart SERVICE_NAME

Example:

sudo systemctl restart ssh

Again, don't do this to SSH on a remote production machine without understanding the consequences.

🧪 Step 15 — Check Whether a Service Is Enabled

Run:

systemctl is-enabled ssh

Possible output:

enabled

This means the service is configured to start automatically during boot.

Compare:

active

with:

enabled

They are different concepts.

active

The service is currently running.

enabled

The service is configured to start automatically when appropriate during system boot.

A service can therefore be:

active + enabled

or:

inactive + enabled

for example, if it has been stopped manually.

🧠 DevOps Troubleshooting Workflow

Suppose your application isn't responding.

A basic investigation might look like:

Application Problem
       ↓
Is the service running?
       ↓
systemctl status
       ↓
Is the process running?
       ↓
ps / pgrep
       ↓
Is CPU or memory high?
       ↓
top
       ↓
Is the process stuck?
       ↓
kill
       ↓
Check logs

Notice how this connects directly with Day 07, where you learned to investigate logs.

Linux troubleshooting isn't about memorizing isolated commands.

It's about building a logical investigation process.

🧪 Mini Challenge

Try these yourself.

Challenge 1

Find your current shell process:

ps
Challenge 2

Find the PID of your shell:

pgrep bash

If you're using a different shell, such as zsh, search for that shell instead.

Challenge 3

Start a background process:

sleep 600 &

Find it:

pgrep sleep
Challenge 4

Check its process information:

ps -p PID -f

Replace PID with the actual number.

Challenge 5

Terminate it gracefully:

kill PID
Challenge 6

Confirm it's gone:

pgrep sleep
🧠 Commands Learned Today
Command	Purpose
ps	View processes
ps aux	View detailed process list
pgrep	Find process IDs
top	Monitor processes/resources
jobs	View shell background jobs
bg	Resume a job in background
fg	Bring a job to foreground
kill	Send a signal to a process
systemctl status	Check service status
systemctl start	Start a service
systemctl stop	Stop a service
systemctl restart	Restart a service
systemctl is-enabled	Check whether service starts at boot
🔧 Troubleshooting
systemctl: command not found

Your environment may not use systemd.

This is common in some containers, WSL configurations, or minimal Linux environments.

You can still practice:

ps
top
pgrep
kill
jobs
Unit ssh.service could not be found

SSH may not be installed, or the service may have a different name.

Find related services:

systemctl list-unit-files | grep -i ssh
Failed to connect to bus

This often indicates that systemd isn't running in your current environment.

Don't worry—you can continue the process portion of today's lab.

kill: No such process

The process may have already terminated.

Check:

pgrep sleep
kill -9 temptation 😄

Don't use:

kill -9

as your first response.

Try:

kill PID

first.

Think:

Graceful termination before forced termination.

📸 What to Capture for LinkedIn

For today's LinkedIn screenshot, create a simple troubleshooting story.

Capture:

Screenshot 1 — Process Investigation
ps aux | head

and:

pgrep sleep
Screenshot 2 — Resource Monitoring

Capture the top output showing:

PID
%CPU
%MEM
COMMAND
Screenshot 3 — Process Lifecycle

Show:

sleep 600 &
pgrep sleep
kill PID
pgrep sleep

The story becomes:

Process Started
      ↓
PID Identified
      ↓
Process Monitored
      ↓
Process Terminated

That's much more interesting for LinkedIn than simply posting a list of Linux commands.

✍️ LinkedIn Post — Day 10/100

🚀 Day 10/100 — What Is Actually Running on a Linux Server?

Imagine getting an alert:

🚨 "The application is not responding."

You SSH into the server.

The first question shouldn't immediately be:

"Should I restart everything?"

Instead, start investigating.

Is the application process running?

Is it consuming too much CPU?

Is memory usage unusually high?

Is the service stopped?

Is the process stuck?

This is where Linux process and service management becomes essential for DevOps.

Today I learned the difference between a process and a service.

A process is a running instance of a program and normally has a unique:

PID → Process ID

A service is typically a long-running application managed by the operating system's service manager.

I practiced commands including:

🔹 ps — View processes
🔹 ps aux — Detailed process information
🔹 pgrep — Find process IDs
🔹 top — Monitor CPU and memory usage
🔹 jobs — View background jobs
🔹 fg / bg — Manage foreground/background jobs
🔹 kill — Send signals to processes
🔹 systemctl — Manage Linux services

I also created a background process:

sleep 600 &

Found its PID:

pgrep sleep

and then gracefully terminated it:

kill PID

The exercise helped me understand an important troubleshooting pattern:

Application Problem
       ↓
Check Service
       ↓
Check Process
       ↓
Check CPU / Memory
       ↓
Investigate Logs
       ↓
Take Action

My biggest takeaway today:

Good troubleshooting starts with observation, not action.

It's tempting to restart a service when something breaks.

But before taking action, understand:

What is running?

What is consuming resources?

What failed?

Why did it fail?

That's the mindset I'm trying to build throughout this 100-day journey.

Day 10/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #DevOpsJourney #LinuxAdministration #Troubleshooting #Automation #CloudComputing #LearningInPublic

📁 Recommended GitHub Files

Save today's lesson as:

day10-linux-processes-services.md

Lab structure:

labs/
└── day10/
    └── README.md

You can also save useful commands in:

labs/day10/process-commands.sh
🔜 Day 11/100 — Linux Package Management

Tomorrow we'll learn how DevOps engineers install, update, remove, and manage software on Linux.

We'll cover:

apt
 ↓
Package
 ↓
Repository
 ↓
Install
 ↓
Update
 ↓
Remove

You'll install a real package, inspect it, remove it, and understand how package management becomes important when preparing servers for applications and DevOps tools.
