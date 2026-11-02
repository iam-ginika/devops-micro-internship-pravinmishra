# Assignment 6 — AI-Assisted Terraform Drift and Policy Review

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Student Details

**Full Name:** Ginika Ndubuisi  
**GitHub Repository/Folder URL:** https://github.com/iam-ginika/devops-micro-internship-pravinmishra.git

---

## Purpose

Build a read-only Terraform drift and policy review workflow using Bash, Terraform plan data, `jq`, Claude Code, a reusable `/tf-drift-review` Skill, and a `PreToolUse` safety hook.

The workflow must follow this pattern:

```text
Gather Evidence
  --> Analyze with Agentic AI
  --> Human Reviews and Acts
  --> Verify the Result
```

The `/tf-drift-review` Skill and `tf-drift-check.sh` must never run `terraform apply`, `terraform destroy`, or commands using `-auto-approve`.

---

# Task 1 — Confirm the Clean Baseline and Create the Workspace

## Goal

Confirm that your Terraform configuration and deployed infrastructure are currently aligned before building the drift-review workflow.

## Evidence

### Screenshot 1 — Clean Terraform Plan

Add a screenshot of `terraform plan` showing no pending changes.

[Output](screenshots/t-plan2.png)

---

### Screenshot 2 — Assignment Workspace

Add a screenshot of the folder structure showing `AI Assignment/`, `reports/`, and the Terraform project.

[Output](screenshots/path.png)

## Questions

### 1. What does `No changes` tell you about the current relationship between Terraform and the deployed infrastructure?

It means the Terraform code, the state file and the real infrastructure all match. Nothing has drifted, so Terraform has nothing to create, change or destroy.

### 2. Why is a clean baseline important before introducing a test change?

A clean baseline (No changes) confirms everything is in sync before you change anything. Then, when you make a test change, any change the plan shows comes from your edit alone, not from leftover drift or someone else’s manual changes. That makes the result easy to read, check and roll back.

---

# Task 2 — Create Project Context and Safety Rules in `CLAUDE.md`

## Goal

Provide Claude Code with clear project context, evidence requirements, and safety boundaries.

## Evidence

### Screenshot 3 — Project Context and Safety Rules

Add a screenshot of `CLAUDE.md` open in VS Code showing the Project Overview, Review Workflow, Safety Rules, and Output Rules.

[Output](screenshots/claude-md3.png)

## Questions

### 1. Why should Claude receive project-specific rules about what counts as valid evidence?

Without clear rules, Claude might treat its own assumptions, a confident-sounding summary or a partial result as proof that a task is done. Project-specific rules spell out what actually counts as evidence, for example real terraform plan output, a successful curl response or a screenshot of the deployed resource. That keeps Claude’s claims checkable and stops it from saying something works when it hasn’t been verified.

### 2. Why must the human remain responsible for running `terraform apply`?

terraform apply makes real changes to live infrastructure. It can create costs, delete resources or cause outages, and some of that can’t be undone. The human has to review the plan and make that final call, because they’re accountable for the infrastructure and can judge risk and context that Claude might miss. Claude can suggest the change and explain the plan, but the person approving it should be the one who runs it.

### 3. Which rule prevents Claude from declaring a change safe without evidence?

This is the rule from Safety Rules:

“Do not claim a change is safe unless the available evidence supports that conclusion.”

---

# Task 3 — Build the Terraform Drift and Policy Check Script

## Goal

Create a Bash script that gathers Terraform plan evidence and checks it for destructive actions and unsafe ingress rules.

## Evidence

### Screenshot 4 — Script Variables and Checks Array

Add a screenshot of the top section of `tf-drift-check.sh` showing the variables and `checks` array.

[Output](screenshots/tf-drift.png)

---

### Screenshot 5 — Destructive-Action and Open-Ingress Checks

Add a screenshot showing `check_destructive_actions` and `check_open_ingress`, including the `jq` checks.

[Output](screenshots/destructive-action.png)
[Output](screenshots/open-ingress.png)

---

### Screenshot 6 — Script Validation and Permissions

Add a screenshot showing successful `bash -n` and `ls -l` output.

[Output](screenshots/bash-n.png)

## Questions

### 1. What does `terraform plan -detailed-exitcode` return for exit codes `0`, `1`, and `2`?

