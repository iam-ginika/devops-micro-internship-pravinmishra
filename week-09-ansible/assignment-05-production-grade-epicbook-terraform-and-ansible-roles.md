# Assignment — Deploy EpicBook with Terraform and Ansible Roles

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will deploy the EpicBook web application using Terraform and Ansible roles.

Terraform provisions the cloud infrastructure, including one Ubuntu VM and one managed MySQL database. Ansible roles configure the VM, install required software, deploy the EpicBook application, configure Nginx, connect the app to the managed MySQL database, and verify the deployment.

---

# Task 1 — Set Up the Project Folder Layout

## Goal

Create the project folder structure for Terraform and Ansible roles.

Terraform will be used to provision the cloud infrastructure. Ansible roles will be used to configure the VM and deploy the EpicBook application.

### Evidence

#### Screenshot 1 — Terminal showing the completed `epicbook-prod` project structure

[Output](screenshots/project_structure.png)

---

### Notes

Answer the following in your own words:

**1. Which cloud provider did you choose for this assignment?**

AWS

---

**2. Why is it useful to keep Terraform files and Ansible files in separate folders?**

They do different jobs. Terraform provisions the infrastructure: servers, networks, security groups. Ansible configures what runs on those servers once they exist: packages, services, app setup. Keeping them in separate folders makes that boundary clear. Each tool keeps its own files (Terraform state and `.tf` files, Ansible inventory, playbooks and roles) without mixing, so the project is easier to navigate, debug and maintain. It also lets you run, change or reuse one part without touching the other. For example, you can re-run an Ansible playbook without touching the infrastructure, or swap the Terraform setup while keeping the same configuration code.

---

**3. What is the purpose of the `roles` directory in Ansible?**

The `roles` directory breaks configuration into reusable, self-contained units. Each role groups everything for one job, like installing Nginx or setting up a database, into a standard structure: `tasks`, `handlers`, `templates`, `files`, `vars` and `defaults`. Playbooks stay short and readable because they just call roles instead of listing every task. The same role can also be reused across projects, environments or servers without rewriting it.

---

# Task 2 — Provision the Infrastructure with Terraform

## Goal

Run Terraform to provision the cloud infrastructure for the EpicBook deployment.

Terraform will create the VM, managed MySQL database, networking, security rules, and required outputs.

### Evidence

#### Screenshot 2 — `terraform apply` completed successfully

[Output](screenshots/apply2.png)

---

#### Screenshot 3 — Output of `terraform output`

[Output](screenshots/T-output.png)

---

#### Screenshot 4 — Azure Portal or AWS Console showing the VM running

[Output](screenshots/VM.png)

---

#### Screenshot 5 — Azure Portal or AWS Console showing the managed MySQL database created

[Output](screenshots/Database.png)

---

### Notes

Answer the following in your own words:

**1. What resources did Terraform create for this assignment?**

Terraform created 15 resources in AWS (eu-north-1), organised into three modules:

- **Network:** a VPC (10.0.0.0/16), one public subnet for the web server, two private subnets in different availability zones for the database, an internet gateway, a public route table and its subnet association.
- **Security:** a web security group allowing HTTP from anywhere and SSH only from my IP, and an RDS security group that only accepts MySQL traffic (port 3306) from the web server's security group, plus its ingress and egress rules.
- **Compute:** an SSH key pair and an Ubuntu EC2 instance (t3.micro), with a user data script that installs Nginx, Node.js, Git and the MySQL client.
- **Database:** a DB subnet group and a private MySQL 8.0 RDS instance (db.t3.micro) with the database name `epicbook`.


---

**2. Why should you review `terraform plan` before running `terraform apply`?**

`terraform plan` is a preview. It shows exactly what Terraform is about to create, change or destroy before anything happens in the cloud. Reviewing it lets me catch mistakes while they're still free to fix: a wrong region, a wrong database name, an outdated IP in a security group rule, or a resource marked for replacement or deletion that I didn't expect. Once `apply` runs, those changes are real. They can cost money, cause downtime or delete data. In this assignment the plan showed `15 to add, 0 to change, 0 to destroy`, which confirmed it would only create new resources and not touch anything existing.

