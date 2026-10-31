# Capstone Assignment — Deploy the Book Review App Using Terraform and Claude Code Agentic AI

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Student Details

**Full Name:** Ginika Ndubuisi  
**Cloud Platform:** AWS 
**GitHub Repository URL:** https://github.com/iam-ginika/devops-micro-internship-pravinmishra
**Public Application URL / Load-Balancer DNS:** http://book-review-dev-public-alb-1385436621.eu-west-1.elb.amazonaws.com/


---

## Purpose

Deploy the Book Review App using Terraform on AWS or Azure in a secure, highly available, production-style three-tier architecture. Use Claude Code, specialized subagents, Terraform MCP, and validation hooks to support the engineering workflow while keeping all infrastructure-changing operations under human control.

---

# Task 0 — Prepare the Project and Agentic AI Environment

## Goal

Prepare the Book Review App project and configure the provided Claude Code Agentic AI starter kit with project context, specialized subagents, Terraform MCP, validation hooks, and safety guardrails.

## Evidence

### Screenshot 1 — Project `CLAUDE.md`

Add a screenshot of the project `CLAUDE.md` showing the three-tier architecture, security boundaries, Terraform requirements, and human-approval rules.

[Output](screenshots/claude-md.png)
[Output](screenshots/claude-md2.png)

---

### Screenshot 2 — Terraform Engineer Subagent

Add a screenshot showing the Terraform Engineer subagent configuration.

[Output](screenshots/engineer.png)

---

### Screenshot 3 — Architecture and Security Reviewer Subagent

Add a screenshot showing the Architecture and Security Reviewer subagent configuration.

[Output](screenshots/architecture.png)

---

### Screenshot 4 — Terraform MCP Connection

Add a screenshot showing Terraform MCP connected and available.

[Output](screenshots/mcp.png)

---

### Screenshot 5 — Validation Hooks

Add a screenshot showing the configured Claude Code validation hooks.

[Output](screenshots/hook.png)


---

# Task 1 — Design the Three-Tier Architecture

## Goal

Design the required secure, highly available three-tier architecture and create an architecture diagram before building the infrastructure.

The diagram must show:

- VPC or VNet
- Availability Zones or equivalent availability locations
- Six subnets
- Internet connectivity
- NAT or outbound design
- Public load balancer
- Web Tier
- Internal load balancer
- Application Tier
- Managed MySQL
- Read replica
- Main traffic flow

## Architecture Diagram

[Output](screenshots/3-tier-diagram.png)

---

# Task 2 — Build the Terraform Networking and Security Layers

## Goal

Create the modular Terraform project and implement the network and security layers across the required public and private subnets.

## Evidence

### Screenshot 6 — Modular Terraform Project Structure

Add a screenshot showing the modular Terraform project structure.

[Output](screenshots/structure.png)

---

### Screenshot 7 — Six-Subnet Architecture

Add a screenshot showing the six-subnet architecture across two availability locations.

[Output](screenshots/Subnets.png)


---

### Screenshot 8 — Public and Private Tier Separation

Add a screenshot showing the public and private tier separation, including routing and security boundaries.

[Output](screenshots/route-tabls.png)

---

# Task 3 — Build the Load-Balancing and Compute Layers

## Goal

Deploy the public and internal load balancers and the Web and Application compute resources required by the Book Review App.

## Evidence

### Screenshot 9 — Web and Application Compute

Add a screenshot showing the Web and Application compute resources in their required subnets.

[Output](screenshots/instances.png)

---

### Screenshot 10 — Public Load Balancer

Add a screenshot showing the internet-facing public load balancer.

[Output](screenshots/public-alb.png)

---

### Screenshot 11 — Internal Load Balancer

Add a screenshot showing the private internal load balancer.

[Output](screenshots/private-alb.png)

---

### Screenshot 12 — Healthy Targets

Add a screenshot showing healthy target groups or backend pools.

[Output](screenshots/targets.png)

---

# Task 4 — Build the Managed MySQL Database Layer

## Goal

Deploy a private, highly available managed MySQL database with a read replica and restrict database connectivity to the Application Tier.

## Evidence

### Screenshot 13 — Managed MySQL Database

Add a screenshot showing the managed MySQL database deployment.

[Output](screenshots/database.png)

---

### Screenshot 14 — High Availability

Add a screenshot showing the Multi-AZ or high-availability configuration.

[Output](screenshots/multi-az.png)