0: The plan succeeded and there are no changes. Infrastructure matches the configuration.

1: Error. The plan failed, for example from a syntax error, an auth problem or a provider issue.

2: The plan succeeded and there are changes, so drift or pending updates were detected.

### 2. Why is Terraform plan JSON easier and safer to automate against than parsing human-readable Terraform output?

Plan JSON is structured and machine-readable, with a stable schema. Tools like jq can pull exact fields such as resource_changes[].change.actions reliably. Human-readable output is meant for people: it has colors, symbols and wording that can change between Terraform versions. Parsing it with text matching is fragile and can miss or misread changes.

### 3. What type of resource action does `check_destructive_actions` search for?

It looks for delete actions in the plan’s resource_changes[].change.actions, meaning resources Terraform would destroy.

### 4. Why does finding a `delete` action also help detect replacements?

A replacement shows up in the plan JSON as two actions, ["delete", "create"] or ["create", "delete"]. Because both include delete, one check catches straight deletions and destroy-and-recreate replacements alike. 

### 5. Why must this script never run `terraform apply`?

The script is meant to be a read-only review tool. Its job is to gather evidence and report risks. terraform apply changes real infrastructure and can cause costs, outages or data loss that can’t be undone. That decision belongs to a human who has reviewed the findings, as the project’s safety rules require.

---

# Task 4 — Run the Script Against the Clean Baseline

## Goal

Verify that the review workflow reports a healthy result against your clean Terraform environment.

## Evidence

### Screenshot 7 — Healthy Baseline Report

Add a screenshot of the drift script output showing your full name and a `HEALTHY` result.

[Output](screenshots/report.png)

---

### Screenshot 8 — Baseline Script Exit Code

Add a screenshot showing the captured script exit code `0`.

[Output](screenshots/exit-code.png)

## Questions

### 1. What is the Overall Status of your baseline?

HEALTHY. All 3 checks passed, with 0 warnings and 0 failures, and the script exited with code 0.

### 2. Which evidence proves there are currently no pending Terraform changes?

Terraform’s own message: No changes. Your infrastructure matches the configuration.
The check result: [PASS] terraform plan exit code 0 — no pending changes

### 3. Was `reports/tfplan.json` created? Explain why or why not.

No. The plan returned exit code 0, so there were no pending changes to export. The script only creates the plan JSON when changes exist (exit code 2). That’s why the destructive-action and ingress checks both report No pending plan JSON to inspect: there was nothing to analyze, so both passed.

---

# Task 5 — Create and Run the `/tf-drift-review` Claude Code Skill

## Goal

Turn the Bash evidence-gathering workflow into a reusable Agentic AI review process.

## Evidence

### Screenshot 9 — `/tf-drift-review` Skill Configuration

Add a screenshot of `SKILL.md` showing the frontmatter, allowed tools, and safety rules.

[Output](screenshots/skill.png)

---

### Screenshot 10 — Clean Agentic AI Review

Add a screenshot of `/tf-drift-review` showing the clean `HEALTHY` result.

[Output](screenshots/claude-review.png)

## Questions

### 1. Why does this Skill have `Bash`, `Read`, and `Grep`, but not `Write`?

The skill is a read-only review. Bash runs the drift-check script, and Read and Grep inspect the report and plan JSON. Leaving out Write stops Claude from editing .tf files or changing anything in the project, which enforces the rule “Do not edit any .tf file” at the tool level, not only through instructions.

### 2. Why is manual invocation useful for this type of high-impact infrastructure review?

disable-model-invocation: true means the skill runs only when a human types /tf-drift-review. Claude can’t decide to start it on its own. For high-impact infrastructure work, that keeps the human in control of when the review happens and makes every run intentional.

### 3. Which part of the workflow is deterministic Bash automation?

The tf-drift-check.sh script. It runs terraform plan -detailed-exitcode, exports the plan JSON, checks for delete actions and 0.0.0.0/0 or * ingress rules, and writes the PASS/WARN/FAIL report.

### 4. Which part requires Claude's reasoning?

Interpreting the evidence. That means explaining what the exit code and flagged changes mean, judging how serious a deletion or open ingress rule is in context, giving a plain-language risk assessment, deciding whether apply appears safe, and recommending one next step for the human.

