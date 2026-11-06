# Assignment 02 — Provision Linux VMs with Terraform and Run Ansible Ad-Hoc Commands

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will use Terraform to provision three or four Ubuntu Linux Virtual Machines on either Microsoft Azure or Amazon Web Services.

You will configure SSH key-based authentication, organize the servers using a custom Ansible inventory, and run Ansible ad-hoc commands across individual hosts and inventory groups.

---

# Task 1 — Create the Multi-Host Lab Structure

## Goal

Create a separate project directory for the multi-host lab and prepare the Terraform, Ansible, and documentation files.

This project will use the Git repository and Ansible controller prepared in Assignment 01.

### Evidence

#### Screenshot 1 — Terminal showing the complete `ansible-adhoc-lab` project structure

[Output](screenshots/lab.png)

---

#### Screenshot 2 — Terminal showing `git status --short` with the new project files and updated `.gitignore`

[Output](screenshots/git-status.png)

---

### Notes

For this lab I set up a new project folder, ansible-adhoc-lab, inside my Ansible workspace. I organized it by tool:

- terraform/ holds the files that define and provision the infrastructure: providers, variables, main resources, and outputs.
- ansible/ holds inventory.ini, which lists the hosts Ansible will manage.

 I updated .gitignore to include Terraform state, plans and crash logs while excluding lock artifacts, the Python virtual environment, SSH private keys, and other generated or sensitive files. That keeps credentials and machine-specific data out of the repository.

---

# Task 2 — Create the Terraform Configuration

## Goal

Create the Terraform configuration required to provision three or four Ubuntu Linux VMs on your selected cloud platform.

Complete only one option:

- Option A — Microsoft Azure
- Option B — Amazon Web Services

Do not configure both providers for this assignment.

### Evidence

#### Screenshot 3 — Terraform configuration showing the three or four server roles and the `for_each` or `count` implementation

[Output](screenshots/variable.png)

---

#### Screenshot 4 — Terraform configuration showing SSH restricted to the controller IP and HTTP allowed only for web hosts

[Output](screenshots/main.png)

---

#### Screenshot 5 — Terraform output configuration showing how public IP addresses are associated with the server roles

[Output](screenshots/output.png)

---

### Notes

I used Amazon Web Services (AWS) in the eu-north-1 (Stockholm) region and chose the three-VM option, with the server roles web1, app1, and db1. I defined these roles in a vm_roles variable and used Terraform for_each to create one Ubuntu 24.04 LTS EC2 instance (t3.micro) per role from a single resource block, so I didn’t need a separate block for each server.

The configuration creates a dedicated VPC (vpc-ansible-adhoc-lab), a public subnet, an Internet Gateway, and a route table with a default route to the internet. It also creates an AWS key pair (ansible-adhoc-key) from my existing SSH public key. I never referenced the private key in Terraform.

For security, I used two security groups. The common group, attached to every server, allows SSH on port 22 only from my Ansible controller’s public IP as a /32 CIDR, never from 0.0.0.0/0. A second group allows HTTP on port 80 and is attached only to the web host. I also added a validation rule so Terraform rejects any SSH source that isn’t a single /32 address.

Each instance is tagged with its role name, and cloud-init sets the Linux hostname to match (web1, app1, db1). Finally, I set up a public_ips output that maps each server role to its public IP address, which I’ll use for SSH testing and the Ansible inventory.

Challenge: AWS rejected my first security group names because a group name can’t start with sg-. I renamed the groups internally and kept the required sg-ansible-adhoc-lab name as the Name tag shown in the AWS console.

---

# Task 3 — Provision the Infrastructure with Terraform

## Goal

Initialize and validate the Terraform configuration, review the execution plan, provision the selected three or four VMs, and retrieve their public IP addresses.

### Evidence

#### Screenshot 6 — Final `terraform apply` output showing `Apply complete`

[Output](screenshots/apply.png)

---

#### Screenshot 7 — `terraform output public_ips` showing the role-to-IP mapping for all three or four VMs

[Output](screenshots/ip-output.png)

---

#### Screenshot 8 — Azure Portal or AWS Management Console showing all three or four VMs in the `Running` state, with their role-based names visible

[Output](screenshots/instances.png)

---

### Notes

I went to the Terraform working directory, ran terraform fmt and terraform fmt -check to format the configuration, and then ran terraform init to download the AWS provider. terraform validate confirmed the configuration was valid.

