# Assignment 5 — AI-Assisted Sprint Health Report via Jira MCP

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will connect Claude Code to your Jira board through an MCP server, the same way you connected it to GitHub in Week 2, and build a read-only `/sprint-health` skill. The skill reads your current sprint through Jira's API and reports sprint velocity, stories at risk of missing the sprint, and items missing an estimate — but it must never create, edit, comment on, or transition a single ticket itself. You will prove that boundary holds by making a real change on the board yourself and confirming the skill only ever reports, never acts.

---

# Task 1 — Create a Jira API Token

## Goal

Generate an API token from your Atlassian account that the MCP server will use to authenticate with your Jira site. Do not screenshot the token value itself.

### Evidence

#### Screenshot 1 — Jira API token creation confirmation page showing the token name, with the token value not visible

[Output](screenshots/Api.png)
[Output](screenshots/token.png)

### Notes You Must Write (Very Important):

Why does the MCP server need your site URL and account email in addition to the token?

Jira’s REST API uses Basic Auth, which needs both a username (my email) and a secret (the API token). The token alone can’t say which account it belongs to. The URL matters because Atlassian hosts many separate Jira instances, and mine (my-site.atlassian.net) is distinct from everyone else’s.

---

# Task 2 — Create .mcp.json at the Project Root

## Goal

Create or update `.mcp.json` at your project root with a Jira MCP server block, following the same shape as the GitHub MCP server you configured in Week 2.

### Evidence

#### Screenshot 2 — `.mcp.json` open in VS Code showing the Jira server configuration