### 5. Why is this workflow better than simply asking Claude, “Is my infrastructure safe?”

A plain question gets an answer based on guesses, with no real data behind it. This workflow makes Claude gather evidence first (actual plan output and JSON), follow fixed safety rules and report in a set format.

---

# Task 6 — Introduce a Controlled Difference and Detect It

## Goal

Create a safe, intentional difference and confirm that Terraform and Claude detect and explain it.

## Evidence

### Screenshot 11 — Controlled Difference

Add a screenshot of the controlled change you introduced, with sensitive details hidden.

[Output](screenshots/manual-change.png)

---

### Screenshot 12 — Detected Difference and Risk Assessment

Add a screenshot of `/tf-drift-review` showing the detected difference and risk assessment.

[Output](screenshots/tf-drift2.png)
[Output](screenshots/assessment.png)

---

### Screenshot 13 — Detected Drift Report

Add a screenshot of `drift-detected-report.txt` showing your full name and the `WARN` or `FAIL` result.

[Output](screenshots/failure.png)

## Questions

### 1. What change did you introduce?

I manually added the tag TestDrift = manual-change to the book-review-dev-web-sg security group in the AWS Console, outside Terraform.

### 2. Was it true infrastructure drift or a Terraform configuration change?

It was a true infrastructure drift. The real resource in AWS was changed directly, while the Terraform .tf files stayed the same. AWS no longer matches what the code describes.

### 3. What Terraform plan evidence proves that a change is pending?

The report shows [WARN] terraform plan exit code 2 — changes are pending and require review. Exit code 2 means the real infrastructure no longer matches the configuration, so a change is pending.

### 4. Was the action an update, deletion, replacement, or security-rule change?

It was an update: an in-place tag change. It wasn’t a deletion, a replacement or a security-rule change. No ingress rules changed and no 0.0.0.0/0 exposure was added.

### 5. What did Claude recommend?

Claude recommended not applying the plan as it stands. It found no deletions and no new public exposure, and the FAIL was a false positive from an existing port 80 rule on the load balancer. Claude suggested removing the TestDrift tag manually and running /tf-drift-review again to confirm exit code 0. If I’d rather fix the drift with Terraform, I should run terraform apply myself, without -auto-approve, after checking the AMI and IAM values.

### 6. Why should you review the recommendation before taking action?

I already made a real change in AWS, so the fix also affects live infrastructure. Reviewing showed that running terraform apply would not only remove the tag. It would also update 7 other resources with values unknown until apply, including possibly a new AMI. 

---

# Task 7 — Add a `PreToolUse` Hook to Block Unsafe Apply Attempts

## Goal

Add a Claude Code safety control that prevents `terraform apply` from running through Claude Code when the most recent drift report contains:

```text
Overall Status: FAIL
```

## Evidence

### Screenshot 14 — `PreToolUse` Safety Hook

Add a screenshot of `.claude/settings.json` showing the `PreToolUse` safety hook.

[Output](screenshots/pretool.png)

---

### Screenshot 15 — Blocked Apply Attempt

Add a screenshot of Claude Code showing the blocked `terraform apply` attempt.

[Output](screenshots/hook2.png)

## Questions

### 1. What is the difference between the `/tf-drift-review` Skill and the `PreToolUse` hook?

The Skill is a workflow I run on purpose. It gathers evidence and has Claude analyze it and recommend a next step. The hook runs automatically before Claude uses a tool and blocks dangerous commands, like terraform apply, if conditions aren’t met.

### 2. Which component performs analysis?

The /tf-drift-review Skill, through Claude’s reasoning over the drift report and plan JSON.

### 3. Which component enforces the safety gate?

The PreToolUse hook.

### 4. Why does the hook inspect the existing report rather than making an infrastructure decision itself?

The hook is meant to be simple and predictable. It just checks the evidence the script already produced, such as the Overall Status, and allows or blocks the command. Interpreting and judging infrastructure is left to the review process and the human.

### 5. Why is a deterministic guard useful for high-impact commands?

It behaves the same way every time and can’t be argued with or talked around. Even if Claude makes a mistake, the guard still blocks unsafe commands like terraform apply before they run.

---

