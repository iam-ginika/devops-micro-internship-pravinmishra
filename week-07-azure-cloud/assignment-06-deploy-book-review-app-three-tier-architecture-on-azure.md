# Assignment 6 — Capstone: Deploy Book Review App (Three-Tier Architecture) on Azure

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a production-ready, best-practice-compliant three-tier architecture on Azure: separated presentation, application, and database tiers, least-privilege network access, a controlled public entry point, protected secrets, and availability/monitoring evidence.

---

# Task 1 — Design the Azure Three-Tier Architecture

## Goal

Create an architecture diagram and implementation plan identifying the presentation, application, and database components, the chosen Azure services, the public entry point, and the internal traffic paths.

### Evidence

#### Screenshot 1 — Architecture diagram showing the public entry point, three tiers, network boundaries, and traffic flow

[Output](screenshots/Diagram.txt)

---

#### Screenshot 2 — Written architecture assumptions and selected Azure services

Region: Sweden Central | Resource Group: Book-Review-RG | VNet: Book-Review-VNet [10.0.0.0/16]

Overview

I built the Book Review App as a three-tier setup on Azure. The Next.js frontend runs on a web VM behind a public load balancer. The Express API runs on a private app VM behind an internal load balancer. The data lives in a MySQL Flexible Server that can’t be reached from the internet. Each tier has its own subnet and its own firewall rules, and a request can only move one tier inward at a time.

Design Decisions and Why I Made Them

Load balancers instead of exposing the VMs directly.
Users reach the app through a Standard Public Load Balancer on port 80. The app tier sits behind an Internal Load Balancer with a private IP only. Both have health probes, so traffic only goes to a VM that’s responding.

VMs for the web and app tiers.
I wanted full control over the OS, Nginx and the Node.js runtime, and VMs carry over directly from my earlier assignments. I used one VM per tier to stay within the free-trial budget. More can be added to the backend pools later without redesigning anything.

Private-only database.
MySQL Flexible Server uses private access (VNet Integration), and public network access is turned off. Only the app subnet can reach it, on port 3306.

Outbound access without inbound exposure.
The private app tier still needs to download OS updates and npm packages. A NAT Gateway gives it outbound access without opening any inbound path.

Credentials kept off GitHub.
The database credentials are in a .env file on the App VM, with restricted file permissions, listed in .gitignore. They never appear in the code, the repo or the screenshots.

Services by Tier

Entry: Azure Load Balancer (Standard, public)
The only public path for app traffic, on port 80.

Web tier: Azure VM (Ubuntu), Book-Review-Web-VM
Runs Nginx to serve the Next.js frontend.

Web to App: Azure Load Balancer (Standard, internal)
Routes requests privately from the web tier to the backend.

App tier: Azure VM (Ubuntu)
Runs the Node.js/Express API, which uses Sequelize to talk to MySQL.

Data tier: Azure Database for MySQL – Flexible Server
Managed MySQL with private access and automated backups.

Network: Book-Review-VNet with 3 subnets
Web subnet [10.0.1.0/24], App subnet [10.0.2.0/24] and DB subnet [10.0.3.0/24], with the DB subnet delegated to MySQL.

Firewalls: Network Security Groups
One NSG per tier, each with least-privilege inbound rules.

Outbound access: NAT Gateway
Lets the private tier download updates and packages without being reachable from the internet.

Visibility: Azure Monitor
Collects metrics and diagnostic logs, and sends alerts.

How a Request Moves Through the System

The user sends a request to the Public Load Balancer on port 80.
The Public Load Balancer forwards it to the Web VM, where Nginx serves the Next.js frontend.
The frontend’s API calls go to the Internal Load Balancer.
The Internal Load Balancer sends them to the App VM running the Express API.
The API reads from and writes to MySQL Flexible Server privately on port 3306.
The only internet-facing path besides the Public Load Balancer is SSH to the Web VM, and that’s restricted to my own IP.