Next I ran terraform plan and checked the plan before provisioning anything. It showed three Ubuntu EC2 instances (web1, app1, and db1) in the eu-north-1 region, all using the t3.micro instance type. It also showed SSH restricted to my controller IP (105.112.212.240/32) and HTTP allowed only for the web server. In total it planned 11 resources: the VPC, subnet, Internet Gateway, route table and association, two security groups, the key pair, and the three instances. No existing resources would be changed or destroyed.

Once I’d confirmed the plan, I ran terraform apply, which completed with 11 resources added. I used terraform output public_ips to get the public IP address for each server role. In the AWS Management Console, all three instances showed as Running with their role-based names.

---

# Task 4 — Verify SSH Key-Based Access

## Goal

Verify that each managed VM can be accessed from the Ansible controller using SSH key-based authentication.

### Evidence

#### Screenshot 9 — Terminal showing successful SSH hostname output from all VMs

[Output](screenshots/hostname.png)

---

### Notes

I retrieved the public IP addresses of the three VMs with terraform output public_ips, which maps each address to its server role.

I then tested SSH access from my Ansible controller to each server as the ubuntu user, the default user on AWS Ubuntu images. The connections used my existing SSH key pair (~/.ssh/id_ed25519), whose public key Terraform attached to every instance through the ansible-adhoc-key AWS key pair. No remote password was needed. On the first connection to each server, I accepted the host fingerprint.

On each VM I ran hostname, which returned web1, app1, and db1. That confirms key-based SSH access works for all three servers and that each server’s hostname matches its role, as set through cloud-init in the Terraform configuration. SSH worked because the security group allows port 22 only from my Ansible controller’s public IP, using a /32 CIDR.

---

# Task 5 — Create the Custom Ansible Inventory

## Goal

Create an Ansible inventory file that groups the managed VMs by role.

The inventory allows Ansible to run commands against all servers, or only specific groups such as `web`, `app`, or `db`.

### Evidence

#### Screenshot 10 — `inventory.ini` showing the `web`, `app`, and `db` groups

[Output](screenshots/inventory.png)

---

#### Screenshot 11 — Output of `ansible-inventory -i inventory.ini --graph`

[Output](screenshots/graph.png)

---

### Notes

I created inventory.ini and grouped the servers by role: web1 under web, app1 under app, and db1 under db. Each host’s public IP is set with ansible_host. Under [all:vars], Ansible is set to connect as the ubuntu user with my existing SSH key, ~/.ssh/id_ed25519. Running ansible-inventory -i inventory.ini --graph showed each host in its correct group.

---

# Task 6 — Run Ansible Ad-Hoc Commands

## Goal

Run Ansible ad-hoc commands from the controller to verify connectivity, check server information, and manage packages and services across inventory groups.

This task proves that the inventory is working and that Ansible can control multiple managed VMs without writing a playbook.

### Evidence

#### Screenshot 12 — Output of `ansible all -i inventory.ini -m ping`

[Output](screenshots/ping.png)

---

#### Screenshot 13 — Output of `ansible all -i inventory.ini -m command -a "uptime"`

[Output](screenshots/uptime.png)

---

#### Screenshot 14 — Output of `ansible web -i inventory.ini -m apt -a "name=nginx state=present update_cache=yes" --become`

[Output](screenshots/nginx.png)

---

#### Screenshot 15 — Output of `ansible web -i inventory.ini -m service -a "name=nginx state=started enabled=yes" --become`

[Output](screenshots/service.png)

---

#### Screenshot 16 — Output of `ansible all -i inventory.ini -m apt -a "name=htop state=present update_cache=yes" --become`

[Output](screenshots/update.png)

---

#### Screenshot 17 — Output of `ansible web -i inventory.ini -m command -a "systemctl is-active nginx"`

[Output](screenshots/nginx-status.png)

---

### Notes

ansible web -i inventory.ini -m command -a "systemctl is-active nginx

---

# LinkedIn Post Required

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_devops-terraform-ansible-share-7524193133681664000-CldD/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### Screenshot — Published LinkedIn post

[Output](screenshots/LinkedIn.png)

---

# Assignment Questions

Answer the following in your own words:

**1. What is the purpose of an Ansible inventory file?**