# Task 8 — Resolve the Difference and Verify the Final State

## Goal

Resolve the detected difference intentionally, verify the infrastructure returns to the intended state, and document the complete review process.

## Evidence

### Screenshot 16 — Human-Reviewed Resolution

Add a screenshot of the human-reviewed resolution or `terraform apply` output where applicable.

[Output](screenshots/apply3.png)

---

### Screenshot 17 — Final Healthy Review

Add a screenshot of the final `/tf-drift-review` showing `HEALTHY`.

[Output](screenshots/status.png)

---

### Screenshot 18 — Saved Reports

Add a screenshot of `ls -lah reports` showing both:

- `drift-detected-report.txt`
- `resolved-report.txt`

[Output](screenshots/report2.png)

---

### Screenshot 19 — Drift Review Summary

Add a screenshot of `drift-review-summary.md` showing all required sections and your full name.

[Output](screenshots/report2.png)

## Terraform Drift Review Summary

### 1. Change Introduced

Explain the controlled change you introduced.

State whether it was:

- True infrastructure drift, or
- A Terraform configuration change

I manually added the tag TestDrift = manual-change to the book-review-dev-web-sg security group in the AWS Console, outside Terraform.

This was true infrastructure drift: the real resource in AWS changed while the Terraform configuration stayed the same.

### 2. Evidence Collected

Describe the Terraform plan evidence and affected resource.

- terraform plan -detailed-exitcode returned exit code 2, meaning changes were pending.
- The drift report showed Overall Status FAIL (PASS 1, WARN 1, FAIL 1).
- Affected resource: aws_security_group.web (book-review-dev-web-sg), an in-place update to remove the TestDrift tag.
- No delete or replace actions were found in the plan.

### 3. Risk Assessment

Explain the risk identified by the Bash check and Claude Code.

The Bash check flagged one ingress rule open to 0.0.0.0/0. Claude identified this as a false positive: it was the existing HTTP port 80 rule on the public load balancer security group, which had no pending change.

Claude rated the risk as low to moderate. Nothing was being destroyed or newly exposed, but 7 other resources showed values unknown until apply (including the AMI) as a knock-on effect of the tag change. Claude recommended not running terraform apply as it stood.

### 4. Human-Approved Action

Explain the action you reviewed and executed manually.

I reviewed Claude's recommendation and manually removed the TestDrift tag from the web security group, instead of running terraform apply.

### 5. Verification

Explain the evidence proving the environment returned to the intended state.

I reran the drift check. The resolved report showed exit code 0, "No changes. Your infrastructure matches the configuration.", and Overall Status HEALTHY with all checks passing. The report was saved as reports/resolved-report.txt.

### 6. Safety Decision

Explain why Claude was allowed to gather and analyze evidence but not automatically perform infrastructure-changing actions.

Claude was allowed to gather and analyze evidence because those actions are read-only and cannot change infrastructure. Infrastructure-changing actions such as terraform apply or terraform destroy can cause outages, costs or data loss, so they require human review and approval. This was enforced by the CLAUDE.md safety rules, the Skill's limited tools (no Write), and the PreToolUse hook.

### 7. Agentic Loop Mapping

Explain how your workflow followed:

```text
Gather --> Analyze --> Human Act --> Verify
```

Gather: The /tf-drift-review Skill ran tf-drift-check.sh to collect the Terraform plan, exit code and drift report.

Analyze: Claude reviewed the evidence, identified the tag drift, explained the false-positive FAIL and assessed the risk.

Human Act: I reviewed the recommendation and removed the tag manually.

Verify: I reran the check and confirmed exit code 0 and Overall Status HEALTHY.

## Questions

### 1. What action did you execute to resolve the difference?

I manually removed the TestDrift = manual-change tag from the book-review-dev-web-sg security group.

### 2. Did you review `terraform plan` before taking action?

Yes. The plan showed exit code 2 with an in-place update to remove the tag.

### 3. What evidence proves the environment is now aligned?

The second drift review returned exit code 0, “No changes. Your infrastructure matches the configuration.”, and Overall Status HEALTHY

### 4. Why is a second drift review required after the fix?

To prove the fix actually worked.

### 5. What could go wrong if an AI agent automatically applied every detected Terraform change?

