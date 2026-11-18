# Assignment 04 — Deploy Mini Finance on Azure Using Terraform and Ansible

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will provision Azure infrastructure using Terraform and deploy the Mini Finance website using an Ansible multi-play playbook.

Terraform will create the Azure Virtual Machine and networking resources. Ansible will install Nginx, clone the Mini Finance repository, deploy the website, and verify the deployment.

---

# Task 1 — Create the Project Structure

## Goal

Create separate directories and files for the Terraform infrastructure and Ansible configuration.

### Evidence

#### Screenshot 1 — Terminal or VS Code showing the complete `mini-finance` project structure

[Output](screenshots/Project-structure.png)

---

### Notes

I set up the Mini-Finance project in my ansible-onboarding workspace, with separate terraform/ and ansible/ folders. Terraform will provision the Azure infrastructure and Ansible will configure the virtual machine. I created the Terraform files (providers.tf, main.tf, variables.tf, outputs.tf) and the Ansible files (inventory.ini, site.yml). I also added a README.md to document the project and a .gitignore that keeps state files, the .terraform/ folder, plan files, crash logs and private keys out of version control. Finally, I checked the folder structure to confirm the workspace was ready for infrastructure automation.

---

# Task 2 — Create the Azure Infrastructure Using Terraform

## Goal

Use Terraform to provision an Ubuntu Virtual Machine with the required Azure networking and security resources.

### Evidence

#### Screenshot 2 — Terraform code showing the `Allow-SSH` rule for port `22` and the `Allow-HTTP` rule for port `80`

[Output](screenshots/Rules.png)

---

#### Screenshot 3 — Terraform code showing the association between `nsg-mini-finance` and `nic-mini-finance`

[Output](screenshots/Association.png)

---

### Notes

I wrote the Terraform code to provision the Mini-Finance infrastructure on Azure in the Sweden Central region. providers.tf configures the azurerm provider. variables.tf and terraform.tfvars hold the values I can change, such as the region, VM size, SSH key path and my public IP. main.tf creates a resource group, a virtual network and subnet, a static public IP, a network interface (nic-mini-finance) and an Ubuntu 24.04 VM that only accepts SSH key logins. The network security group (nsg-mini-finance) has two inbound rules: Allow-SSH opens port 22 to my IP only, and Allow-HTTP opens port 80 to the public for the website. The NSG is attached directly to the VM’s network interface.

---

# Task 3 — Initialize and Apply the Terraform Configuration

## Goal

Format and validate the Terraform configuration, review the execution plan, and provision the Azure infrastructure.

### Evidence

#### Screenshot 4 — End of the `terraform apply` output showing `Apply complete!` with no errors

[Output](screenshots/Apply-complete.png)

---

#### Screenshot 5 — Output of `terraform output public_ip` showing the VM’s public IP address

[Output](screenshots/Vm-IP-Output.png)

---

### Notes

I initialized and validated the Terraform configuration, then provisioned the Azure infrastructure. terraform fmt tidied the code layout, terraform init downloaded the AzureRM provider, and terraform validate confirmed the configuration was valid. I reviewed the execution plan before applying. It showed 8 resources to create and none to change or destroy: the resource group, virtual network, subnet, network security group, public IP, network interface, NSG-to-NIC association and the Ubuntu 22.04 VM. I then ran terraform apply, confirmed with yes, and the deployment finished with “Apply complete!”. Finally, I ran terraform output public_ip to get the VM’s public IP address, which I’ll use for the SSH test, the Ansible inventory and browser testing.

---

# Task 4 — Verify Passwordless SSH Access

## Goal

Confirm that the Ansible controller can connect to the Terraform-provisioned Azure VM using SSH key authentication.

### Evidence

#### Screenshot 6 — Passwordless SSH command and the returned `mini-finance` hostname

[Output](screenshots/Passwordless-SSH.png)

---

### Notes

For Task 4, I tested passwordless SSH from my Ansible controller to the Azure VM. I got the VM’s public IP from terraform output public_ip and connected as azureuser with my ed25519 private key, running hostname on the VM. On the first connection I accepted the host fingerprint. The VM returned mini-finance, the computer name set in Terraform, and SSH never asked for a password. That confirms the public key Terraform added to the VM works and that my IP is allowed through the Allow-SSH rule, so Ansible will be able to connect the same way.

---

# Task 5 — Create the Ansible Inventory and Verify Connectivity

## Goal

Add the Terraform-provisioned Azure VM to the Ansible inventory and confirm that Ansible can connect to it.

### Evidence

#### Screenshot 7 — Ansible ping output showing `SUCCESS` and `pong` from the Azure VM