---

**3. Why should database passwords not be shown in Terraform output?**

Anything Terraform prints can end up in places it shouldn't: terminal history, screenshots, CI/CD pipeline logs, or shared with teammates. If the password appears there, anyone who sees it could access the database and the application's data. Marking the password `sensitive = true` makes Terraform hide it as `(sensitive value)` in plan and output, while still passing it correctly to the database. 

---

# Task 3 — Verify SSH Key-Based Access

## Goal

Verify that the cloud VM can be accessed from the Ansible controller using SSH key-based authentication.

### Evidence

#### Screenshot 6 — Successful SSH hostname check from the Ansible controller

[Output](screenshots/ssh.png)

---

### Notes

Answer the following in your own words:

**1. What command did you use to verify SSH access?**

`ssh -i ~/.ssh/terraform-aws-vm-key ubuntu@13.60.86.210 "hostname"`

It connects to the EC2 instance as the `ubuntu` user with my private key, runs the `hostname` command on the server, and prints the result back to my terminal without opening an interactive session.


---

**2. What proves that SSH key-based access worked successfully?**

The command printed the server's hostname (its private hostname, in the format `ip-10-0-1-xx`) and never asked me for a password. That proves three things: the server was reachable on port 22, AWS accepted my private key because it matched the public key Terraform uploaded as the `epicbook-key` key pair, and the command ran on the remote server itself, since the hostname shown belongs to the EC2 instance, not my own machine.

---

**3. What would you check if SSH returned `Permission denied (publickey)`?**

That error means the server was reachable but rejected my key. First I would confirm I’m using the right username, which is ubuntu for Ubuntu AMIs, not ec2-user or root. Next I would check that the -i flag points to the private key (not the .pub file) and that it matches the public key Terraform uploaded. I would also check the permissions, because SSH refuses to use a private key unless it is set to chmod 600 and the ~/.ssh folder is 700. Then I would confirm in the EC2 console that the instance was launched with the epicbook-key key pair. Finally, I would run ssh -v to see which keys are being offered and exactly where authentication fails.

---

# Task 4 — Create the Ansible Inventory and Configuration

## Goal

Create the Ansible inventory file and local Ansible configuration for the EpicBook VM.

The inventory tells Ansible which VM to manage and which SSH user to use.

### Evidence

#### Screenshot 7 — `inventory.ini` showing the VM under the `web` group

[Output](screenshots/Inventory2.png)

---

#### Screenshot 8 — Output of `ansible-inventory -i inventory.ini --graph`

[Output](screenshots/Ansible-read.png)

---

#### Screenshot 9 — Output of `ansible web -i inventory.ini -m ping`