It could delete or replace resources, open security rules, or roll out unintended changes.

### 6. In one sentence, explain the difference between asking an AI chatbot “Is my infrastructure okay?” and using this evidence-based Agentic AI workflow.

A chatbot guesses from no real data, while this workflow gathers actual Terraform evidence, analyzes it under safety rules, and leaves the final action and verification to a human.

---

# LinkedIn Post — Mandatory

## Goal

Publish a LinkedIn post in your own words describing:

- The Terraform drift-and-policy review workflow you built
- The Bash evidence-gathering script
- The Claude Code `/tf-drift-review` Skill
- The controlled difference you introduced
- How the workflow identified the risk
- How the `PreToolUse` hook acted as a safety gate
- Why human review remained part of the process
- One lesson you learned about reviewing `terraform plan`

Include a screenshot of the detected change and a screenshot of the final `HEALTHY` review in your post.

Suggested tags:

```text
#DMIByPravinMishra #Terraform #AgenticAI #ClaudeCode #DevOps
```

## LinkedIn Evidence

### LinkedIn Post URL

https://www.linkedin.com/posts/ginikandubuisi_i-built-an-ai-assisted-terraform-drift-and-share-7522912533444243456-H7Ds/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

### Published LinkedIn Post Screenshot — Mandatory

[Output](screenshots/linkedin2.png)

---

# Required Assignment Files

Confirm that the following files are included in your GitHub repository:

- `CLAUDE.md`
- `AI Assignment/tf-drift-check.sh`
- `.claude/skills/tf-drift-review/SKILL.md`
- `.claude/settings.json` containing the safety hook
- `reports/drift-detected-report.txt`
- `reports/resolved-report.txt`
- `drift-review-summary.md`

---

# Submission Instructions

- Complete Tasks 1–8 in sequence.
- Include Screenshots 1–19 exactly as specified.
- Answer every question under Tasks 1–8 in your own words.
- Complete all seven sections of the Terraform Drift Review Summary.
- Include the GitHub repository/folder URL containing the assignment files.
- Include your full name in the required reports and screenshots.
- Include the LinkedIn post URL and a screenshot of the published LinkedIn post.
- Do not expose access keys, passwords, tokens, account IDs, private keys, Terraform secrets, or other sensitive information.
- Review all screenshots carefully and hide or redact sensitive details where necessary.

---

# Completion Checklist

- [ ] Confirmed a clean Terraform baseline
- [ ] Created the required assignment workspace
- [ ] Created or updated `CLAUDE.md`
- [ ] Added project context and safety rules
- [ ] Created `tf-drift-check.sh`
- [ ] Added my full name to the report
- [ ] Validated the Bash script
- [ ] Made the script executable
- [ ] Used `terraform plan -detailed-exitcode`
- [ ] Used Terraform plan JSON
- [ ] Used `jq` to inspect destructive actions
- [ ] Used `jq` to inspect unsafe ingress
- [ ] Confirmed the baseline returns `HEALTHY`
- [ ] Created `/tf-drift-review`
- [ ] Restricted the Skill to appropriate tools
- [ ] Confirmed the Skill remains read-only
- [ ] Confirmed the Skill never runs `terraform apply`
- [ ] Confirmed the Skill never runs `terraform destroy`
- [ ] Introduced a controlled detectable difference
- [ ] Correctly identified whether it was true drift or a configuration change
- [ ] Saved `drift-detected-report.txt`
- [ ] Added the `PreToolUse` safety hook
- [ ] Verified the hook blocks `terraform apply` when the report is `FAIL`
- [ ] Reviewed the Terraform evidence before resolving the change
- [ ] Performed any infrastructure-changing action manually
- [ ] Ran the drift review again after resolution
- [ ] Confirmed the final status is `HEALTHY`
- [ ] Saved `resolved-report.txt`
- [ ] Completed `drift-review-summary.md`
- [ ] Mapped the workflow to `Gather --> Analyze --> Human Act --> Verify`
- [ ] Included all 19 numbered screenshots
- [ ] Answered all required questions
- [ ] Published the required LinkedIn post
- [ ] Added the LinkedIn post URL and screenshot
- [ ] Included the GitHub repository/folder URL
- [ ] Confirmed that no sensitive information is exposed

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