Firewall Rules at Each Boundary
NSG	Allows in	Blocks
Book-Review-Web-NSG	HTTP 80 and HTTPS 443 from Any · SSH 22 from my IP only	All other inbound
App NSG	API port [8080] from the web subnet only	All inbound from the internet
DB NSG	MySQL 3306 from the app subnet only	All inbound from the internet
Keeping It Running and Recoverable
Health probes on both load balancers take an unhealthy VM out of rotation.
MySQL Flexible Server runs automated backups with a [7]-day retention period.
Azure Monitor tracks the VMs, both load balancers and the database, with alerts on key metrics.
I confirmed healthy targets and ran an availability test before submitting.
Build Sequence
Resource group, then the VNet with its web, app and DB subnets
NSGs for each subnet, plus the NAT Gateway and route table for the private tier
MySQL Flexible Server with private access, then the schema import
App VM with the Express API and its .env, then the Internal Load Balancer
Web VM with Nginx and Next.js, then the Public Load Balancer
Azure Monitor diagnostics and alerts
End-to-end test: one read, one write, a check that the private tiers can’t be reached, and an availability test

---

# Task 2 — Create the Azure Network Foundation

## Goal

Create a dedicated Resource Group and VNet with separate subnets for the web, application, and database tiers, keeping the application and database tiers without direct public access.

### Evidence

#### Screenshot 3 — Resource Group overview showing the assignment resources

[Output](screenshots/Resource-group1.png)
[Output](screenshots/Resource-group2.png)

---

#### Screenshot 4 — VNet overview showing the address space and all required subnets

[Output](screenshots/Book-review-vnet.png)
[Output](screenshots/Book-review-subnet.png)

---

#### Screenshot 5 — Route-table or Private DNS evidence where applicable

[Output](screenshots/Route-table.png)

---

# Task 3 — Configure Security and Secret Management

## Goal

Apply least-privilege NSG rules so traffic flows Internet → public entry point → web tier → application tier → database tier, and store credentials in Azure Key Vault or another approved secure mechanism.

### Evidence

#### Screenshot 6 — NSG rules proving least-privilege access between the tiers

[Output](screenshots/DB-NSG.png)
[Output](screenshots/App-NSG.png)
[Output](screenshots/Web-NSG.png)

---

#### Screenshot 7 — Key Vault or approved secret-management configuration (without displaying secret values)

[Output](screenshots/Secret-config.png)

---

# Task 4 — Deploy the Presentation (Web) Tier

## Goal

Deploy the Book Review App presentation layer on the approved web-tier compute service, configured to route requests to the internal application-tier endpoint, and not directly exposed except through the public entry service.

### Evidence

#### Screenshot 8 — Web-tier compute overview showing subnet and availability configuration

[Output](screenshots/Web-VM.png)
[Output](screenshots/Web-tier-subnet.png)

---

#### Screenshot 9 — Terminal or service output proving the presentation layer is running

[Output](screenshots/Service-output.png)

---

# Task 5 — Deploy the Business (Application) Tier

## Goal

Deploy the Book Review App backend privately in the application subnet, configured to use the private database endpoint and secured environment values, reachable only through its internal endpoint.

### Evidence

#### Screenshot 10 — Application-tier compute overview showing private subnet placement

[Output](screenshots/App-tier.png)
[Output](screenshots/App-subnet.png)

---

#### Screenshot 11 — Backend process, service, or listening-port evidence

[Output](screenshots/Listening-port2.png)

---

#### Screenshot 12 — Internal health-check or API response (without exposing secrets)

[Output](screenshots/API-Response.png)

---

# Task 6 — Deploy the Managed Database Tier

## Goal

Create a private Azure managed database (public access disabled), with availability/backup/retention settings, the Book Review App schema imported, and access restricted to the application tier only.

### Evidence

#### Screenshot 13 — Database overview showing private connectivity and public access disabled

[Output](screenshots/Database-network.png)

---

#### Screenshot 14 — Availability, backup, and retention configuration

[Output](screenshots/Database-config.png)

---

#### Screenshot 15 — Successful schema or connectivity verification (without exposing credentials)

[Output](screenshots/Verification.png)

---

# Task 7 — Configure Traffic Management, Availability, and Monitoring

## Goal

Configure the approved public entry service with health probes and backend pools, internal routing for the application tier where required, and enable Azure Monitor/diagnostics/logs/alerts for the key resources.

### Evidence

