# Assignment 7 — AI-Assisted AWS Security and Cost Audit

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash script that audits the AWS resources you deployed earlier this week — your S3 static site, EC2 instance(s), security groups, RDS database, and EBS volumes — for common security and cost misconfigurations.

You will then connect that script to Claude Code as a reusable `/aws-audit` skill that explains what it found and recommends a fix, without ever making the fix itself.

Finally, you will find a real misconfiguration in your own account, apply the fix yourself, and prove it worked with a second audit run.

---

# Task 1 — Confirm Your AWS Resources and Set Up Your Workspace

## Goal

Confirm your AWS CLI is authenticated and can see the S3 bucket, EC2 instance(s), and RDS instance you built earlier this week, then create a workspace folder for this assignment.

### Evidence

#### Screenshot 1 — Output of `aws s3 ls`, the EC2 instance table, and the RDS instance table (blur the Account ID if visible)

[Output](screenshots/s3-list.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort`

[Output](screenshots/sort.png)

---

### Notes You Must Write (Very Important)

**1. Which resources from this week's earlier assignments did you see in the listings?**

I saw my S3 static website bucket, my EC2 instance in the [running] state, and my RDS database, all in eu-north-1. 

**2. Why must you confirm your resources exist before writing an audit script against them?**

An audit is only meaningful if it is checking real resources. If the CLI points at the wrong region or account, or a resource was deleted, the script could report “PASS” simply because it found nothing to check. That would give a false sense of security. Confirming first also proves my credentials and region are set up correctly, so any later errors come from the script and not from my setup.

---

# Task 2 — Define Safety Rules in CLAUDE.md

## Goal

Create a `CLAUDE.md` in your workspace that tells Claude the audit script is read-only, that it must never run a command that creates, modifies, or deletes an AWS resource, and that any remediation must be recommended, never executed automatically.

### Evidence

#### Screenshot 3 — `CLAUDE.md` open in VS Code showing all four sections

[Output](screenshots/Claude-sections.png)

---

### Notes You Must Write (Very Important)

**1. Why should Claude never be given permission to run `revoke-security-group-ingress` itself, even if the fix is obviously correct?**

Claude should not run revoke-security-group-ingress on its own, since the command makes a direct change to live AWS infrastructure. A rule that looks risky may still be needed, and removing it without review could cut off SSH access, break the application, or stop other services that depend on that port. Claude’s job is to spot the problem, explain what could go wrong, and suggest the fix. A human then checks the change and decides whether to run it.

**2. Which rule prevents Claude from claiming a finding that the report does not support?**

This Safety Rule: “Do not claim a finding unless the report contains supporting evidence.” It is reinforced by “Use only the Bash audit report as the primary source of evidence” and by Output Rule 3, which requires Claude to show the exact evidence from the report for every WARN or FAIL finding. Together, these mean Claude cannot guess, assume, or invent problems. Every finding must trace back to a specific line in the report.

---

# Task 3 — Plan the Audit with Claude Code

## Goal

Ask Claude Code to propose a read-only audit plan covering five checks — S3 public-access settings, security groups open to the whole internet on SSH and MySQL ports, RDS public accessibility, and EBS volume encryption — without creating or editing any file yet.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan

[Output](screenshots/Claude-output.png)

---

### Notes You Must Write (Very Important)

**1. Which part of this task represents the Gather phase?**

The Gather phase is when Claude reads CLAUDE.md and the workspace and works out which read-only AWS CLI commands would collect the information needed for each check. Nothing is changed or created. Claude only collects context and evidence before any script is written.

**2. Did every proposed command start with `describe-`, `get-`, or `list-`? Why does that matter?**

Yes, in the AWS CLI, these prefixes are read-only: they return information but never create, modify, or delete anything. Commands such as create-, put-, modify-, delete-, revoke-, and authorize- change resources. Checking the prefixes is a quick way to confirm the plan follows the safety rules in CLAUDE.md.

---

# Task 4 — Build the AWS Audit Script

## Goal

Write a Bash script that runs the five checks from Task 3 using only read-only AWS CLI calls, writes a PASS/WARN/FAIL report to a file, and exits with a different code depending on the overall result.

Make it executable and confirm it has no syntax errors.

### Evidence

#### Screenshot 5 — Top section of `aws-audit.sh` showing the variables and the checks array

[Output](screenshots/Audit.png)

---

#### Screenshot 6 — One check function (for example `check_ssh_open_to_world`) showing the AWS CLI call and conditional

[Output](screenshots/Check.png)

---

#### Screenshot 7 — Output of `bash -n scripts/aws-audit.sh` and `ls -l scripts/aws-audit.sh`

[Output](screenshots/Script.png)

---

### Notes You Must Write (Very Important)

**1. What is stored in the checks array, and how does the loop use it?**

The checks array stores the names of the five check functions.
These are check_s3_public_access, check_ssh_open_to_world, check_mysql_open_to_world, check_rds_public_access, and check_ebs_encryption.
After the header is printed, a for loop goes through the array one item at a time.
The loop calls each function by name using "$check_function".
Each function writes a PASS, WARN, or FAIL result to the report and updates its counter.

**2. Why does every AWS CLI call in this script use `--query` and `--output text` instead of parsing raw JSON?**

--query pulls out only the value each check needs, such as PubliclyAccessible or the number of open security groups.
--output text returns that value as plain text like True, False, or 2.
Bash can then compare it directly using [ ] and -gt.
No extra tools like jq are needed to read the result.
This keeps each check short, easy to read, and less likely to break.

**3. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

The exit code shows how serious the result is without anyone reading the report.
The script returns 0 for HEALTHY, 1 for WARN, and 2 for FAIL.
A CI/CD pipeline, scheduled job, or the /aws-audit skill can use this code to decide what to do next.
It can continue, send an alert, or stop, depending on the result.
Using one exit code for everything would hide the difference between a small warning and a real security failure.

---

# Task 5 — Run the Baseline Audit

## Goal

Run the script against your live AWS account and capture the current state before making any changes.

### Evidence

#### Screenshot 8 — Output of `./scripts/aws-audit.sh` showing your Full Name and all five checks

[Output](screenshots/Script-output.png)

---

#### Screenshot 9 — Output showing the captured exit code and final summary

[Output](screenshots/exit-code.png)

---

### Notes You Must Write (Very Important)

**1. What is the overall status of your baseline audit?**

The overall status of my baseline audit is FAIL.
The script exited with code 2.
Two checks passed, one returned WARN, and two returned FAIL.

**2. Did any check return FAIL or WARN? If so, which one, and what evidence did it show?**

The S3 check returned FAIL. The report showed that bucket pravin-portfolio-ginika-ndubuisi-eu-north-1 has BlockPublicAcls=False and IgnorePublicAcls=False, so public access could be granted through ACLs rather than only through the bucket policy.
The SSH check returned FAIL. The report showed that 3 security groups allow SSH on port 22 from 0.0.0.0/0, meaning anyone on the internet can attempt to connect.
These 3 groups are still attached to other EC2 instances in my account, so I kept them instead of deleting them.
The EBS check returned WARN. The report showed that 2 EBS volumes are not encrypted.

**3. If every check passed, what does that tell you about the security posture of your account so far?**

Not every check passed in my baseline audit.
The MySQL and RDS checks passed, which shows my database is not exposed to the public internet.

---

# Task 6 — Build and Run the /aws-audit Skill

## Goal

Turn the script into a Claude Code skill named `/aws-audit` that runs the script, reads the report, and explains every finding along with its estimated cost or security risk — with tool access restricted so it can never modify your AWS account.

### Evidence

#### Screenshot 10 — `SKILL.md` showing the frontmatter, tool restrictions, and safety rules

[Output](screenshots/Skill.png)

---

#### Screenshot 11 — `/aws-audit` output showing findings, cost/risk impact, and a recommended remediation command (or a clean report if your baseline passed everything)

[Output](screenshots/report1.png)
[Output](screenshots/report2.png)

---

### Notes You Must Write (Very Important)

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill only needs Bash to run the audit script, Read to open the report, and Grep to search it for WARN and FAIL lines. It has no reason to create or change any files.
Leaving out Write means the skill cannot edit the script, change the report, or create a file containing remediation commands. This follows the principle of least privilege, where a tool gets only the permissions its job needs.

**2. What part is performed by Bash, and what part is performed by Claude?**

Bash runs the read-only AWS CLI commands and collects the evidence.
It compares each result and writes PASS, WARN, or FAIL to the report.
Claude reads the report and explains each finding in plain language. Claude also estimates the cost or security risk of each finding and recommends a fix command.

**3. Why is estimating cost/risk impact something the AI adds on top of a plain PASS/FAIL script?**

The audit script only tests fixed conditions and labels each one PASS, WARN, or FAIL. It cannot tell me how serious a finding is or what it could lead to. Claude fills that gap by explaining the risk behind each result, such as SSH open to 0.0.0.0/0 inviting brute-force attempts on my instances, and by estimating any cost impact, such as the price of a breach or of running resources I no longer need. This turns a list of technical results into information I can act on and prioritise.

---

# Task 7 — Fix a Real Finding and Re-Verify

## Goal

Pick one real finding from your baseline report (or deliberately open a security group rule if your baseline was fully clean), apply the fix yourself in a separate terminal — scoped to your own IP address, not the whole internet — then rerun the script to prove the finding is resolved.

### Evidence

#### Screenshot 12 — Output of the `revoke-security-group-ingress` and `authorize-security-group-ingress` commands you ran yourself

[Output](screenshots/revoke.png)
[Output](screenshots/Authorize.png)

---

#### Screenshot 13 — Rerun of `./scripts/aws-audit.sh` showing the finding is now PASS

[Output](screenshots/Rerun.png)

---

### Notes You Must Write (Very Important)

**1. Which exact finding did you fix, and what command did you run?**

I fixed two findings from my baseline audit. The first was the S3 finding, where my bucket pravin-portfolio-ginika-ndubuisi-eu-north-1 had BlockPublicAcls and IgnorePublicAcls set to false. I checked the settings with aws s3api get-public-access-block, then ran aws s3api put-public-access-block to turn both ACL settings on. I left the two policy settings off so my static site could keep working through its bucket policy. The second was the SSH finding, where port 22 was open to 0.0.0.0/0. I used aws ec2 revoke-security-group-ingress to remove the open rule and then aws ec2 authorize-security-group-ingress to allow SSH from my own IP. When I reran the audit, both checks showed PASS.

**2. Why did you scope the new rule to your own IP address instead of leaving it open to `0.0.0.0/0`?**

A rule set to 0.0.0.0/0 lets any device on the internet try to reach my instance over SSH, which leaves it open to scanning and password-guessing attacks. Adding /32 to my public IP means only my machine can connect. I still have the access I need to manage the instance, and the rest of the internet is shut out. This keeps access as narrow as possible, which is the principle of least privilege.

**3. Did Claude execute the remediation command, or did you? Why does that matter?**

I ran the remediation command myself in a separate terminal. Claude’s role was to read the report, explain each finding, and suggest the commands, but the /aws-audit skill has no permission to change anything in my AWS account. This matters since these commands change live infrastructure, and a wrong change could lock me out of an instance or break my website. Running them myself meant I checked each one first and stayed accountable for the result.

**4. Which phase of the Agentic Loop does the Bash script represent? Which phase does Claude's explanation represent? Which phase is you running the fix?**

The Bash script is the Gather phase, as it uses read-only AWS CLI calls to collect evidence about the current state of my account. Claude’s explanation is the Analyze phase, where it reads that evidence, explains the security and cost impact, and proposes a fix. Me running the commands is the Act phase, where a human approves and applies the change. Rerunning the script afterwards completes the loop as the Verify phase, confirming that the fixes worked.

---

# LinkedIn Post (Required)

## Goal

Create a LinkedIn post including:

- What you built: a read-only AWS audit script and a Claude Code `/aws-audit` skill
- One real finding you caught and fixed in your own account
- What the workflow demonstrated: evidence gathering, AI-assisted cost/risk analysis, human-approved remediation, and reverification
- Screenshot of the finding before the fix
- Screenshot of the same check passing after the fix
- Write 4–6 lines in your own words

Suggested tags:

`#DMIByPravinMishra #AWS #AgenticAI #ClaudeCode #DevOps`

### Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_dmibypravinmishra-aws-claudecode-ugcPost-7521025015614517248-y3jF/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### Screenshot of Published LinkedIn Post

[Output](screenshots/Linkedin6.png)

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:

- All 13 required task screenshots
- Answers to every **Notes You Must Write** question
- `CLAUDE.md`
- `scripts/aws-audit.sh`
- `.claude/skills/aws-audit/SKILL.md`
- `reports/aws-audit-report.txt` baseline report and the reverified report from Task 7
- GitHub folder or repository URL containing the assignment files
- Your Full Name visible in the required outputs
- LinkedIn post URL
- Screenshot of the published LinkedIn post
- GitHub repository URL (containing all assignment files)

---

# Completion Checklist

- [ ] Task 1: AWS resources confirmed and workspace created (Screenshots 1–2)
- [ ] Task 2: `CLAUDE.md` created with project context and safety rules (Screenshot 3)
- [ ] Task 3: Claude produced a read-only five-check audit plan before any script existed (Screenshot 4)
- [ ] Task 4: `aws-audit.sh` built, executable, and passes `bash -n` (Screenshots 5–7)
- [ ] Task 5: Baseline audit captured and saved with Full Name visible (Screenshots 8–9)
- [ ] Task 6: `/aws-audit` skill loads and runs successfully with no Write permission (Screenshots 10–11)
- [ ] Task 7: A real finding was fixed by you and reverified as PASS (Screenshots 12–13)
- [ ] Skill never executed a remediation command
- [ ] New security group rule is scoped to your own IP, not `0.0.0.0/0`
- [ ] All 13 required task screenshots are included
- [ ] All "Notes You Must Write" questions are answered in your own words
- [ ] No AWS credentials or unblurred account IDs exposed
- [ ] LinkedIn post published and URL submitted
- [ ] GitHub repository URL included in submission
- [ ] All assignment files committed and visible in GitHub repository

---

# Final Submission

Submit your GitHub repository URL containing all assignment files, screenshots, reports, and output.

### GitHub Repository URL

Paste your GitHub repository URL here:

`Add your GitHub repository URL here`

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*