[Output](screenshots/Ping-success.png)

---

### Configuration File

Copy and paste the complete contents of your `ansible/inventory.ini` file below:

```ini
[web]
20.240.160.172

[web:vars]
ansible_user=azureuser
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# Task 6 — Create the Multi-Play Ansible Playbook

## Goal

Create one Ansible playbook containing separate plays to install Nginx, deploy the Mini Finance website, and verify the deployment.

### Evidence

#### Screenshot 8 — `site.yml` showing Play 1 and the beginning of Play 2

Screenshot must show:

- Play 1 targeting the `web` group
- Installation of `nginx`, `git`, and `rsync`
- Nginx service configured as started and enabled
- Beginning of Play 2 with the Git repository URL and synchronization task

[Output](screenshots/Play1&2.png)

---

#### Screenshot 9 — `site.yml` showing the deployment destination, handler, and Play 3 verification

Screenshot must show:

- Website destination `/var/www/html/`
- Ownership set to `www-data:www-data`
- Nginx reload handler
- Play 3 targeting `localhost`
- The `uri` verification and `assert` condition

[Output](screenshots/Destination.png)

---

### Configuration File

Copy and paste the complete contents of your `ansible/site.yml` file below:

```yaml
---
- name: Install and configure Nginx
  hosts: web
  become: true

  tasks:
    - name: Update apt package cache
      ansible.builtin.apt:
        update_cache: true

    - name: Install required packages
      ansible.builtin.apt:
        name:
          - nginx
          - git
          - rsync
        state: present

    - name: Ensure Nginx is started and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true


- name: Clone and deploy Mini Finance website
  hosts: web
  become: true

  tasks:
    - name: Clone or update Mini Finance repository
      ansible.builtin.git:
        repo: "https://github.com/pravinmishraaws/mini-finance-project"
        dest: /opt/mini-finance
        version: main
        update: true

    - name: Synchronize website files to /var/www/html/
      ansible.posix.synchronize:
        src: /opt/mini-finance/
        dest: /var/www/html/
        rsync_opts:
          - "--exclude=.git"
      delegate_to: "{{ inventory_hostname }}"
      notify: Reload Nginx

    - name: Set ownership of website files
      ansible.builtin.file:
        path: /var/www/html/
        owner: www-data
        group: www-data
        recurse: true

  handlers:
    - name: Reload Nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded


- name: Verify Mini Finance deployment from controller
  hosts: localhost
  connection: local
  become: false
  gather_facts: false

  tasks:
    - name: Check website returns HTTP 200
      ansible.builtin.uri:
        url: "http://{{ groups['web'][0] }}"
        status_code: 200
      register: website_result

    - name: Assert website is reachable
      ansible.builtin.assert:
        that:
          - website_result.status == 200
        success_msg: "Mini Finance website is live and returned HTTP 200"
        fail_msg: "Mini Finance website did not return HTTP 200"

```

---

# Task 7 — Validate and Run the Ansible Playbook

## Goal

Validate the syntax of the multi-play Ansible playbook and run it to install Nginx, deploy the Mini Finance website, and verify the deployment.

### Evidence

#### Screenshot 10 — Successful playbook syntax check showing `playbook: site.yml`

[Output](screenshots/Playbok-syntax.png)

---

#### Screenshot 11 — Play 3 output showing the successful HTTP verification and assertion

[Output](screenshots/Playbook3.png)

---

#### Screenshot 12 — Final `PLAY RECAP` showing `failed=0` and `unreachable=0`

[Output](screenshots/Playbook-Recap.png)

---

### Notes

I validated and ran the multi-play Ansible playbook from the ansible directory. First I ran ansible-playbook -i inventory.ini site.yml --syntax-check, which returned playbook: site.yml, confirming the YAML and module names were correct. I then ran the full playbook. Play 1 updated the apt cache, installed Nginx, Git and rsync, and made sure Nginx was started and enabled. Play 2 cloned the Mini Finance repository into /opt/mini-finance, synced the website files into /var/www/html/ without the .git folder, set ownership to www-data:www-data, and triggered the handler to reload Nginx because the files had changed. Play 3 ran on my controller, sent an HTTP request to the VM’s public IP, and the assertion passed with status code 200. The final PLAY RECAP showed failed=0 and unreachable=0, which confirmed the website was deployed and serving correctly.

---

# Task 8 — Test the Mini Finance Website in a Browser

## Goal

Confirm that the Mini Finance website is publicly accessible through the Azure VM’s public IP address.

### Evidence

#### Screenshot 13 — Mini Finance website successfully loading in the browser, with the Azure VM’s public IP address visible in the address bar

[Output](screenshots/Mini-finance%20App.png)

---

### Website URL

Add your deployed website URL below:

```text
http://20.240.160.172
```

---

# Task 9 — Create the Project README

## Goal

Create a `README.md` file to document the Mini Finance infrastructure and deployment project.

### Evidence

#### Screenshot 14 — Completed `README.md` displayed in the VS Code Markdown preview or terminal

[Output](screenshots/Ass4-ReadME.png)

---

### README Content

Copy and paste the complete contents of your `README.md` file below:

```markdown
# Mini Finance on Azure: Terraform Builds It, Ansible Ships It

