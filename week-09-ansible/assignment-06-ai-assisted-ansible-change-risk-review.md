# Assignment 6 — AI-Assisted Ansible Change Risk Review

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build an AI-assisted Ansible risk-review workflow using `ansible-playbook --check --diff`, Bash scripting, and Claude Code.

You will review possible server changes before applying them, classify risky tasks, and keep the final apply decision under human control.

---

# Task 1 — Confirm EpicBook Connectivity and Create the Workspace

## Goal

Confirm that your previous EpicBook Ansible project is working before creating the risk-review automation.

### Evidence

#### Screenshot 1 — Output of `ansible web -i inventory.ini -m ping`

[Output](screenshots/Successful-ping.png)

---

#### Screenshot 2 — Output of `ansible-playbook -i inventory.ini site.yml --syntax-check`

[Output](screenshots/Check4.png)

---

#### Screenshot 3 — Output of `pwd` and `find . -maxdepth 4 -type d | sort`

[Output](screenshots/Workspace-structure.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Ansible can reach your EpicBook VM?**

Running ansible -i inventory.ini web -m ping returned epicbook | SUCCESS with “ping”: “pong”. That shows Ansible connected to the VM over SSH with my key, logged in as ubuntu and ran Python on the server. The full playbook run also finished with unreachable=0.

---

**2. Why should you confirm playbook syntax before building a risk-review script?**

A risk-review script reads the playbook to spot problems. If the YAML is broken, the script would fail or give misleading results, and I wouldn’t know whether a problem was a real risk or just a syntax error. Checking syntax first with --syntax-check means the script starts from a valid playbook and only reports genuine issues.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Create a `CLAUDE.md` file that tells Claude Code how this project must behave.

### Evidence

#### Screenshot 4 — `CLAUDE.md` open in VS Code or terminal showing the safety rules

[Output](screenshots/Claude-md.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude Code have project-specific safety rules?**

Claude Code can run commands and edit files, so without clear limits it could change live infrastructure by mistake. The rules in CLAUDE.md set its role for this project: it gathers evidence and explains risk, but never applies changes. That keeps the workflow read-only and predictable, and stops the AI from taking actions that could break the server or expose secrets.

---

**2. Why should the human run the real Ansible playbook manually?**

Applying changes to a live server is a decision with real consequences, like downtime, data loss or a broken app. The human reviews the risk report, understands the impact, and stays accountable for the decision. Running the playbook manually keeps a person in control of every change that reaches production, instead of an automated tool acting on its own.

---

**3. Which rule prevents Claude Code from applying changes automatically?**

The rule “Never run ansible-playbook without --check.” It means Claude Code can only do dry runs, which show what would change without changing anything. It is backed up by “Never apply, converge, or fix the playbook automatically,” and “Never run ad-hoc commands that change state on the managed VM.”

---

# Task 3 — Ask Claude Code to Plan the Risk Review

## Goal

Use Claude Code to produce a read-only plan before writing the Bash script.

### Evidence

#### Screenshot 5 — Claude Code showing the four-category risk-classification plan

[Output](screenshots/Read-only-plan.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The Gather phase is the dry run: the script runs ansible-playbook with --check --diff against the EpicBook playbook and saves the raw output to a log file. It collects evidence of what would change, including changed tasks, diffs, handlers and the play recap, without changing anything on the VM.

---

**2. Which part represents the Analyze phase?**

The Analyze phase is where the changed tasks pulled from that log are sorted into the four risk categories: service restarts or handlers, firewall changes, user or sudo changes, and package or file removal. Each risky task is flagged with its category and likely impact, and the result goes into a report. Claude Code then reviews the report and explains the risk so I can decide whether to run the real playbook.

---

**3. How did you verify Claude Code did not create or edit files?**

My prompt told Claude Code to only read and propose a plan, without touching any files. Its output showed it ran just one read-only command to look at the playbook’s task names. When it finished, I checked the project folder and confirmed there were no new files and none of the existing files had been modified.

---

# Task 4 — Build the Ansible Risk Review Script

## Goal

Create a Bash script that runs an Ansible dry run and classifies risky changes.

### Evidence

#### Screenshot 6 — Top section of `ansible-check-review.sh` showing `full_name`, `playbook_path`, `inventory_path`, and the `checks` array

[Output](screenshots/Check-review.png)

---

#### Screenshot 7 — Middle section showing `extract_changed_tasks` and `check_tasks_matc                              hing_pattern`

[Output](screenshots/Check-review-middle.png)
[Output](screenshots/Check-review-middle2.png)

---

#### Screenshot 8 — Bottom section showing the loop, summary, and exit behavior

[Output](screenshots/Summary.png)

---

#### Screenshot 9 — Output of `bash -n ansible-check-review.sh` and `ls -l ansible-check-review.sh`

[Output](screenshots/Check-review-list.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the `changed_tasks` array?**

It stores the name of every task that the dry run reported as changed, such as “nginx : Deploy EpicBook Nginx site configuration”. The extract_changed_tasks function fills it, and the check functions then use it to count the changes and test each name against the four risk patterns.

---

**2. Which function finds changed tasks from the Ansible output?**

extract_changed_tasks. It uses awk to read the raw dry-run output, remembers the most recent TASK [...] line, and when a changed: [host] line follows, prints that task line once. sed then strips the TASK [ ] and asterisks, and each clean task name is added to changed_tasks.

---

**3. Why does the script use `--check --diff`?**

--check makes it a dry run, so Ansible reports what would change without changing anything on the EpicBook VM. --diff shows the before-and-after content of files and templates. Together they give real evidence of the changes while keeping the review read-only, which is what CLAUDE.md requires.

---

**4. Why does the script use different exit codes for healthy, warning, and failed results?**

The exit code reports the result in a form other tools can act on without reading the report. 0 means healthy with no changes, 1 means warning because changes exist and need review, and 2 means fail because risky changes were found or the dry run broke. A CI pipeline or another script can use the code to decide whether to continue, pause for review, or stop.

---

# Task 5 — Run the Baseline Dry-Run Review

## Goal

Run the script against your current EpicBook playbook and confirm the baseline risk status.

### Evidence

#### Screenshot 10 — Output of `./ansible-check-review.sh`

[Output](screenshots/Check5.png)

---

#### Screenshot 11 — Output of `echo "Captured Exit Code: $script_exit_code"` and `cat reports/ansible-risk-report.txt`

[Output](screenshots/Exit-code.png)

---

### Notes

Answer the following in your own words:

**1. What was the overall status of your baseline run?**

The overall status was WARN - changes present, review recommended. The dry run completed with a PLAY RECAP showing unreachable=0 and failed=0. The report had 6 PASS, 1 WARN and 0 FAIL.

---

**2. Did any tasks report `changed`?**

Yes, one task: common : Update apt cache. It shows as changed because the apt package cache on the server is older than the one-hour cache_valid_time set in the task, so Ansible would refresh

---

**3. Were any changed tasks flagged as risky?**

No. The one changed task didn’t match any of the four risk categories (service restarts, firewall, user or sudo, or removal), so all four risk checks passed. Refreshing the apt cache only updates the package lists and doesn’t install, remove or restart anything, so it is low risk.

---

**4. What does the script exit code mean?**

The script exited with code 1, which means warning. There are changes that should be reviewed before applying, but none of them are flagged as risky. Code 0 would mean healthy with no changes, and code 2 would mean risky changes were found or the dry run failed.

---

# Task 6 — Create and Run the Claude Code Skill

## Goal

Turn the Bash script into a reusable Claude Code skill called `/ansible-risk-review`.

### Evidence

#### Screenshot 12 — `SKILL.md` showing the frontmatter, allowed tools, and safety rules

[Output](screenshots/Skill.png)

---

#### Screenshot 13 — Claude Code output after running `/ansible-risk-review`

[Output](screenshots/Claude-review.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill allow `Bash`, `Read`, and `Grep`?**

These are the only tools the skill needs. Bash runs the ansible-check-review.sh script, Read opens the risk report and the raw Ansible output, and Grep searches those files for specific tasks or lines. Together they let Claude Code gather and inspect evidence without being able to change anything.

---

**2. Why does this skill not allow file editing?**

The workflow is meant to be read-only. Leaving out Edit and Write means Claude Code can’t change the playbook, roles, inventory, Terraform files or secrets, even by mistake. That backs up the CLAUDE.md rule “Never edit playbook, role, inventory, Terraform, or secret files.”

---

**3. What part is handled by Bash?**

The Bash script handles the gathering and the mechanical checks. It runs ansible-playbook with --check --diff, saves the raw output, pulls out the changed tasks, matches them against the four risk patterns, writes the report, and returns an exit code of 0, 1 or 2.

---

**4. What part is handled by Claude Code?**

Claude Code handles the analysis. It reads the report and raw output, explains each changed task, its risk category and likely real-world impact, and gives one clear recommendation. It then hands the decision back to me to review and run the real playbook only if I choose to.

---

**5. Why is this better than asking Claude Code if the playbook is safe without giving it evidence?**

Without evidence, Claude Code can only guess from the code, and it can’t know what would actually change on the live server. The script gives it real dry-run results from the current server state, so its answer is based on facts rather than assumptions. The report and raw log also make the decision traceable, because I can check the same evidence myself.

---

# Task 7 — Introduce a Controlled Risky Change and Let the Skill Catch It

## Goal

Add a small controlled risky change in your lab playbook and confirm the script and Claude Code catch it before applying.

### Evidence

#### Screenshot 14 — The added risky task inside the role file

[Output](screenshots/Risky-file.png)

---

#### Screenshot 15 — Output of `./ansible-check-review.sh`

[Output](screenshots/Review.png)

---

#### Screenshot 16 — Claude Code `/ansible-risk-review` output showing the risky finding

[Output](screenshots/Claude-review2.png)
[Output](screenshots/Claude-review2.2.png)

---

#### Screenshot 17 — Output of `cat reports/risky-change-report.txt`

[Output](screenshots/report.png)

---

### Notes

Answer the following in your own words:

**1. Which risk category did the added task fall into?**

Package or file removal. The task “common : Remove temporary EpicBook risk test file” has “Remove” in its name and uses state: absent, so the script’s removal check flagged it.

---

**2. What evidence proves the task would change something?**

The --check --diff dry run listed it as a changed task, and the diff showed /tmp/epicbook-risk-test going from state: file to state: absent. That confirms the file currently exists on the VM and would be deleted. The play recap showed changed=2, which includes this task.

---

**3. Did Claude Code apply the playbook?**

No. It only ran the read-only review script with --check --diff, read the reports, and explained the risk. It edited no files, didn’t use sudo, and gave me the real playbook command to run only if I decide to proceed.

---

**4. Why is it important that Claude Code only analyzed the risk?**

Deleting a file can’t be undone, so a human should confirm the path is correct and the change is wanted. Keeping Claude Code to analysis means it explains the impact, but I stay in control of what actually changes on the live server.

---

**5. Which phase of the Agentic Loop is represented by the Bash report?**

The Gather phase. The Bash script ran the dry run and produced the report and raw output as evidence. Claude Code then used that evidence in the Analyze phase to explain the risk and recommend what to do.

---

# Task 8 — Apply as the Human, Verify, and Write the Change Summary

## Goal

Review the risky-change report, apply the playbook manually as the human operator, and verify the result.

### Evidence

#### Screenshot 18 — Output of the real playbook run showing the final recap with `failed=0`

[Output](screenshots/P-Recap.png)

---

#### Screenshot 19 — Output of `ansible web -i inventory.ini -m ping`

[Output](screenshots/Successful-ping2.png)

---

#### Screenshot 20 — Second `/ansible-risk-review` output after applying the change

[Output](screenshots/Status1.png)
[Output](screenshots/Status2.png)

---

#### Screenshot 21 — Output of `ls -lah reports`

[Output](screenshots/Reports.png)

---

#### Screenshot 22 — `change-summary.md` showing all required sections and your Full Name

[Output](screenshots/Change-summary.png)

---

### Notes

Answer the following in your own words:

**1. What command did you run to apply the change for real?**

From the epicbook-prod/ansible folder, I ran:
ansible-playbook -i inventory.ini site.yml --vault-password-file ~/.vault_pass.txt
It ran without --check, so the changes were actually applied, and it finished with failed=0.

---

**2. Who made the final decision to apply the playbook?**

I did, as the human operator. Claude Code only gathered the evidence and explained the risk. I reviewed the report, confirmed the removal only affected my own test file under /tmp, and then decided to run the playbook myself.

---

**3. What evidence proves the VM is still reachable?**

Running ansible web -i inventory.ini -m ping after the apply returned “ping”: “pong” with SUCCESS. The post-apply risk review also completed with unreachable=0 in the recap.

---

**4. Why should the risk review be run again after applying?**

To confirm the change worked and the server now matches the playbook. In the second review the removal task no longer appeared as changed, which proved the file was deleted. It also checks that nothing unexpected changed and that the server is still healthy.

---

**5. What could go wrong if an AI agent applied Ansible changes automatically?**

It could apply a risky or incorrect change without anyone checking it first, such as deleting the wrong files, restarting services during busy hours, locking users out by changing SSH or sudo settings, or wiping database data. Those mistakes can cause downtime or permanent data loss, and no person would have approved or be accountable for the decision.

---

# LinkedIn Post Required

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_the-ai-can-review-the-change-only-i-get-share-7524202283895697408-MfuW/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### Screenshot — Published LinkedIn post

[Output](screenshots/LinkedIn4.png)

---

# Required Files

Confirm that the following files are included in your GitHub repository or assignment folder:

- [ ] `CLAUDE.md`
- [ ] `ansible-check-review.sh`
- [ ] `.claude/skills/ansible-risk-review/SKILL.md`
- [ ] `reports/risky-change-report.txt`
- [ ] `reports/post-apply-report.txt`
- [ ] `change-summary.md`

---

# Submission Instructions

- Add all required screenshots in your submission.
- Full Name must be visible in required screenshots and reports.
- All required notes must be answered clearly.
- Do not expose SSH private keys, passwords, cloud credentials, database credentials, or secret environment variables.
- Add your GitHub repository or folder URL inside this document.

---

# Completion Checklist

- [ ] Task 1: EpicBook connectivity confirmed and workspace created
- [ ] Task 2: `CLAUDE.md` created with safety rules
- [ ] Task 3: Claude Code produced a read-only risk-review plan
- [ ] Task 4: `ansible-check-review.sh` created and syntax checked
- [ ] Task 5: Baseline dry-run review completed
- [ ] Task 6: Claude Code `/ansible-risk-review` skill created and tested
- [ ] Task 7: Controlled risky change introduced and detected
- [ ] Task 8: Human applied the change and verified the result
- [ ] Risky-change report saved
- [ ] Post-apply report saved
- [ ] Change summary completed
- [ ] All screenshots added
- [ ] All notes answered
- [ ] LinkedIn post published
- [ ] LinkedIn post URL added
- [ ] No sensitive information exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra and The CloudAdvisory, focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## Resources

- DMI Official Website: [https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme)
- University: [https://university.pravinmishra.com?utm_source=github&utm_medium=readme](https://university.pravinmishra.com?utm_source=github&utm_medium=readme)
- Discord Community: [https://discord.pravinmishra.com?utm_source=github&utm_medium=readme](https://discord.pravinmishra.com?utm_source=github&utm_medium=readme)
- Blog: [https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*