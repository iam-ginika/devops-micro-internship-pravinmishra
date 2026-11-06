# Assignment 03 — Deploy a Static Website to Multiple Servers Using a Multi-Play Ansible Playbook

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Student Details

**Full Name:** GINIKA NDUBUISI
**Cloud Platform Used:** AWS   
**Server 1 URL:** `http://<SERVER_1_PUBLIC_IP>`  
**Server 2 URL:** `http://<SERVER_2_PUBLIC_IP>`

---

## Purpose

In this assignment, you will create a multi-play Ansible playbook to install Nginx, deploy a static website to two Ubuntu servers, and verify that the website is accessible from both servers.

You may use either AWS EC2 instances or Azure Virtual Machines as your managed servers.

---

# Task 1 — Create the Project Structure

## Goal

Create the required folders and files for the Ansible project.

## Evidence

### Screenshot 1 — Terminal or VS Code showing the complete `static-web` project structure

[Output](screenshots/web-files.png)

---

# Task 2 — Configure the Ansible Inventory

## Goal

Add both Ubuntu servers to the Ansible inventory.

## Evidence

### Screenshot 2 — Output of `ansible-inventory -i inventory.ini --graph` showing `web1` and `web2`

[Output](screenshots/webs.png)

---

## Configuration File

Copy and paste the complete contents of your `inventory.ini` file below:

```ini
[web]
web1 ansible_host=16.171.47.224
web2 ansible_host=13.48.67.250

[web:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ginny/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3

```

---

# Task 3 — Verify Ansible Connectivity

## Goal

Confirm that the Ansible controller can connect to both servers.

## Evidence

### Screenshot 3 — Ansible ping output showing `SUCCESS` and `pong` for both servers

[Output](screenshots/success.png)

---

# Task 4 — Download and Personalize the Static Website

## Goal

Download `index.html` to the Ansible controller and personalize the website with your full name.

## Evidence

### Screenshot 4 — Edited `files/index.html` showing the footer line with your full name

[Output](screenshots/INDEX.png)

---

# Task 5 — Create the Multi-Play Ansible Playbook

## Goal

Create a single Ansible playbook containing separate plays for installation, deployment, and verification.

## Configuration File

Copy and paste the complete contents of your `site.yml` file below:

```yaml
yaml
---
- name: Install and configure Nginx
  hosts: web
  become: true
  tasks:
    - name: Update the APT package cache
      ansible.builtin.apt:
        update_cache: true

    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

- name: Deploy the static website
  hosts: web
  become: true
  tasks:
    - name: Copy index.html to the web root
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: "0644"
      notify: Reload nginx

  handlers:
    - name: Reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded

- name: Verify both websites from the controller
  hosts: localhost
  connection: local
  gather_facts: false
  become: false
  tasks:
    - name: Send an HTTP GET request to each web server
      ansible.builtin.uri:
        url: "http://{{ hostvars[item].ansible_host }}"
        status_code: 200
      loop: "{{ groups['web'] }}"
      register: website_checks

    - name: Confirm each server returned HTTP 200
      ansible.builtin.assert:
        that:
          - item.status == 200
        success_msg: "{{ item.item }} returned HTTP {{ item.status }}"
      loop: "{{ website_checks.results }}"

```

---

# Task 6 — Validate the Playbook Syntax

## Goal

Check the playbook for YAML or Ansible syntax errors before running it.

## Evidence

### Screenshot 5 — Successful syntax-check output showing `playbook: site.yml`

[Output](screenshots/check.png)

---

# Task 7 — Run the Multi-Play Playbook

## Goal

Install Nginx, deploy the website, and verify both servers in one playbook run.

## Evidence

### Screenshot 6 — Play 3 verification showing HTTP `200` for both servers

[Output](screenshots/Verification.png)

---

### Screenshot 7 — Final play recap showing `unreachable=0` and `failed=0` for `web1`, `web2`, and `localhost`

[Output](screenshots/recap.png)

---

# Task 8 — Verify Idempotency