## Project Objective

The goal of this project was to take the Mini Finance static website from a GitHub repository to a live public URL on Azure, with no manual clicking in the portal and no hand-editing on the server.

I split the work between two tools. Terraform creates the Azure network, firewall and Ubuntu VM, then hands me the VM's public IP. Ansible picks up that IP, turns the bare VM into an Nginx web server, deploys the site, and finally checks from my own machine that the site answers with HTTP 200. The project shows how provisioning and configuration fit together while staying separate.

## Tools and Technologies

- **Terraform** (AzureRM provider 4.x): describes every Azure resource as code
- **Microsoft Azure**: hosts the infrastructure in the Sweden Central region
- **Ansible**: runs from my WSL Ubuntu controller inside a Python virtual environment
- **Nginx**: serves the website on port 80
- **Git**: pulls the Mini Finance source onto the VM
- **rsync** (via `ansible.posix.synchronize`): copies the site into the web root and skips the `.git` folder

## Infrastructure Created

All resources live in one resource group, so the whole project can be removed with a single `terraform destroy`.

- **Resource Group** `rg-mini-finance`: holds everything for this project
- **Virtual Network** `vnet-mini-finance` (`10.0.0.0/16`): private address space for the project
- **Subnet** `subnet-mini-finance` (`10.0.1.0/24`): where the VM's network card sits
- **Network Security Group** `nsg-mini-finance`: two inbound rules:
  - `Allow-SSH`: port 22, allowed only from my public IP (`/32`)
  - `Allow-HTTP`: port 80, allowed from `0.0.0.0/0`
- **Public IP** `pip-mini-finance`: Standard SKU, static, so the address doesn't change between reboots
- **Network Interface** `nic-mini-finance`: IP configuration `ipconfig-mini-finance`, connected to the subnet and public IP, with the NSG attached directly to it
- **Ubuntu 22.04 VM** `vm-mini-finance`: size `Standard_D2s_v5`, computer name `mini-finance`, user `azureuser`, SSH key login only (passwords disabled)

My Terraform files are split by purpose: `providers.tf` for the AzureRM setup, `variables.tf` for inputs such as region, VM size, key path and my IP, `main.tf` for the resources, and `outputs.tf` for `public_ip`, `ssh_command` and `website_url`. Values specific to me live in `terraform.tfvars`, which is excluded from Git.

## Ansible Deployment Workflow

`inventory.ini` puts the VM's public IP in a `[web]` group and tells Ansible to log in as `azureuser` with my ed25519 private key. `site.yml` then runs three plays in order:

**Play 1: Install and configure Nginx** (`hosts: web`, `become: true`)
Refreshes the apt cache, installs `nginx`, `git` and `rsync`, and makes sure Nginx is started now and enabled on boot.

**Play 2: Clone and deploy the website** (`hosts: web`, `become: true`)
Clones the Mini Finance repository into `/opt/mini-finance`, then syncs it into `/var/www/html/` without the `.git` folder. Both folders are on the VM, so the sync task is delegated to the VM itself with `delegate_to: "{{ inventory_hostname }}"`. The web root is then owned by `www-data:www-data`. A handler reloads Nginx gracefully, but only when the sync actually changed files.

**Play 3: Verify from the controller** (`hosts: localhost`, `connection: local`, no `become`)
Uses `ansible.builtin.uri` to request `http://<VM public IP>`, registers the response, and uses `ansible.builtin.assert` to confirm `status == 200`.

## Verification

I checked the deployment at each hand-off point rather than only at the end:

1. `terraform plan` showed 8 resources to add and nothing to change or destroy. `terraform apply` finished with `Apply complete!`.
2. `ssh -i ~/.ssh/id_ed25519 azureuser@<PUBLIC_IP> "hostname"` returned `mini-finance` without asking for a password.
3. `ansible web -i inventory.ini -m ping` returned `SUCCESS` and `pong`.
4. `ansible-playbook --syntax-check` returned `playbook: site.yml`. The full run ended with `failed=0` and `unreachable=0`, and the Play 3 assertion passed with HTTP 200.
5. Opening `http://<PUBLIC_IP>` in my browser loaded the Mini Finance site with its full styling.