---

### Screenshot 15 — Read Replica

Add a screenshot showing the read replica configuration.

[Output](screenshots/replica.png)

---

### Screenshot 16 — Private Database Access

Add a screenshot showing that the database is private and accepts MySQL traffic only from the Application Tier.

[Output](screenshots/access.png)

---

# Task 5 — Validate, Review, and Apply the Terraform Configuration

## Goal

Validate the Terraform configuration, review the execution plan using both Agentic AI and human judgment, and apply the infrastructure changes only after all required checks pass.

## Evidence

### Screenshot 17 — Terraform Validation

Add a screenshot showing successful `terraform validate` output.

[Output](screenshots/t-validate.png)

---

### Screenshot 18 — Terraform Plan

Add a screenshot showing the Terraform plan output.

[Output](screenshots/t-plan.png)

---

### Screenshot 19 — Terraform Apply

Add a screenshot showing successful `terraform apply` completion.

[Output](screenshots/t-apply.png)

---

# Task 6 — Deploy and Configure the Book Review Application

## Goal

Deploy and configure the Book Review App across the Web, Application, and Database tiers and verify the complete application functionality.

## Evidence

### Screenshot 20 — Homepage

Add a screenshot showing the Book Review App homepage through the public endpoint.

[Output](screenshots/browser3.png)

---

### Screenshot 21 — Login or Authentication

Add a screenshot showing successful login or authentication.

[Output](screenshots/login.png)

---

### Screenshot 22 — Book Data

Add a screenshot showing the book listing or book details.

[Output](screenshots/book-r.png)

---

### Screenshot 23 — Review Functionality

Add a screenshot showing the review functionality working successfully.

[Output](screenshots/review.png)

---

### Screenshot 24 — Backend or API Evidence

Add a screenshot showing that the backend or API is working successfully.

[Output](screenshots/bankend.png)

---

### Screenshot 25 — Database Reads and Writes

Add a screenshot showing successful database reads and writes.

[Output](screenshots/database2.png)

## Public Application URL

**Public Application URL / DNS:** http://book-review-dev-public-alb-1385436621.eu-west-1.elb.amazonaws.com/

---

# Task 7 — Demonstrate the Agentic AI Workflow

## Goal

Demonstrate how Claude Code assisted with Terraform generation, architecture and security review, and evidence-based troubleshooting while infrastructure-changing decisions remained under human control.

You do not need to submit your complete Claude Code conversation history. Include only focused evidence.

## Evidence

### Screenshot 26 — AI-Assisted Terraform Generation

Add a screenshot showing one useful example of AI-assisted Terraform generation or improvement.

[Output](screenshots/AI.png)

---

### Screenshot 27 — Architecture or Security Review

Add a screenshot showing one structured architecture or security review result.

[Output](screenshots/proposed-sg.png)

---

### Screenshot 28 — AI-Assisted Troubleshooting

Add a screenshot showing one AI-assisted troubleshooting interaction based on collected evidence.

[Output](screenshots/troubleshooting.png)

---

# Task 8 — Complete the Final Architecture Review

## Goal

Review the completed infrastructure against the original capstone requirements and resolve significant architecture, security, reliability, and cost issues.

Confirm that the final review covers:

- Tier separation
- Availability
- Public exposure
- Routing
- Security rules
- Load balancing
- Database privacy
- Secrets
- Terraform quality
- Module structure
- Reliability
- Obvious cost risks

Use Screenshot 27 as the focused evidence for the structured architecture or security review.

---

# Task 9 — Answer the Reflection Questions

## Goal

Reflect on the architecture, Terraform implementation, and Agentic AI workflow. Answer each question briefly in your own words.

## Architecture

### 1. Why did you separate the Web, Application, and Database tiers?

I separated the Web, Application, and Database tiers so that each tier has one job and its own security rules. The web tier serves users, the application tier runs the backend, and the database tier stores the data. Each tier only accepts traffic from the tier in front of it, so if one tier is compromised, the others are still protected. It also makes each tier easier to scale and troubleshoot on its own.

### 2. Why is the Application Tier private?

The Application Tier is private because the backend never needs to be reached directly from the internet. The app servers have no public IP and only accept port 3001 from the internal ALB. Users reach the API through the public ALB and Nginx, which keeps the backend and its secrets off the internet.

### 3. Why is MySQL private?