## Goal

Run the playbook again and confirm that it does not make unnecessary changes.

## Evidence

### Screenshot 8 — Second playbook run showing the play recap with `changed=0`, `unreachable=0`, and `failed=0` for both web servers

[Output](screenshots/2nd-verification.png)

---

# Task 9 — Test Both Websites Manually

## Goal

Confirm that the static website is accessible from both public IP addresses.

## Evidence

### Screenshot 9 — `curl -I` output showing HTTP `200 OK` from both servers

[Output](screenshots/Head-request.png)

---

### Screenshot 10 — Browser showing the website from Server 1 with the public IP and your full name visible

[Output](screenshots/Browser1.png)

---

### Screenshot 11 — Browser showing the website from Server 2 with the public IP and your full name visible

[Output](screenshots/Browser2.png)

---

## Website URLs

Add both deployed website URLs below:

```text
Server 1: http://16.171.47.224
Server 2: http://13.48.67.250/
```

---

# Task 10 — Complete the Project README

## Goal

Document how the project works and record what you learned.

## README Content

Copy and paste the complete contents of your `README.md` file below:

```markdown
# Multi-Play Ansible Static Website Deployment

## Project Overview

I used a single multi-play Ansible playbook to deploy a static website to two Ubuntu web servers on AWS. The playbook installs and starts Nginx, copies my personalized `index.html` from the Ansible controller to both servers, reloads Nginx only when the website file changes, and checks from the controller that both websites return HTTP 200.

## Environment

- Cloud platform: Amazon Web Services (AWS), eu-north-1 (Stockholm)
- Operating system: Ubuntu 24.04 LTS
- Number of managed servers: 2 (web1 and web2)
- Web server: Nginx

## How to Run the Playbook

```bash
ansible-playbook -i inventory.ini site.yml
```

## Issue Faced and Solution

My mobile network kept switching between different public IP addresses. The security group only allows SSH from my controller's IP as a /32, so Ansible timed out whenever my IP changed. I fixed this by checking my current IP with `curl -4 ifconfig.me`, updating the allowed IP in my Terraform variables, and running `terraform apply`, which updated the SSH rule without opening SSH to everyone.

## What I Learned

I learned how to split one playbook into several plays, each with its own job, and how one inventory group lets the same tasks run on several servers at once. I also learned that a handler only runs when a task reports a change, and how to verify a deployment automatically from the controller with the `uri` and `assert` modules.

## Why Installation and Deployment Are Separate

Installing Nginx and deploying website content are different jobs that change at different rates. The Nginx setup rarely changes, while the website content may be updated often. Keeping them in separate plays makes the playbook easier to read and maintain, and lets me update the website without touching the server setup.

## Benefit of the Ansible Copy Module

The `copy` module sends one approved version of the file from the controller to every server, so both servers get exactly the same content. The servers don't need Git installed or access to the repository, and Ansible only copies the file when it has changed.

## Idempotency

Idempotency means running the same playbook again gives the same result without making unnecessary changes. On my second run, Nginx was already installed and running, and `index.html` already matched, so the copy task showed `ok` and the reload handler did not run. The play recap showed `changed=0` for both web servers.

## Website Verification Using the URI Module

After the playbook run, I checked both websites manually, outside Ansible. Running `curl -I` against each server's public IP returned `HTTP/1.1 200 OK`. I then opened `http://16.171.47.224` (web1) and `http://13.48.67.250` (web2) in a browser. Both loaded the same DevOps Micro Internship page with my name, Ginika Ndubuisi, in the footer, confirming that the personalized `index.html` was deployed correctly to both servers.
```

---

# LinkedIn Post Required

## Evidence

### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_this-week-i-wrote-a-multi-play-ansible-playbook-share-7524196872052453376-OZ-3/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

### Screenshot — Published LinkedIn post

[Output](screenshots/LinkedIn2.png)

---

# Assignment Questions

Answer the following in your own words:

**1. What issue did you face while completing this assignment, and how did you fix it?**

My mobile network kept giving me a new public IP address, and my security group only allowed SSH from one /32 address, so Ansible kept timing out with “UNREACHABLE”. I found my current IP with curl -4 ifconfig.me, updated the allowed IP in my Terraform variables, and ran terraform apply to update the SSH rule. I also got “Host key verification failed” on the first ping, which I fixed by connecting to each new server once with SSH and accepting its fingerprint.

---

**2. What did you learn from this assignment?**

I learned how to organise one playbook into several plays, each with a clear job, and how to run the same tasks on two servers at once through an inventory group. I also learned how handlers only run when something actually changes, and how to test a deployment automatically from the controller instead of checking each server by hand.

---

**3. Why is it useful to split installation, deployment, and verification into separate plays?**

Each play has one responsibility, which makes the playbook easier to read, troubleshoot and update. The Nginx installation rarely changes, while the website content changes more often, so keeping them separate means I can update the site without touching the server setup. Having verification as its own play also gives a clear final check that the whole deployment worked.

---

**4. What is one benefit of using the Ansible `copy` module instead of cloning the website directly from Git on every managed server?**

The copy module pushes the same approved file from the controller to every server, so both servers get identical content. The servers don’t need Git installed or access to the repository, and Ansible only transfers the file when it has actually changed.

---

**5. What does idempotency mean in this assignment?**

Idempotency means I can run the playbook again and it only makes changes when something is different from the desired state. On my second run, Nginx was already installed and running and index.html already matched, so the tasks showed ok, the reload handler didn’t run, and both web servers showed changed=0.

---

**6. What does the Ansible `uri` module verify in Play 3?**

The uri module runs on the controller and sends an HTTP request to each web server’s public IP to confirm the website responds. It expects status code 200, meaning the server returned the page successfully. The results are saved with register, and an assert task confirms each server returned 200.

---

# Required Files

Confirm that the following files are included in your assignment folder:

- [ ] `inventory.ini`
- [ ] `site.yml`
- [ ] `files/index.html`
- [ ] `README.md`

---

# Submission Instructions

- Add all required screenshots in the correct order.
- Full Name must be visible in required screenshots.
- Include both deployed website URLs.
- Paste `inventory.ini`, `site.yml`, and `README.md` as editable text.
- Answer all assignment questions clearly in your own words.
- Add your LinkedIn post URL.
- Do not expose SSH private keys, passwords, cloud account IDs, or other sensitive information.

---

# Completion Checklist

- [ ] Task 1: `static-web` folder structure is complete
- [ ] Task 2: Both servers are listed under the `[web]` group in `inventory.ini`
- [ ] Task 2: Inventory graph shows `web1` and `web2`
- [ ] Task 3: Ansible ping returns `SUCCESS` and `pong` for both servers
- [ ] Task 4: `files/index.html` contains your full name
- [ ] Task 5: `site.yml` contains three separate plays
- [ ] Task 5: Play 1 installs, starts, and enables Nginx
- [ ] Task 5: Play 2 deploys `index.html` using the `copy` module
- [ ] Task 5: Nginx reload handler is included
- [ ] Task 5: Play 3 verifies both web servers from the controller
- [ ] Task 6: Playbook syntax check passes
- [ ] Task 7: First playbook run completes with `unreachable=0` and `failed=0`
- [ ] Task 7: URI verification returns HTTP `200` for both servers
- [ ] Task 8: Second playbook run demonstrates idempotency
- [ ] Task 8: Second run shows `changed=0` for both web servers
- [ ] Task 9: Both `curl -I` commands return HTTP `200 OK`
- [ ] Task 9: Website loads from Server 1
- [ ] Task 9: Website loads from Server 2
- [ ] Task 9: Full name is visible on both deployed websites
- [ ] Task 10: `README.md` contains all required explanations
- [ ] Screenshots 1–11 are included
- [ ] `inventory.ini`, `site.yml`, and `README.md` are pasted as editable text
- [ ] Both website URLs are included
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