## Challenge and Solution

**Challenge:** On the first playbook run, Play 1 finished normally but Play 2 froze on the repository clone task. There was no error and no progress.

**Cause:** I tested the clone directly on the VM with `git ls-remote`, and GitHub replied `could not read Username`. GitHub answers that way when a repository doesn't exist at the given address or isn't public. Git was waiting for a login that Ansible could never type in, so the task hung instead of failing.

**Solution:** I stopped the run with `Ctrl + C`, confirmed the correct public repository address, updated the repo URL in `site.yml`, and ran the playbook again. The clone completed in seconds, and all three plays finished successfully.

Lesson: when an Ansible task hangs rather than fails, it's usually waiting for input. Running the same command by hand on the server shows the hidden prompt straight away.

## What I Learned

- **Clear ownership makes troubleshooting faster.** When something broke, I knew which tool to look at: network or VM problems meant Terraform, and anything inside the server meant Ansible.
- **Outputs are the bridge.** Terraform's `public_ip` output fed my SSH test, my Ansible inventory and my browser check.
- **Variables keep code reusable.** Region, VM size, key path and my IP all live in variables, so I can change region or size by editing one file instead of the resources.
- **Verification belongs in the playbook.** The `uri` and `assert` play means a successful run proves the site works, not just that the tasks ran.
- **Reruns should be boring.** Running Terraform and the playbook a second time showed few or no changes, which is what idempotent automation should look like.
- **Least access by default.** Port 22 is limited to my IP, and the VM accepts only my SSH key.