MySQL is private because it holds all the users and reviews, so it must be the hardest part to reach. It sits in private subnets with no internet route, and it only accepts port 3306 from the application tier. Not even the web tier can connect to it.

### 4. Why are multiple Availability Zones used?

Multiple Availability Zones are used so the app keeps running if one AZ fails. The subnets, both ALBs, the EC2 instances and the RDS database are spread across eu-west-1a and eu-west-1b. One trade-off: I used a single NAT gateway to save cost, so it is a single point of failure for outbound internet traffic.

### 5. What is the difference between Multi-AZ/high availability and a read replica?

The difference is that Multi-AZ is for availability, and a read replica is for performance. Multi-AZ keeps a standby copy in another AZ that takes over automatically if the primary fails, but it cannot be used for reads. A read replica is a separate copy with its own endpoint that the app can read from to reduce load on the primary, but it does not fail over automatically.

## Terraform

### 6. How did you divide your Terraform into modules?

I divided my Terraform into five modules, one for each layer: network, security, load-balancer, database and compute. The root main.tf calls each module, variables.tf holds the settings, and outputs.tf exposes values I need later, such as the ALB DNS names.

### 7. How do the modules communicate through variables and outputs?

The modules communicate by passing outputs from one module into another module's input variables in the root main.tf. For example, the network module outputs vpc_id and the subnet IDs, which the security, load-balancer, database and compute modules use. The security module outputs the security group IDs, and the load-balancer module outputs the target group ARNs for the Auto Scaling groups.

### 8. What did you specifically check in `terraform plan`?

In terraform plan, I checked:

* that the number of resources to add, change and destroy was what I expected
* where 0.0.0.0/0 was used, to make sure nothing internal was open to the internet
* that port 3001 and port 3306 were only open to the correct security groups
* that nothing important would be replaced, especially the RDS database
* that no passwords or secrets appeared in the plan

## Agentic AI

### 9. What was the purpose of `CLAUDE.md`?

The purpose of CLAUDE.md was to give Claude the project rules to follow every time. It defined the architecture, the Terraform conventions, the build order, the security rules and the troubleshooting steps. It also required human approval before any apply, and it is why Claude flagged requests that broke the rules.

### 10. What work did the Terraform Engineer subagent perform?

The Terraform Engineer subagent designed and built each phase: network, security, load balancing, database and compute. It checked the app code for the real ports and settings, used the Terraform MCP server for current documentation, wrote the Terraform code, and ran fmt, validate and plan. It also fixed the missing read replica and wrote the deployment runbook.

### 11. What did the Architecture and Security Reviewer identify?

The Architecture and Security Reviewer identified a bug in the bootstrap scripts that stopped setup right after cloning the repo (git "dubious ownership"). It also flagged that the site uses HTTP only, that the single NAT gateway is a single point of failure, and that the instances needed an IAM role for SSM. It confirmed that the IAM permissions were tightly scoped and that ports 3001 and 3306 were not exposed.

### 12. Why did you use Terraform MCP instead of relying only on Claude's existing Terraform knowledge?

I used Terraform MCP because the AWS provider changes often and Claude's built-in knowledge can be out of date. MCP let Claude check the current documentation, for example the newer security group rule resources and the write-only password arguments that keep secrets out of Terraform state.

### 13. What was the purpose of your validation hooks?

The purpose of my validation hook was to check Terraform automatically after every change Claude made. Each time Claude edited or wrote a file, the hook ran terraform fmt and terraform validate, so errors were caught immediately instead of at plan time.

### 14. Describe one real issue Claude helped you troubleshoot.

One real issue Claude helped me troubleshoot was the read replica not being created. Claude checked CloudTrail and found that AWS rejected the replica because RDS was managing the master password, which is not supported for MySQL read replicas. It was fixed by storing the password in my Secrets Manager secret, turning off RDS password management, and then creating the replica again. It worked.

### 15. Describe one recommendation you reviewed, modified, or rejected instead of accepting blindly.

One recommendation I modified was the guideline's NEXT_PUBLIC_API_URL setting. It pointed to the internal ALB on port 3001, but the frontend calls the API from the user's browser, and a browser cannot reach an internal load balancer. I pointed it to the public ALB with /api instead, and Nginx forwards those requests to the internal ALB. I also limited SSH to my own IP instead of opening port 22 to everyone.

---

# Task 10 — Publish the Mandatory LinkedIn Post

## Goal

