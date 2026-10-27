# Assignment 6 — Capstone Assignment — Deploy Book Review App (Three-Tier Architecture) on AWS

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a fully production-style three-tier architecture on AWS: a Next.js Web Tier behind Nginx and a public ALB, a private Node.js/Express App Tier behind an internal ALB, and a private Multi-AZ MySQL RDS database with a read replica. You are expected to design, deploy, isolate, debug, and document the result independently.

---

# Task 1 — Architecture Diagram

## Goal

Create an architecture diagram showing the custom VPC (10.0.0.0/16), the six subnets across two Availability Zones (two public Web Tier, two private App Tier, two private Database Tier), the public ALB, Web Tier EC2/Nginx, internal ALB, private App Tier EC2, private Multi-AZ RDS with its read replica, and the permitted traffic flow.

### Evidence

#### Diagram image or link

[Output](screenshots/Three%20tier.png)

---

# Task 2 — AWS Region & Services Used

## Goal

Record the AWS Region used and list every AWS service used across networking, compute, load balancing, security, and the database.

### Notes

**Region:**

Europe (Stockholm), eu-north-1

---

**Services:**

Networking: Amazon VPC, subnets, Internet Gateway, NAT Gateway, Elastic IP, route tables

Compute: Amazon EC2 (Ubuntu 24.04, t3.micro) for the Web and App tiers

Load balancing: Elastic Load Balancing, with an internet-facing Application Load Balancer, an internal Application Load Balancer, and target groups

Security: Security groups (Web, App, DB), EC2 key pair

Database: Amazon RDS for MySQL, RDS read replica, DB subnet group

---

# Task 3 — Public Entry Point

## Goal

Confirm the Book Review App loads through the public ALB DNS name.

### Evidence

#### Public ALB DNS

Paste your public ALB DNS name here:

http://Book-Review-Web-ALB-426189917.eu-north-1.elb.amazonaws.com

---

# Task 4 — Evidence Screenshots

## Goal

Capture visual proof of every tier and load balancer.

### Evidence

#### Web EC2

[Output](screenshots/web.png)

---

#### App EC2

[Output](screenshots/App.png)

---

#### Public ALB

[Output](screenshots/web-alb.png)

---

#### Internal ALB

[Output](screenshots/app-alb.png)

---

#### RDS + Replica

[Output](screenshots/Databases.png)

---

#### App UI proof

[Output](screenshots/App-ui.png)

---

# Task 5 — Summary

## Goal

Summarize what worked in the final deployment, the issues encountered and how each was fixed, and the tools or sources used to research and debug.

### Notes

**What worked:**

I deployed the Book Review App as a three-tier architecture in a custom VPC (10.0.0.0/16) across two Availability Zones in eu-north-1. Users reach the app through a public ALB, which sends traffic to a Web EC2 running Nginx and the Next.js frontend (managed by PM2). Nginx forwards /api requests to an internal ALB, which routes them to a private App EC2 running the Node.js/Express backend (PM2). The backend connects to a private Multi-AZ RDS MySQL database, and a read replica is provisioned alongside it. The App and Database tiers have no public IP and are reachable only through the tier in front of them, enforced by chained security groups. Registration, login, and the book list all work through the public ALB DNS name.

---

**Issues + fixes:**

Issue: mysql from the App EC2 failed with ERROR 2003 (HY000): Can't connect to MySQL server ... (110).
Fix: Resolved the connection timeout by correcting the security group configuration for the database, then confirmed the backend connected to RDS and returned 200 OK from /api/books.

Issue: The Public ALB DNS name returned 503 Service Temporarily Unavailable.
Fix: The Web target group had no registered targets, so I registered Book-Review-Web-EC2 on port 80.

Issue: sudo nginx -t failed with invalid number of arguments in "proxy_pass" directive in /etc/nginx/sites-enabled/book-review:8.
Fix: Corrected the proxy_pass line in the Nginx config, then nginx -t reported the syntax as ok and the test as successful.

Issue: The homepage showed “No books available” even though /api/books returned data.
Fix: Found with grep that src/app/page.js line 11 called ${process.env.NEXT_PUBLIC_API_URL}/api/books, which became /api/api/books. Changed it to ${process.env.NEXT_PUBLIC_API_URL}/books, then rebuilt and restarted the frontend with pm2.

---

**Tools/sources used:**

AWS Management Console, with the AWS documentation for VPC, ALB, and RDS

The assignment’s step-by-step guide

Terminal tools: SSH/SCP, curl, grep, nano, nginx -t, pm2 logs, systemctl

Browser DevTools and direct /api/books URL tests to isolate failures layer by layer

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post sharing the capstone deployment, including the public ALB DNS (or a redacted screenshot), three to five lines on what you built and why it is production-style, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/feed/update/urn:li:activity:7520491571759431680/

---

#### Screenshot of LinkedIn post

[Output](screenshots/Linkedin5.png)

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, RDS credentials, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram completed
- [ ] Task 2: AWS Region and services documented
- [ ] Task 3: Public ALB DNS confirmed working
- [ ] Task 4: All six evidence screenshots captured (Web Tier, App Tier, both ALBs, RDS + replica, app UI)
- [ ] Task 5: Deployment summary completed (what worked, issues/fixes, tools/sources)
- [ ] LinkedIn post published and URL submitted
- [ ] App Tier and Database Tier confirmed not publicly accessible
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