```

---

# LinkedIn Post Required

## Evidence

#### Screenshot 15 — Published LinkedIn post showing the text and at least one deployment screenshot

[Output](screenshots/LinkedIn-post.png)

---

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_the-mini-finance-website-was-deployed-to-share-7528732805426540544-0aIc/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

### LinkedIn Submission Notes

**One challenge you faced and how you fixed it:**

My playbook froze on the task that clones the Mini Finance repository. There was no error; it just never finished. I ran git ls-remote with the same URL on the VM and got could not read Username, which meant GitHub couldn’t find a public repository at that address and was asking for a login that Ansible could never provide. I stopped the run, confirmed the correct public repository URL, updated it in site.yml, and ran the playbook again. This time the clone took seconds and all three plays passed.

---

**One real-world example where you can use this learning:**

A small clinic or agency that needs a marketing site or a staff portal could get the same setup. Terraform builds a consistent Azure environment for them, and Ansible deploys and updates the site. When the client wants a test copy before a launch, I can run the same code with a different variables file and get an identical environment in minutes. When the test is done, terraform destroy removes it so they stop paying for it.

---

# Assignment Questions

Answer the following in your own words:

**1. What did you provision using Terraform in this assignment?**

I used Terraform to build the full Azure environment in the Sweden Central region:

- a resource group (rg-mini-finance)
- a virtual network (vnet-mini-finance, 10.0.0.0/16)
- a subnet (subnet-mini-finance, 10.0.1.0/24)
- a network security group (nsg-mini-finance) with Allow-SSH and Allow-HTTP rules
- a Standard static public IP (pip-mini-finance)
- a network interface (nic-mini-finance), with the NSG attached to it
- an Ubuntu 22.04 VM (vm-mini-finance) that only accepts SSH key logins

Terraform also outputs the VM’s public IP for the next steps.

---

**2. What did Ansible configure and deploy in this assignment?**

Ansible installed Nginx, Git and rsync on the VM and made sure Nginx was running and enabled on boot. It then cloned the Mini Finance repository into /opt/mini-finance, synced the site files into /var/www/html/ without the .git folder, set the owner to www-data, and reloaded Nginx when the files changed. Finally, it checked from my controller that the website returned HTTP 200.

---

**3. Why is SSH access on port `22` restricted to your public IP address?**

SSH gives full control of the server, so it should only be reachable by the person managing it. Limiting port 22 to my IP with /32 means bots scanning the internet can’t even attempt to log in. Combined with key-only login, that makes it very hard for anyone else to get in.

---

**4. Why is HTTP port `80` open to the internet?**

The Mini Finance site is meant to be public, so anyone with the IP needs to be able to load it in a browser. Port 80 only reaches Nginx serving static pages, not the server’s controls, so opening it to everyone is low risk.

---

**5. What is the purpose of the Ansible inventory file?**

The inventory tells Ansible which servers to manage and how to connect to them. Mine puts the VM’s public IP in a [web] group and sets the login user (azureuser) and the private key to use. The playbook targets hosts: web, so adding more servers to that group later would deploy to all of them without changing the playbook.

---

**6. Why does the playbook use separate plays for install, deploy, and verify?**

Each play has one job, which makes the playbook easier to read and troubleshoot. If something fails, the play name tells me straight away whether it was the server setup, the deployment or the check. The plays also need different settings: install and deploy run on the VM with become: true, while verify runs on my own machine with no sudo, testing the site the way a visitor would.

---

**7. Why is `rsync` useful when deploying website files?**

rsync only copies files that have changed, so redeployments are fast and the task reports changed only when content is actually different. That’s what lets the Nginx reload handler run only when it’s needed. It can also skip folders I don’t want served, such as .git, so repository history isn’t exposed on the web server.

---

**8. What does the Ansible `uri` module verify in this assignment?**

The uri module sends an HTTP request from my controller to the VM’s public IP and checks that the response is status code 200. That proves the whole chain works from outside: the firewall allows port 80, Nginx is running, and the site files are in place. The assert task then confirms status == 200, so the playbook fails clearly if the site isn’t up.

---

**9. What issue did you face during this assignment, and how did you fix it?**

The repository clone task in Play 2 hung with no error. Testing the same URL on the VM with git ls-remote showed that GitHub was asking for a username, which happens when the repository isn’t publicly available at that address. I stopped the playbook, found the correct public repository URL, updated site.yml, and reran it. The clone finished quickly and the playbook completed with failed=0 and unreachable=0.

---

**10. What did you learn from using Terraform and Ansible together?**

Terraform and Ansible each own one layer. Terraform builds and wires up the cloud resources, and Ansible handles everything inside the server. Terraform’s public_ip output is the link between the two, because it becomes the host in the Ansible inventory. I also learned to build checks into the automation instead of assuming success, and that a well-written setup can be run again safely, rebuilt from scratch, or destroyed completely with a few commands.

---

# Required Files

Confirm that the following files are included in your assignment folder:

- [ ] `.gitignore`
- [ ] `README.md`
- [ ] `terraform/providers.tf`
- [ ] `terraform/main.tf`
- [ ] `terraform/variables.tf`
- [ ] `terraform/outputs.tf`
- [ ] `ansible/inventory.ini`
- [ ] `ansible/site.yml`

---

# Submission Instructions

- Add all required screenshots in the correct order.
- Full Name must be visible in required screenshots.
- Add the Azure VM public IP address.
- Add the final Mini Finance website URL.
- Paste `inventory.ini`, `site.yml`, and `README.md` as editable text.
- Answer all assignment questions clearly in your own words.
- Add your LinkedIn post URL.
- Do not expose SSH private keys, passwords, Azure credentials, subscription IDs, Terraform state contents, or other sensitive information.

---

# Completion Checklist

- [ ] Task 1: `mini-finance` project structure created
- [ ] Task 1: `.gitignore` created
- [ ] Task 2: Terraform Azure infrastructure code created
- [ ] Task 2: `Allow-SSH` rule configured for port `22`
- [ ] Task 2: `Allow-HTTP` rule configured for port `80`
- [ ] Task 2: NSG associated with the Network Interface
- [ ] Task 3: `terraform fmt` completed
- [ ] Task 3: `terraform init` completed
- [ ] Task 3: `terraform validate` completed successfully
- [ ] Task 3: `terraform apply` completed successfully
- [ ] Task 3: `terraform output public_ip` displayed the VM public IP
- [ ] Task 4: Passwordless SSH works from the Ansible controller
- [ ] Task 5: `inventory.ini` created
- [ ] Task 5: Ansible ping returns `SUCCESS` and `pong`
- [ ] Task 6: `site.yml` contains three separate plays
- [ ] Task 6: Play 1 installs Nginx, Git, and rsync
- [ ] Task 6: Play 2 clones and deploys the Mini Finance website
- [ ] Task 6: Play 3 verifies HTTP status code `200`
- [ ] Task 7: Playbook syntax check passes
- [ ] Task 7: Ansible playbook completes successfully
- [ ] Task 7: Final recap shows `failed=0` and `unreachable=0`
- [ ] Task 8: Mini Finance website loads in the browser
- [ ] Task 8: Azure VM public IP is visible in the browser screenshot
- [ ] Task 9: `README.md` completed
- [ ] Screenshots 1–15 are included
- [ ] `inventory.ini`, `site.yml`, and `README.md` are pasted as editable text
- [ ] Assignment questions are answered
- [ ] LinkedIn post published with Anyone visibility
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