Publish a LinkedIn post describing the capstone, the technical work completed, the Agentic AI workflow, and the lessons learned.

Write the post in your own words, include at least one project image or other proof, and ensure that it can be viewed by the submission reviewer.

## LinkedIn Post URL

**LinkedIn Post URL:** linkedin.com/posts/ginikandubuisi_capstone-completed-another-big-step-forward-share-7522325294972657664-vimS/?highlightedUpdateUrn=urn%3Ali%3Aactivity%3A7522325298302885888&highlightedUpdateType=SOCIAL_SHARE&origin=SOCIAL_SHARE&utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

# Submission Instructions

- Complete Tasks 0–10 in sequence.
- Include all Screenshots 1–28 exactly as specified.
- Ensure that your full name is visible in the required screenshots.
- Include the selected cloud platform.
- Include the completed architecture diagram.
- Include the modular Terraform project structure.
- Include the working public application URL or public load-balancer DNS.
- Include all required Agentic AI workflow evidence.
- Answer all 15 reflection questions briefly in your own words.
- Include the published LinkedIn post URL.
- Do not expose cloud credentials, database passwords, SSH private keys, JWT secrets, access tokens, account IDs, Terraform state containing sensitive values, or other confidential information.
- Review all screenshots and project files carefully before submitting through GitHub.

---

# Completion Checklist

- [ ] Selected AWS or Azure
- [ ] Added and reviewed the Agentic AI starter files
- [ ] Configured `CLAUDE.md`
- [ ] Configured the Terraform Engineer subagent
- [ ] Configured the Architecture and Security Reviewer subagent
- [ ] Connected Terraform MCP
- [ ] Configured validation hooks and safety guardrails
- [ ] Created the architecture diagram
- [ ] Created the six-subnet design
- [ ] Configured public Web Tier routing
- [ ] Kept the Application Tier private
- [ ] Kept the Database Tier private
- [ ] Configured tier-specific Security Groups or NSGs
- [ ] Restricted backend port `3001`
- [ ] Restricted MySQL port `3306` to the Application Tier
- [ ] Created the public load balancer
- [ ] Created the internal load balancer
- [ ] Configured listeners and health checks
- [ ] Deployed the Web Tier compute resources
- [ ] Deployed the private Application Tier compute resources
- [ ] Provisioned private managed MySQL
- [ ] Configured Multi-AZ or high availability
- [ ] Configured a read replica
- [ ] Created the modular Terraform project
- [ ] Used variables, outputs, and module dependencies
- [ ] Used current Terraform documentation through MCP
- [ ] Used hooks for deterministic validation
- [ ] Completed `terraform fmt`
- [ ] Completed `terraform validate`
- [ ] Reviewed `terraform plan`
- [ ] Completed the Terraform Engineer review
- [ ] Completed the Architecture and Security review
- [ ] Applied the infrastructure only after human approval
- [ ] Deployed and configured the backend
- [ ] Deployed and configured the frontend
- [ ] Configured Nginx where required
- [ ] Configured the internal backend endpoint
- [ ] Configured the public frontend endpoint
- [ ] Verified the homepage
- [ ] Verified login or authentication
- [ ] Verified book data
- [ ] Verified review functionality
- [ ] Verified the backend API
- [ ] Verified database reads and writes
- [ ] Verified healthy load-balancer targets
- [ ] Included AI-assisted Terraform generation evidence
- [ ] Included one architecture or security review
- [ ] Included one AI-assisted troubleshooting example
- [ ] Completed the final architecture review
- [ ] Answered all 15 reflection questions
- [ ] Published the mandatory LinkedIn post
- [ ] Added the LinkedIn post URL
- [ ] Captured all 28 required screenshots
- [ ] Confirmed that my full name is visible in the required screenshots
- [ ] Checked that no secrets or sensitive information are exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory), focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations through hands-on experience.

---

## Resources

- Book Review App Repository: [https://github.com/pravinmishraaws/book-review-app](https://github.com/pravinmishraaws/book-review-app)
- DMI Official Website: [https://dmi.pravinmishra.com](https://dmi.pravinmishra.com)
- University: [https://university.pravinmishra.com](https://university.pravinmishra.com)
- Discord Community: [https://discord.pravinmishra.com](https://discord.pravinmishra.com)
- Blog: [https://dmi.pravinmishra.com/blog](https://dmi.pravinmishra.com/blog)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra on LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory on LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