[Output](screenshots/Epicbook-success.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `inventory.ini`?**

It tells Ansible which servers to manage and how to connect to them. It groups servers (like `[web]`) and stores connection details such as the IP address, SSH user and key, so playbooks know where to run.

---

**2. What does `ansible_host` store?**

It stores the actual IP address or hostname Ansible connects to. In my inventory, epicbook is just a friendly name, and ansible_host=13.60.86.210 is the real public IP of my EC2 instance.

---

**3. What does `ansible_ssh_private_key_file` tell Ansible?**

It tells Ansible which private key to use when logging into the server over SSH. Mine points to /home/ginny/.ssh/terraform-aws-vm-key, the pair to the public key Terraform uploaded to AWS.

---

**4. Why is `host_key_checking = False` used only for this temporary lab?**

It stops Ansible from asking me to confirm the server’s identity on first connection, which is convenient when servers are created and destroyed often in a lab. In production it’s unsafe, because it also skips the check that protects against connecting to a fake server (a man-in-the-middle attack). Real environments should keep host key checking on.

---

# Task 5 — Create the Main Ansible Playbook

## Goal

Create the main Ansible playbook that runs the required roles in the correct order.

The `site.yml` file will call the `common`, `nginx`, and `epicbook` roles.

### Evidence

#### Screenshot 10 — `site.yml` showing the roles in the correct order

[Output](screenshots/Site.png)

---

#### Screenshot 11 — Output of `ansible-playbook -i inventory.ini site.yml --syntax-check`

[Output](screenshots/Syntax-check.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `site.yml`?**

It is the main playbook that ties everything together. It tells Ansible which hosts to target (the web group), whether to use elevated privileges, and which roles to run and in what order. Running this one file configures the whole server.

---

**2. Why should the roles run in the order `common`, `nginx`, and `epicbook`?**

Each role depends on the one before it. The common role prepares the server first by updating packages and installing basic tools. The nginx role then installs and configures the web server. The epicbook role comes last because the application needs those packages and Nginx already in place before it can be deployed and served. Running them out of order could cause failures because something the app needs wouldn’t exist yet.

---

**3. What does `become: true` allow Ansible to do?**

It lets Ansible run tasks with root (sudo) privileges on the server. This is needed for admin tasks like installing packages, editing system config files, and starting or restarting services, which the normal ubuntu user can’t do on its own.

---

# Task 6 — Create the `common` Role

## Goal

Create the `common` role to prepare the Ubuntu VM with the basic packages required for the EpicBook deployment.

This role handles the common server setup before Nginx and the application are configured.

### Evidence

#### Screenshot 12 — `roles/common/tasks/main.yml` showing the common setup tasks

[Output](screenshots/Main-yml.png)

---

### Notes

Answer the following in your own words:

**1. What is the responsibility of the `common` role?**

It prepares the server with the basic setup every server needs, no matter what it will run. That includes updating the package cache, upgrading packages, and installing general tools like curl, git and the MySQL client. It gives the other roles a clean, ready base to build on.

---

**2. Why should Nginx installation not be placed inside the `common` role?**

Nginx is specific to web servers, not something every server needs. Keeping it in its own role keeps each role focused on one job, makes the code easier to read and troubleshoot, and lets me reuse the common role on other servers (like a database or worker server) without installing a web server they don’t need.

---

**3. Why is `mysql-client` useful in this deployment?**

The EpicBook database runs on a separate RDS instance, not on the EC2 server. The mysql-client lets the server connect to that remote database from the command line, so I can test the connection, check the database exists, and load the schema or seed data the app needs.

---

# Task 7 — Create the `nginx` Role

## Goal

Create the `nginx` role to install Nginx and configure it as a reverse proxy for the EpicBook application.

Nginx will receive browser traffic on port `80` and forward it to the EpicBook Node.js application running on the VM.

### Evidence

#### Screenshot 13 — `roles/nginx/tasks/main.yml` showing Nginx installation and site configuration tasks

[Output](screenshots/Nginx-main.png)

---

#### Screenshot 14 — `roles/nginx/templates/epicbook.conf.j2` showing the reverse proxy configuration

[Output](screenshots/Template.png)

---

### Notes

Answer the following in your own words:

**1. What is the responsibility of the `nginx` role?**

It installs and configures Nginx as the web server for EpicBook. It deploys the EpicBook site configuration from a template, enables it, removes the default site, checks the configuration is valid with nginx -t, and makes sure Nginx is running and starts on boot. When the configuration changes, it reloads Nginx through a handler.

---

**2. Why is Nginx configured as a reverse proxy in this deployment?**

The EpicBook app runs on Node.js on an internal port (not port 80), so users shouldn’t reach it directly. Nginx listens on port 80 for public traffic and forwards requests to the app behind it. This keeps the app port private and lets Nginx handle web traffic efficiently. It also gives one place to add things like HTTPS, caching or load balancing later without changing the app.

---

**3. Why should the application port come from `group_vars/web.yml` instead of being hard-coded?**

Keeping the port in a variable means it is defined in one place. If it changes, I update group_vars/web.yml once and every template and task that uses it picks up the new value automatically. That avoids mismatches between the Nginx config and the app, and makes the roles reusable across environments that run the app on different ports.

---

# Task 8 — Create the `epicbook` Role

## Goal

Create the `epicbook` role to deploy the EpicBook application, connect it to the managed MySQL database, and run the application on port `8080` using PM2.

### Evidence

#### Screenshot 15 — `roles/epicbook/tasks/main.yml` showing application deployment tasks

[Output](screenshots/Epicbook-main.png)

---

#### Screenshot 16 — Task or file showing how the database connection is configured, with secrets hidden

[Output](screenshots/Configuration.png)

---

#### Screenshot 17 — Task or output showing the EpicBook application managed by PM2

[Output](screenshots/playbook.png)

---

### Notes

Answer the following in your own words:

**1. What is the responsibility of the `epicbook` role?**

It deploys the EpicBook application itself. It creates the app directory, clones the code from GitHub, installs the Node.js dependencies, writes the .env file with the database connection settings, imports the SQL files into the RDS database if it is still empty, and starts the app with PM2 so it keeps running.

---

**2. Why is PM2 used for the EpicBook Node.js application?**

If the app were started with a plain node server.js, it would stop as soon as the terminal closed or the app crashed. PM2 runs it in the background as a managed process, restarts it automatically if it crashes, and saves the process list so it can come back after a reboot. It also gives simple commands to check the app’s status and logs, like pm2 list and pm2 logs.

---

**3. Why should database passwords not be hard-coded in public files?**

Anything committed to a public repository can be read by anyone, and it stays in the Git history even after it’s deleted. A leaked password would let someone access or wipe the database. Storing it in Ansible Vault keeps it encrypted, so the files can be shared safely and only someone with the vault password can read it.

---

**4. What does it mean for the application to run on port `8080` while Nginx listens on port `80`?**

The Node.js app runs internally on port 8080, and Nginx sits in front of it on port 80, the standard HTTP port users reach in their browser. When someone visits the server’s IP, Nginx receives the request on port 80 and forwards it to the app on port 8080, then sends the response back. Users never connect to the app directly, and port 8080 doesn’t need to be open to the internet.

---

# Task 9 — Create Group Variables

## Goal

Create reusable variables for the EpicBook deployment.

The `group_vars/web.yml` file stores values that can be reused across the Ansible roles.

### Evidence

#### Screenshot 18 — `group_vars/web.yml` showing the application, PM2, and database variables, with passwords hidden or masked

[Output](screenshots/web-yml.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `group_vars/web.yml`?**

It holds the shared variables for all servers in the web group, so the roles use variables instead of hard-coded values. If something changes, I update it in one place.

---

**2. Which values did you store in `group_vars/web.yml`?**

The app settings (repo, install path, user and port), the PM2 app name, the Nginx server name, and the database host, name and user. The password is only referenced as {{ vault_db_password }}.

---

**3. How did you handle the database password securely?**

I encrypted it with Ansible Vault in a separate vault.yml file and only referenced it in web.yml. The playbook decrypts it at runtime with --ask-vault-pass, and no_log: true keeps it out of the output.

---

# Task 10 — Run the Ansible Playbook

## Goal

Run the Ansible playbook to configure the VM and deploy the EpicBook application.

The playbook should run the roles in this order:

1. `common`
2. `nginx`
3. `epicbook`

### Evidence

#### Screenshot 19 — Ansible playbook output showing the roles running

[Output](screenshots/Running-playbook.png)

---

#### Screenshot 20 — Final Ansible recap showing `failed=0`

[Output](screenshots/Fianl-recap.png)

---

#### Screenshot 21 — Output of `ansible web -i inventory.ini -m command -a "systemctl is-active nginx" --become`

[Output](screenshots/Check2.png)

---

#### Screenshot 22 — Output of `ansible web -i inventory.ini -m command -a "pm2 status"`

[Output](screenshots/Pm2-status.png)

---

#### Screenshot 23 — Output of `ansible web -i inventory.ini -m command -a "curl -I http://localhost:8080"`

[Output](screenshots/Local-host.png)

---

### Notes

Answer the following in your own words:

**1. What command did you run to execute the Ansible playbook?**

ansible-playbook -i inventory.ini site.yml --ask-vault-pass

The --ask-vault-pass flag prompts for the vault password so Ansible can decrypt the database password at runtime.

---

**2. How do you know all roles completed successfully?**

The PLAY RECAP showed failed=0 and unreachable=0. Every task in the common, nginx and epicbook roles finished as ok or changed, and the only skipped task was the SQL import, because the database tables already existed.

---

**3. What proves that Nginx is active?**

I ran ansible web -i inventory.ini -m command -a “systemctl is-active nginx” --become and it returned active with rc=0. That confirms the Nginx service is running on the server.

---

**4. What proves that PM2 is managing the EpicBook application?**

I ran ansible web -i inventory.ini -b -m command -a “pm2 status” and it showed a process named epicbook with status online. That confirms PM2 started the app and is keeping it running in the background.

---

**5. What proves that the EpicBook application responds on port `8080`?**

I ran ansible web -i inventory.ini -m command -a “curl -I http://localhost:8080” and it returned HTTP/1.1 200 OK with X-Powered-By: Express. That shows the Node.js app is listening on port 8080 and serving the EpicBook page.

---

# Task 11 — Verify the EpicBook Deployment

## Goal

Verify that the EpicBook application is running, accessible in the browser, and connected to the managed MySQL database.

### Evidence

#### Screenshot 24 — Output of `curl -I http://<public_ip>`

[Output](screenshots/public-ip.png)

---

#### Screenshot 25 — Output of the cart API test command

[Output](screenshots/Cart.png)

---

#### Screenshot 26 — Output of the `/cart` HTTP status check

[Output](screenshots/cart-output.png)

---

#### Screenshot 27 — Browser showing the EpicBook application loaded from `http://<public_ip>`

[Output](screenshots/Browser3.png)

---

### Notes

Answer the following in your own words:

**1. What HTTP response did you receive from the public application URL?**

Running curl -I http://13.60.86.210 returned HTTP/1.1 200 OK, with Server: nginx and X-Powered-By: Express. That shows the request reached Nginx on port 80 and was forwarded to the EpicBook app.

---

**2. What did the cart API test prove?**

Sending a POST to /api/cart with {“bookId”:1} returned 200 OK and JSON containing the book “28 Summers” with a quantity of 1. That proved the full stack works end to end: Nginx forwarded the request, the Express app handled it, and the app read from and wrote to the RDS MySQL database.

---

**3. What did the `/cart` status check return?**

It returned 200, meaning the cart page loads successfully through Nginx.

---

**4. What issue did you face during verification, and how did you fix it?**

Requests to the public IP returned 404 Not Found from Nginx, with no X-Powered-By: Express header, even though curl localhost:8080 on the server worked. The Nginx config was correct, but my first playbook run had failed at the database import task, and when a play fails Ansible skips its pending handlers. So the Reload nginx handler never ran, and Nginx kept serving its old default config. I fixed it by reloading Nginx with ansible web -i inventory.ini -b -m service -a “name=nginx state=reloaded”, and the public URL then returned 200 OK.

---

# LinkedIn Post Required

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_a-404-error-taught-me-something-about-ansible-share-7524200010503188480-gc1V/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### Screenshot — Published LinkedIn post

[Output](screenshots/LinkedIn3.png)

---

# Assignment Questions

Answer the following in your own words:

**1. Why is Terraform used for infrastructure provisioning?**

Terraform defines the infrastructure as code, so the VPC, subnets, security groups, EC2 instance and RDS database are created the same way every time. It shows a plan before making changes, tracks what it created in a state file, and can rebuild or destroy everything with one command.

---

**2. Why are Ansible roles useful for production-style deployments?**

Roles split the configuration into focused, reusable parts (common, nginx and epicbook), each with one job. That keeps the playbook short and readable, makes problems easier to trace, and lets the same role be reused on other servers or projects.

---

**3. What is the purpose of `group_vars/web.yml`?**

It holds the shared variables for the web servers, like the app repo, port, PM2 name and database settings. The roles use these variables instead of hard-coded values, so changes are made in one place.

---

**4. Why should database passwords not be committed to GitHub?**

Anything pushed to GitHub can be read by others and stays in the Git history even after it’s deleted. A leaked password would give someone access to the database. I stored mine encrypted in Ansible Vault and kept terraform.tfvars in .gitignore.

---

**5. What is the purpose of Nginx in this deployment?**

Nginx is the reverse proxy. It receives public traffic on port 80 and forwards it to the Node.js app on port 8080, so the app is never exposed directly to the internet.

---

**6. Why should the managed MySQL database not be publicly accessible?**

A public database can be scanned and attacked from anywhere on the internet. Mine sits in private subnets with publicly_accessible set to false, and its security group only allows port 3306 from the web server’s security group.

---

**7. Why is PM2 used for the EpicBook Node.js application?**

PM2 runs the app in the background, restarts it if it crashes, and saves the process list so it can come back after a reboot. It also gives simple commands to check status and logs.

---

**8. What does idempotency mean in Ansible?**

Running the same playbook several times gives the same result without repeating work or breaking anything. For example, my SQL import only runs when the database has no tables, so later runs skip it instead of loading the data twice.

---

**9. What issue did you face during the deployment, and how did you fix it?**

The database import failed because the EpicBook SQL files hard-code a database called bookstore, both in USE statements and in the table names, while my database was named epicbook. I updated the import task to replace every bookstore with {{ db_name }} and to run the schema file before the seed files. Because that first run failed, the Nginx reload handler never ran either, so the public URL returned 404. I fixed that by reloading Nginx manually.

---

**10. What security improvement would you make before using this setup in production?**

I would add HTTPS with an SSL certificate, and move the database password into AWS Secrets Manager. I would also run PM2 as a non-root user instead of root.

---

# Required Files

Confirm that the following files are included in your GitHub repository or assignment folder:

- [ ] `README.md`
- [ ] Terraform files under either `terraform/azure/` or `terraform/aws/`
- [ ] `ansible/ansible.cfg`
- [ ] `ansible/inventory.ini`
- [ ] `ansible/site.yml`
- [ ] `ansible/group_vars/web.yml`
- [ ] `ansible/roles/common/tasks/main.yml`
- [ ] `ansible/roles/nginx/tasks/main.yml`
- [ ] `ansible/roles/nginx/templates/epicbook.conf.j2`
- [ ] `ansible/roles/epicbook/tasks/main.yml`

---

# Submission Instructions

- Add all required screenshots in your submission.
- Full Name must be visible in required screenshots.
- Mention the cloud provider used: Azure or AWS.
- Add the VM public IP address.
- Add the final application URL.
- Add Terraform output proof.
- Add Ansible role tree proof.
- Add all required notes and assignment question answers.
- Add your LinkedIn post URL.
- Do not expose SSH private keys, passwords, cloud credentials, database credentials, Terraform state files, subscription IDs, or account IDs.

---

# Completion Checklist

- [ ] Task 1: Project folder layout created
- [ ] Task 2: Terraform infrastructure provisioned
- [ ] Task 3: SSH key-based access verified
- [ ] Task 4: Ansible inventory and configuration created
- [ ] Task 5: Main Ansible playbook created
- [ ] Task 6: `common` role created
- [ ] Task 7: `nginx` role created
- [ ] Task 8: `epicbook` role created
- [ ] Task 9: Group variables created
- [ ] Task 10: Ansible playbook run completed
- [ ] Task 11: EpicBook deployment verified
- [ ] Terraform files created under only one cloud provider folder
- [ ] One Ubuntu VM was created
- [ ] One managed MySQL database was created
- [ ] SSH port `22` is restricted to the controller public IP
- [ ] HTTP port `80` is accessible
- [ ] MySQL port `3306` is not publicly open
- [ ] `ansible web -i inventory.ini -m ping` returns `SUCCESS`
- [ ] `site.yml` calls the roles in the correct order
- [ ] Database secrets are hidden or handled securely
- [ ] Nginx is active
- [ ] PM2 shows the EpicBook application running
- [ ] EpicBook responds on port `8080`
- [ ] Public URL loads in the browser
- [ ] Cart API verification works
- [ ] Playbook completes with `failed=0`
- [ ] Screenshots 1–27 are included
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