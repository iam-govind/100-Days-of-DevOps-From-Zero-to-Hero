Day 07/100 — Essential Linux Commands for DevOps

Welcome to Day 07/100 of your restarted 100 Days of DevOps — From Zero to Hero journey.

Yesterday, we learned why Linux is such an important foundation for DevOps and practiced basic commands.

Today, we'll take the next step: using Linux commands to inspect, search, and troubleshoot information—skills you'll use constantly when working with servers and application logs.

🎯 Today's Objective

By the end of today, you should be comfortable with:

grep
head
tail
less
wc
sort
history
Pipes |
Output redirection
Searching application logs

Most importantly, you'll practice these commands using a simulated application log, rather than just running isolated commands.

🧠 The Problem

Imagine your application is running on a production Linux server.

Users suddenly report:

"The application is slow and sometimes returning errors."

The server has thousands of log entries.

You could open the entire log and manually search through it—but that's not practical.

You need answers quickly:

How many errors occurred?
Which users experienced errors?
What happened most recently?
Which error appears most frequently?
What happened around a particular time?

This is where Linux command-line tools become extremely powerful.

Instead of manually searching thousands of lines:

Application Log
     ↓
Thousands of entries
     ↓
Linux Commands
     ↓
Relevant Information
     ↓
Troubleshoot Faster

This command-line approach is one of the reasons Linux skills are so valuable in DevOps.

🔥 Today's Most Important Concept: Pipes

One of the most powerful Linux concepts is:

|

called a pipe.

It allows the output of one command to become the input of another.

For example:

cat application.log | grep ERROR

Think of it as:

application.log
      ↓
     cat
      ↓
    grep
      ↓
 ERROR lines

You can chain commands together to answer more complex questions.

💻 Hands-on Lab — Troubleshooting Application Logs
Step 1: Create today's lab
cd ~/devops-linux
mkdir -p day07
cd day07

Verify:

pwd
Step 2: Create a Simulated Application Log

Run:

cat > application.log <<'EOF'
2026-08-30 08:00:01 INFO Application started
2026-08-30 08:01:15 INFO User login successful user=alice
2026-08-30 08:02:10 INFO User login successful user=bob
2026-08-30 08:03:22 ERROR Database connection failed
2026-08-30 08:04:11 INFO Retry connection
2026-08-30 08:05:43 INFO Database connection restored
2026-08-30 08:06:15 INFO User login successful user=charlie
2026-08-30 08:07:20 WARNING High memory usage
2026-08-30 08:08:31 ERROR API request failed status=500
2026-08-30 08:09:12 INFO API request successful
2026-08-30 08:10:05 INFO User logout user=alice
2026-08-30 08:11:17 ERROR Database connection failed
2026-08-30 08:12:33 INFO Retry connection
2026-08-30 08:13:40 INFO Database connection restored
2026-08-30 08:14:22 INFO User login successful user=david
2026-08-30 08:15:18 WARNING High CPU usage
2026-08-30 08:16:44 ERROR API request failed status=500
2026-08-30 08:17:30 INFO API request successful
2026-08-30 08:18:12 INFO User logout user=bob
2026-08-30 08:19:50 ERROR Authentication service unavailable
2026-08-30 08:20:10 INFO Authentication service restored
EOF

Check the file:

cat application.log
🔎 Step 3: Search for Errors with grep

Run:

grep "ERROR" application.log

You'll see only the error entries.

This is extremely useful when troubleshooting large logs.

You can also search for warnings:

grep "WARNING" application.log
🔢 Step 4: Count Errors

Run:

grep -c "ERROR" application.log

Expected:

5

You've just answered:

How many errors occurred?

without manually counting them.

📊 Step 5: Count All Log Entries

Run:

wc -l application.log

Expected:

21 application.log

wc -l counts the number of lines.

You can combine it with grep:

grep "ERROR" application.log | wc -l

Expected:

5

This demonstrates the power of pipes:

grep
 ↓
ERROR lines
 ↓
wc
 ↓
Count
🕐 Step 6: See the Latest Logs

Use:

tail application.log

This shows the last 10 lines.

To see only the last 5:

tail -n 5 application.log

Expected output will contain the latest five log entries.

This is particularly useful when investigating a problem that just occurred.

📌 Step 7: See the First Few Logs

Use:

head -n 5 application.log

This displays the first five lines.

So:

head

helps you inspect the beginning.

tail

helps you inspect the end.

👀 Step 8: Use less

For a large log file:

less application.log

Inside less:

Space → Next page
b → Previous page
/ERROR → Search for ERROR
n → Next search result
q → Quit

For real production logs, less can be much more practical than printing an entire file to the terminal.

🔤 Step 9: Sort Information

Let's extract all log levels:

awk '{print $3}' application.log | sort

The exact output depends on the log structure, but this demonstrates an important pattern:

Extract → Sort

You don't need to master awk today. We'll revisit it later.

📈 Step 10: Find the Most Common Error

Run:

grep "ERROR" application.log | sed 's/.*ERROR //' | sort | uniq -c

You should see counts for repeated error messages.

This demonstrates how multiple Linux commands can be combined into a small troubleshooting workflow.

🧪 Step 11: Search for Database Problems

