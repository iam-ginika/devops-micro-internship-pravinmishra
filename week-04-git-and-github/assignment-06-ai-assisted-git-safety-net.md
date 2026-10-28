# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

[Output](screenshots/New-branch.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

Creating a dedicated branch helps to keep the `main` branch stable and avoid making changes directly to the main codebase. It allows work to be reviewed through a pull request before being merged. It also makes it easier to track changes, identify issues, and collaborate safely with other team members.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

[Output](screenshots/New-file.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

The assignment uses an obviously fake key to prevent the exposure of real credentials and protect sensitive information. It allows students to practise identifying and handling secrets safely without risking unauthorized access to real systems. In real projects, API keys and other credentials should be stored securely as environment variables or in a secrets manager, rather than hardcoded in the code or shared publicly.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

[Output](screenshots/Hook.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

[Output](screenshots/Hookspath.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

Tracking hooks/pre-commit in the repository allows the team to share, review, and maintain the hook through version control. Files inside .git/hooks/ are local to one person's machine and are not included when others clone the repository. Keeping the hook in the repo ensures that everyone can access the same checks and follow the same rules.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

The Git pre-commit hook intercepts a commit attempt before Git creates the commit, allowing checks to run before changes are saved in the repository's history. The PreToolUse hook intercepts a tool call in Claude Code before the tool executes, allowing the action to be checked or blocked.

Both act as checkpoints before an action takes place. They help enforce rules, prevent unwanted actions, and improve safety by checking whether an action should be allowed to proceed.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

[Output](screenshots/Commit-rejected.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

Line 7 matched the fake key:

if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then

It matched because the fake key starts with AKIA followed by 16 uppercase letters or numbers, which matches the regular expression of AWS access key IDs. The hook therefore identified it as a possible secret and blocked the commit.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, the hook would not necessarily catch a secret stored in a poorly named variable if its value did not match any of the patterns in the regular expression. This shows that fixed rules can only detect specific patterns they are designed to recognize. Some secrets may go undetected, so additional security checks and secret-scanning tools are important.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

[Output](screenshots/Skill.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

[Output](screenshots/pr-review.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

`Bash` allows the skill to run read-only Git commands like `git diff --cached` and `git status` to see what is staged. `Read` lets it open files for more context. It does not have `Write` so it cannot change, fix, or create any files. It can only review and suggest. This keeps me in control of the code and follows the principle of least privilege, where a tool only gets the access it needs to do its job.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

No, they did not flag the same things. The pre-commit hook only caught the fake AWS key because it matched the `AKIA` pattern in the regular expression. `/pr-ready` caught the fake key, but it also flagged the debug `echo` statement that printed the key and pointed out that the script had no usage notes explaining its purpose. The hook missed the debug statement because it has no rule for it. However, the hook can actually block the commit every time, while `/pr-ready` can only give advice.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

[Output](screenshots/Commit-success.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

[Output](screenshots/pr-review2.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the hardcoded fake AWS key from `scripts/notify.sh` and replaced it with a line that reads the value from an environment variable instead. I also removed the `echo` line that printed the key. After staging the fixed file, the commit went through with no BLOCKED message. I did not use the placeholder key `/pr-ready` suggested because it still starts with `AKIA` followed by 16 characters, so my hook would have blocked it too.

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

[Output](screenshots/pull-request2.png)

---

#### PR Link

https://github.com/iam-ginika/devops-micro-internship-pravinmishra/pull/1

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I did not edit the drafted description. Before using it, I read it and compared it with the actual change in the "Files changed" tab. It correctly described what the PR does: it adds one standalone file, `scripts/notify.sh`, with no credentials, and the fake AWS key and debug output have been removed. 

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The PR description could have described changes that were not true, which would mislead reviewers. A reviewer might approve a change thinking a security issue was fixed when it was not. Also, if I had followed its suggestion, my own hook would have blocked the commit. 

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

The hook, skill, and demo script are my own practice work and are not meant to be added to the class repository. The upstream repository is shared by everyone in the program, so opening a PR there would add unnecessary noise for the maintainer and other students. It could also lead to my practice files being merged into the shared materials by mistake. Targeting my own fork keeps the shared repository clean.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Gather is when the staged changes are collected. The pre-commit hook reads the staged files with `git diff --cached`, and `/pr-ready` runs Git commands through `Bash` to read the staged diff.

---

**2. Which step(s) represent Analyze?**

Analyze is when the changes are checked. The hook compares the staged content against its secret patterns and checks file sizes. `/pr-ready` reviews the diff, writes a risk report of things that need a second look, and drafts a PR title and description.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is when I fixed the file, ran `git commit` and `git push`, and opened the Pull Request myself. A human must do these steps because they change the project history and make the code public, which is hard to undo. The AI can make mistakes, so a person needs to review the work and take responsibility for what gets committed and shared.

---

**4. Which step is Verify?**

Verify is Task 5, where I committed again after the fix and the hook allowed it, and I ran `/pr-ready` a second time and got a clean report. Checking the PR's base repository and the "Files changed" tab before merging was also part of verifying.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

The hook gives the same result every time and can actually block a commit, but it only catches patterns it was written for. The AI skill can catch things a fixed rule cannot, like debug statements and missing context, but it can make mistakes and cannot enforce anything, so using both gives stronger protection.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/ginikandubuisi_this-devops-assignment-was-about-building-share-7520925874884591617-B-uw/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- Giving an AI skill only read access (no `Write`) lets it review my work without changing anything.
- Working on a feature branch keeps `main` stable, and a Pull Request should target the right base repository.
- Fixed rules are consistent and enforceable, but they only catch the patterns they are written for.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/iam-ginika/devops-micro-internship-pravinmishra

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

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