#### Screenshot 16 — Public entry service showing listener, frontend endpoint, and healthy web targets

[Output](screenshots/Public-LB.png)

---

#### Screenshot 17 — Internal application-tier load-balancing or routing configuration where applicable

[Output](screenshots/Internal-LB.png)

---

#### Screenshot 18 — Azure Monitor, diagnostic settings, logs, metrics, or alert evidence

[Output](screenshots/Monitor.png)

---

# Task 8 — Validate the Production-Style Deployment

## Goal

Confirm the Book Review App works end to end through the public endpoint, with at least one database read and one write, confirm private tiers are not internet-reachable, and complete a safe availability test.

### Evidence

#### Screenshot 19 — Browser showing the Book Review App through the public endpoint

[Output](screenshots/Browser.png)

---

#### Screenshot 20 — Proof of successful database-backed read and write operations

[Output](screenshots/Backend-operation.png)

---

#### Screenshot 21 — Evidence that private tiers are not publicly accessible

[Output](screenshots/App=access.png)

---

#### Screenshot 22 — Availability-test and healthy-target evidence

[Output](screenshots/Health-status.png)

---

#### Public Endpoint

Paste your public endpoint URL here:

http://4.225.191.80/

---

### Notes

Summarize what worked, issues encountered and how they were fixed, and the availability/security/secrets/monitoring/backup choices made.

What worked

The Book Review App is running end to end as a three-tier deployment in Sweden Central. Users reach it only through the Standard Public Load Balancer at http://4.225.191.80/. The Load Balancer forwards traffic to the Web VM, where Nginx serves the Next.js frontend. The frontend's API calls go privately through the Internal Load Balancer to the Express API on the App VM. The API reads from and writes to MySQL Flexible Server over private access on port 3306. Both load balancers report healthy backend targets. I confirmed at least one database read (loading books and reviews) and one write (submitting a review) through the public endpoint. I also confirmed that the App VM and the database can't be reached from the internet.

Issues encountered and how I fixed them

No issues

Availability

- Health probes on both the public and internal load balancers take any unresponsive VM out of rotation.
- Each tier runs one VM to stay within the free-trial budget. 
- I checked availability with the healthy-target status on both load balancers and a test against the public endpoint.

Security

- Each tier has its own subnet and its own NSG with least-privilege inbound rules:
  - Web NSG: HTTP 80 and HTTPS 443 from the internet, and SSH 22 from my IP only
  - App NSG: API port 8080 from the web subnet only
  - DB NSG: MySQL 3306 from the app subnet only
- The App VM and the database have no public IPs. Public network access is disabled on MySQL Flexible Server.
- Traffic can only move one tier inward at a time: Internet → Public LB → Web → Internal LB → App → DB.

Secrets

- The database credentials are stored in a `.env` file on the App VM, readable only by the owner (`chmod 600`). The file is listed in `.gitignore`.
- No passwords, connection strings, keys or subscription IDs appear in the code, the GitHub repo or any screenshot.
- A planned improvement is to move the credentials into Azure Key Vault and have the App VM read them through a Managed Identity.

Monitoring

- Azure Monitor collects metrics for the VMs, both load balancers and the MySQL server.
- Diagnostic settings are enabled on the key resources, and alert rules cover key metrics such as VM CPU, load balancer health probe status and database availability.

Backup

- MySQL Flexible Server runs automated backups with a 7-day retention period, which allows point-in-time restore within that window.

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, keys, connection strings, or subscription IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram and assumptions documented (Screenshots 1–2)
- [ ] Task 2: Network foundation created with isolated tiers (Screenshots 3–5)
- [ ] Task 3: Least-privilege security and secret management configured (Screenshots 6–7)
- [ ] Task 4: Presentation tier deployed (Screenshots 8–9)
- [ ] Task 5: Application tier deployed privately (Screenshots 10–12)
- [ ] Task 6: Managed database tier deployed privately (Screenshots 13–15)
- [ ] Task 7: Public entry, internal routing, and monitoring configured (Screenshots 16–18)
- [ ] Task 8: End-to-end validation and availability test completed (Screenshots 19–22, Public Endpoint, Notes)
- [ ] No sensitive data exposed

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