The inventory file tells Ansible which servers it manages and how to reach them. It lists each host with its IP address (through ansible_host) and holds shared connection settings, like the SSH user and private key. It also lets you put hosts into groups, so you can run a command on one server, one group, or every server at once without typing IP addresses each time.

---

**2. What is the difference between the `web`, `app`, and `db` groups in your inventory?**

Each group stands for a server role. web holds web1, the public-facing server that runs Nginx and is the only one with HTTP (port 80) open. app holds app1, which would run the application logic. db holds db1, which would host the database. Grouping by role lets me target a single tier. For example, I installed Nginx only on the web group and installed htop on all.

---

**3. What does the Ansible `ping` module verify?**

A SUCCESS with “ping”: “pong” means the host is fully ready for Ansible to manage.

---

**4. Why do package installation commands require `--become`?**

Installing packages with apt and managing system services both need root privileges. I connect as the regular ubuntu user, so --become tells Ansible to escalate with sudo for that command. Without it, the apt and service tasks fail with a permission error.

---

**5. When would you use an ad-hoc command instead of a playbook?**

I’d use an ad-hoc command for quick, one-off tasks: checking connectivity, checking uptime or disk space, restarting a service, or installing a single package across a group. For anything I need to repeat, keep in version control, or that has several steps that depend on each other, a playbook is the better choice because it is reusable and documented.

---

**6. What is one challenge you faced while setting up SSH or inventory, and how did you fix it?**

The main challenge was keeping SSH locked down while still being able to reach all three servers. My security group allows port 22 only from my Ansible controller’s public IP as a /32, not from 0.0.0.0/0, so only my machine can connect. The downside is that if my public IP changes, SSH gets blocked and Ansible can’t reach the servers. To fix it, I update the controller IP variable in Terraform and run terraform apply again, which updates the security group rule.

---

# Required Files

Confirm that the following files are included in your assignment workspace:

- [ ] `ansible-adhoc-lab/README.md`
- [ ] `ansible-adhoc-lab/terraform/providers.tf`
- [ ] `ansible-adhoc-lab/terraform/main.tf`
- [ ] `ansible-adhoc-lab/terraform/variables.tf`
- [ ] `ansible-adhoc-lab/terraform/outputs.tf`
- [ ] `ansible-adhoc-lab/ansible/inventory.ini`
- [ ] Updated `.gitignore`

---

# Submission Instructions

- Add all required screenshots from the tasks.
- Full Name must be visible in required screenshots.
- Mention whether you used Azure or AWS.
- Mention whether you used the three-VM option or four-VM option.
- Add the public IP addresses of the VMs, redacted if preferred.
- Add your `inventory.ini` proof.
- Add a short explanation of what you learned.
- Answer all assignment questions clearly in your own words.
- Add your LinkedIn post URL.
- Do not expose SSH private keys, Terraform state files, cloud credentials, passwords, access keys, secret keys, account IDs, or subscription IDs.

---

# Completion Checklist

- [ ] Task 1: `ansible-adhoc-lab` project structure created
- [ ] Task 1: `.gitignore` updated for Terraform files
- [ ] Task 2: Terraform configuration created
- [ ] Task 2: Server roles defined for either three or four VMs
- [ ] Task 2: `count` or `for_each` used
- [ ] Task 2: SSH restricted to the controller public IP
- [ ] Task 2: HTTP allowed only for web hosts
- [ ] Task 2: Terraform output maps roles to public IPs
- [ ] Task 3: Terraform initialized successfully
- [ ] Task 3: Terraform configuration validated
- [ ] Task 3: Terraform apply completed successfully
- [ ] Task 3: All selected VMs are running
- [ ] Task 4: SSH key-based access works for every VM
- [ ] Task 5: `inventory.ini` contains `web`, `app`, and `db` groups
- [ ] Task 5: `ansible-inventory -i inventory.ini --graph` shows the correct groups
- [ ] Task 6: `ansible all -i inventory.ini -m ping` returns `SUCCESS`
- [ ] Task 6: Ad-hoc commands run successfully
- [ ] Task 6: `--become` was used for package and service tasks
- [ ] Task 6: Nginx is active on the `web` group
- [ ] Screenshots 1–17 are included
- [ ] Assignment questions are answered
- [ ] LinkedIn post published
- [ ] LinkedIn post URL added
- [ ] No sensitive information is exposed

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