[Output](screenshots//mcp.png)

### Notes You Must Write (Very Important):

Compare this jira block to the github block from Week 2 Assignment 5. The GitHub server ran via npx (a Node.js package); this one runs via uvx (a Python package) — what stays exactly the same shape despite that difference, and why doesn't Claude Code care which language a given MCP server is written in?

What stays the same: the shape. Both blocks are mcpServers → <name> → { command, args, env }. Only the values differ:

command is the launcher (npx for Node, uvx for Python). Each one fetches a package and runs it.

args says which package to run (npx needs -y to skip its install prompt; uvx doesn’t).

env holds credentials. Both blocks are empty here, but real use needs tokens: a GitHub PAT, and a Jira URL, email and API token.

Why the language doesn’t matter: Claude Code doesn’t run the server’s code. It starts it as a separate process and talks to it with JSON-RPC messages over stdin/stdout, using the MCP protocol (initialize, tools/list, tools/call). Any program that speaks MCP works, whatever language it’s written in. The contract is the protocol, not the implementation.

---

# Task 3 — Add Your Credentials to settings.local.json

## Goal

Add your Jira site URL, account email, and API token to `.claude/settings.local.json`, and confirm that file is listed in `.gitignore` so it is never committed.

### Evidence

#### Screenshot 3 — `settings.local.json` open in VS Code showing the `env` section, with the actual token value blurred or covered

[Output](screenshots/jira-settings.-local.png)

### Notes You Must Write (Very Important):

Why must JIRA_API_TOKEN live in settings.local.json and never in .mcp.json?

.mcp.json is meant to be shared. It lives in the project and gets committed to git so the whole team gets the same server setup. Anything in it ends up in the repo and its history.

settings.local.json is personal. Claude Code keeps it out of git automatically, so it stays on your machine only.

---

# Task 4 — Verify the Connection with /mcp

## Goal

Restart Claude Code and confirm the Jira MCP server shows as connected.

### Evidence

#### Screenshot 4 — `/mcp` output showing `jira: connected`

[Output](screenshots/Jira-connection.png)

---

# Task 5 — Run a Live Query to Prove Real Board Data

## Goal

Ask Claude to list the issues in your current active sprint through the Jira MCP connection, and confirm the result matches what you see on your live board in the browser.

### Evidence

#### Screenshot 5 — Claude's response showing the live sprint issue list retrieved via Jira MCP

[Output](screenshots/Issues.png)

### Notes You Must Write (Very Important):

How did you confirm this was real board data and not something Claude guessed?

I confirmed the sprint report was real by watching Claude Code call the Jira MCP tools and return raw results from my “DevOps Micro-Internship Website” project instead of answering from its own knowledge. I then opened the active sprint on my Jira board in the browser and compared it with Claude’s output, and the sprint name, issue keys, summaries, issue types, statuses, assignees, story points, and priorities all matched. I also counted the issues and added up the story points myself, and my totals matched Claude’s. These checks showed that the data came from my live Jira board through the MCP connection and was not guessed or made up.

---

# Task 6 — Build the /sprint-health Skill

## Goal

Create a `/sprint-health` skill restricted to read-only Jira tools plus `Read`, with no issue-mutating tools and no `Write`. Run it and confirm it produces a report covering sprint velocity, at-risk stories, and items missing an estimate.

### Evidence

#### Screenshot 6 — `SKILL.md` frontmatter showing `allowed-tools` limited to read-only Jira tools plus `Read`, with `disable-model-invocation: true`

[Output](screenshots/sprint-health.png)

#### Screenshot 7 — `/sprint-health` output showing the full triage report against your real sprint

[Output](screenshots/sprint-health-report.png)

### Notes You Must Write (Very Important):

1. Which Jira MCP tools does this skill's allowed-tools list include, and which mutating tools (create issue, update issue, transition issue, add comment) does it deliberately exclude?

The allowed-tools list includes four read-only Jira MCP tools: jira_search, jira_get_issue, jira_get_sprint, and jira_get_board. It also includes Read, which is a local file tool. It deliberately excludes the mutating tools jira_create_issue, jira_update_issue, jira_transition_issue, and jira_add_comment, so the skill can view the board but never change it.


2. Why does a Scrum Master need this restriction more than almost any other role in this course?

A Scrum Master facilitates the team but does not own its work. The developers own their stories and status updates. The board is the team’s shared source of truth. If the skill changed an issue on its own, it could misrepresent progress across the whole sprint and undermine the team’s self-organization. Keeping it read-only means it only reports, and humans decide what to change.


---

# Task 7 — Prove the Skill Never Mutates the Board

## Goal

Manually update one ticket on your board in the browser (for example, move a story to "Done" or add a missing estimate), then run `/sprint-health` again and confirm the new report reflects your change — proving the skill only ever reads live state and never wrote to the board itself.

### Evidence

#### Screenshot 8 — Second `/sprint-health` run showing the report now reflects your manual board change

[Output](screenshots/sprint-health-report2.png)

### Notes You Must Write (Very Important):

Map this assignment to Gather → Analyze → Human Act → Verify from Week 3 Assignment 6. Which step did you perform manually in the browser, and why must that step stay human?

1. Gather: I ran /sprint-health, and it pulled the live data from my active Jira sprint through the read-only Jira MCP connection.
2. Analyze: The skill went through the sprint data and showed me the velocity so far, the stories at risk, and the items missing estimates.
3. Human Act: I went into Jira in the browser and made the change myself, either by updating the ticket’s status or adding the missing estimate. This step has to stay human because it changes the real project board. That decision needs human judgment and should come from the person accountable for the work, not from an AI tool.
4. Verify: I ran /sprint-health again, and the new report showed my change. This proved that the skill only reads what is on the board and does not change anything in Jira itself.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post including a screenshot of your `/sprint-health` report.

Write a caption in your own words explaining:
- What you built using Claude Code and Jira MCP
- What you learned about connecting AI to live sprint data
- How the sprint-health report helps identify issues that need attention
- Why you restricted the assistant to read-only access and kept board changes under human control

Include these hashtags:

#DMIByPravinMishra #AgenticAI #ClaudeCode #DevOps #Agile

Add the credit note for your track, as provided in the assignment guidelines, and replace `YOUR-GITHUB-USERNAME` with your GitHub username.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_another-hands-on-project-completed-in-my-share-7520927625486417920-tPuX/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### Screenshot 9 — Published LinkedIn post showing the caption and sprint-health report image

[Output](screenshots/LinkedIn4.png)

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 9 required screenshots
- All the required notes

---

# Completion Checklist

- [ ] Task 1: Jira API token created, value never screenshotted (Screenshot 1)
- [ ] Task 2: `.mcp.json` has the Jira server block (Screenshot 2)
- [ ] Task 3: Credentials stored in `settings.local.json`, token blurred, file gitignored (Screenshot 3)
- [ ] Task 4: `/mcp` shows the Jira server connected (Screenshot 4)
- [ ] Task 5: Live query returned real sprint data, verified against the browser (Screenshot 5)
- [ ] Task 6: `/sprint-health` skill created with correct read-only `allowed-tools`, and produced a full report (Screenshots 6–7)
- [ ] Task 7: A manual board change was reflected in a second `/sprint-health` run (Screenshot 8)
- [ ] Skill never created, edited, transitioned, or commented on any issue
- [ ] Reflection answered (Notes)
- [ ] No API token value exposed
- [ ] LinkedIn Post URL
- [ ] LinkedIn Screenshot 1

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