Run:

grep "Database" application.log

You should find entries related to:

Database connection failed
Database connection restored

Now count failures:

grep "Database connection failed" application.log | wc -l

Expected:

2
🚨 Step 12: Find HTTP 500 Errors

Run:

grep "status=500" application.log

Expected:

2026-08-30 08:08:31 ERROR API request failed status=500
2026-08-30 08:16:44 ERROR API request failed status=500

Count them:

grep -c "status=500" application.log

Expected:

2
🔁 Step 13: Follow a Log in Real Time

This is one of the most useful commands you'll learn today:

tail -f application.log

The terminal will wait for new entries.

Open another terminal and run:

cd ~/devops-linux/day07
echo "2026-08-30 08:21:00 INFO New request received" >> application.log

You should see the new entry appear automatically in the first terminal.

Press:

Ctrl + C

to stop tail -f.

This is commonly useful when watching logs while troubleshooting a running application.

🔗 The DevOps Connection

Today you've essentially performed a basic troubleshooting workflow:

Application
     ↓
Logs
     ↓
Linux Commands
     ↓
Search
     ↓
Filter
     ↓
Count
     ↓
Identify Problem
     ↓
Troubleshoot

Later in your journey, tools such as container platforms, Kubernetes, and monitoring systems will generate enormous amounts of information.

The ability to confidently work from a terminal will remain extremely valuable.

🧠 Commands Learned Today
Command	Purpose
grep	Search text
head	Show beginning of file
tail	Show end of file
tail -f	Follow a changing log
less	View large files interactively
wc -l	Count lines
sort	Sort output
uniq -c	Count repeated values
history	Show previous commands
|	Pipe output between commands
>	Redirect/overwrite output
>>	Append output
🧪 Mini Challenge

Without looking at the previous commands, try to answer these questions:

1. How many INFO entries are there?

Hint:

grep -c "INFO" application.log
2. How many warnings?
grep -c "WARNING" application.log
3. How many authentication-related errors?
grep "ERROR" application.log | grep "Authentication"
4. Show the last 3 log entries.
tail -n 3 application.log
5. Search for every failed operation.
grep "failed" application.log

Try solving these yourself before checking the commands.

🔧 Troubleshooting
application.log: No such file or directory

Check:

pwd
ls

You should be inside:

~/devops-linux/day07
grep returns nothing

Linux searches are case-sensitive.

For example:

grep "ERROR" application.log

is different from:

grep "error" application.log

For a case-insensitive search:

grep -i "error" application.log
tail -f appears stuck

That's expected.

It's waiting for new log entries.

Press:

Ctrl + C

to stop it.

less doesn't exit

Press:

q
📸 What to Capture for LinkedIn

For today's post, a strong screenshot would show your terminal with:

grep -c "ERROR" application.log
grep -c "WARNING" application.log
grep "status=500" application.log
tail -n 5 application.log

Even better, capture a screenshot showing:

tail -f application.log

while a new log entry appears.

That demonstrates real hands-on troubleshooting, rather than simply listing commands.

✍️ LinkedIn Post — Day 07/100

🚀 Day 07/100 — When Production Breaks, Logs Become Your First Clue

Imagine receiving this message at 2 AM:

"Users are getting errors in production."

You connect to the Linux server.

There are thousands of log entries.

Now what?

Scrolling through everything manually isn't troubleshooting.

It's searching for a needle in a haystack. 🔍

The real challenge isn't getting access to the logs.

It's being able to quickly answer:

❓ How many errors occurred?
❓ What happened most recently?
❓ Which service is failing?
❓ Are HTTP 500 errors increasing?
❓ Is the database connection failing?

This is where Linux command-line skills become extremely powerful.

Today, I created a simulated application log and practiced commands such as:

🔹 grep — Search for specific information
🔹 head / tail — Inspect the beginning or end of logs
🔹 tail -f — Follow logs in real time
🔹 wc — Count entries
🔹 sort / uniq — Analyze repeated information
🔹 | — Combine commands into powerful workflows

For example:

Logs
 ↓
grep ERROR
 ↓
wc -l
 ↓
Number of Errors

And:

Live Application Logs
        ↓
     tail -f
        ↓
Real-Time Troubleshooting

What looked like a few simple Linux commands suddenly became a practical troubleshooting toolkit.

My biggest takeaway today:

DevOps isn't only about deploying applications. It's also about understanding what happens when things don't go as planned.

💭 A thought for today:

When production has a problem, the question isn't:

"Where is the error?"

The better question is:

"How quickly can I find the information that helps me understand the problem?"

That's where strong Linux fundamentals start making a real difference.

Day 07/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #DevOpsJourney #LearningInPublic #Troubleshooting #Automation #CloudComputing #DevOpsLearning

🔜 Day 08/100 — Linux File Permissions

Tomorrow we'll tackle one of the most important Linux concepts for DevOps:

Who can access what—and why?

You'll learn:

r → Read
w → Write
x → Execute

We'll work with:

Users
Groups
File permissions
chmod
chown
Permission errors
Executable scripts

And you'll deliberately create a permission denied problem and fix it—exactly the kind of troubleshooting you'll encounter on real Linux systems.
