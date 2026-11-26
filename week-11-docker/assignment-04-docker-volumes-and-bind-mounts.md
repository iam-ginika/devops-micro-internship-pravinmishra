# Assignment 4 — Docker Volumes and Bind Mounts

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will use Docker Bind Mounts and Docker Volumes to persist logs and application data outside a container’s lifecycle. You will verify that data remains available after containers are removed and recreated.

---

# Task 1 — Persist Nginx Logs Using a Bind Mount

## Goal

Deploy an Nginx container with a Bind Mount and verify that its log files remain on the VM host after the container is removed.

### Evidence

#### Screenshot 1 — Nginx Image Pull

Add a screenshot of the terminal showing successful completion of:

```bash
docker pull nginx:alpine
```

![Output](screenshots/Pull2.png)

---

#### Screenshot 2 — Host Log Directory

Add a screenshot of the terminal showing the created host directory:

```text
$HOME/nginx-logs
```

![Output](screenshots/Directory.png)

---

#### Screenshot 3 — Running Nginx Container with Port Mapping

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show the `myweb` container with:

```text
0.0.0.0:80->80/tcp
```

![Output](screenshots/Myweb2.png)

---

#### Screenshot 4 — Nginx Welcome Page

Add a browser screenshot showing the Nginx Welcome Page at:

```text
http://<YOUR-VM-PUBLIC-IP>
```

Ensure that the VM public IP is visible in the address bar. Add your full name as a clear caption directly below the screenshot.

![Output](screenshots/Welcome-page2.png)

---

#### Screenshot 5 — Bind-Mounted Log Files

Add a screenshot of the terminal showing the host log files and access-log content from:

```text
$HOME/nginx-logs
```

The output must show `access.log`, `error.log`, and an access-log entry created when you opened the Nginx page.

![Output](screenshots/logs.png)

---

#### Screenshot 6 — Nginx Container Removed

Add a screenshot of the terminal showing successful completion of:

```bash
docker stop myweb
docker rm myweb
```

![Output](screenshots/Stop-rm-success.png)

---

#### Screenshot 7 — Logs Persist After Container Removal

Add a screenshot of the terminal showing that `access.log` and `error.log` still exist in:

```text
$HOME/nginx-logs
```

The access log must retain its content after the container has been removed.

![Output](screenshots/Logs2.png)

---

# Task 2 — Share Persistent Data Using a Docker Volume

## Goal

Deploy backend and frontend containers that share data through a named Docker Volume. Verify that the data remains after both containers are removed and recreated.

### Evidence

#### Screenshot 8 — Project File Structure

Add a screenshot of the terminal showing the `two-tier-app` project structure, including separate `backend` and `frontend` directories with a `Dockerfile` and `index.js` file in each.

![Output](screenshots/File-structure.png)

---

#### Screenshot 9 — Custom Docker Network

Add a screenshot of the terminal showing `mynetwork` in:

```bash
docker network ls
```

![Output](screenshots/Network2.png)

---

#### Screenshot 10 — Docker Volume

Add a screenshot of the terminal showing `shared-data` in:

```bash
docker volume ls
```

![Output](screenshots/Volume.png)

---

#### Screenshot 11 — Backend Dockerfile

Add a screenshot of the terminal showing the completed backend `Dockerfile`.

![Output](screenshots/Backend-DF.png)

---

#### Screenshot 12 — Backend Image Build

Add a screenshot of the terminal showing successful completion of the `backend-app:latest` image build.

![Output](screenshots/Build1.png)
![Output](screenshots/Build2.png)

---

#### Screenshot 13 — Running Backend Container

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show the running `backend` container.

![Output](screenshots/Container2.png)

---

#### Screenshot 14 — Frontend Dockerfile

Add a screenshot of the terminal showing the completed frontend `Dockerfile`.

![Output](screenshots/Frontend-DF.png)

---

#### Screenshot 15 — Frontend Image Build

Add a screenshot of the terminal showing successful completion of the `frontend-app:latest` image build.

![Output](screenshots/Frontend-build.png)

---

#### Screenshot 16 — Running Backend and Frontend Containers

Add a screenshot of the terminal showing:

```bash
docker ps
```

The output must show both `backend` and `frontend` containers running. Only `frontend` must have the published port mapping:

```text
0.0.0.0:80->80/tcp
```

![Output](screenshots/Both-containers.png)

---

#### Screenshot 17 — Backend Write Operation

Add a screenshot of the terminal showing a successful backend write operation to the shared Docker Volume.

The output must include:

```text
Data written: Hello from Backend!
```

![Output](screenshots/Backend-hello.png)

---

#### Screenshot 18 — Frontend Reads Shared Data

Add a browser screenshot showing:

```text
Hello from Backend!
```

Add your full name as a clear caption directly below the screenshot.

![Output](screenshots/Browser-hello.png)

---

#### Screenshot 19 — First Shared-Data Update

Add a browser screenshot showing:

```text
Test Data 1
```

Add your full name as a clear caption directly below the screenshot.

![Output](screenshots/Test1.png)

---

#### Screenshot 20 — Second Shared-Data Update

Add a browser screenshot showing:

```text
Test Data 2 - New Update
```

Add your full name as a clear caption directly below the screenshot.

![Output](screenshots/Test2.png)

---

#### Screenshot 21 — Container Removal and Recreation

Add a screenshot of the terminal showing the `frontend` and `backend` containers removed and recreated using the same `shared-data` Docker Volume.

![Output](screenshots/Shared-data.png)

---

#### Screenshot 22 — Data Persists After Recreation

Add a browser screenshot showing:

```text
Test Data 2 - New Update
```

This proves that the `shared-data` Docker Volume outlived both application containers.

Add your full name as a clear caption directly below the screenshot.

![Output](screenshots/Browser-sd.png)

---

# Storage Persistence Notes

Write a short explanation covering:

- The difference between a Bind Mount and a Docker Volume
- How Task 1 proved Bind Mount persistence
- How Task 2 proved Docker Volume persistence
- Why Docker Volumes are commonly used for application data

Difference between a Bind Mount and a Docker Volume

A Bind Mount connects a specific directory on the host to a directory inside a container, and I choose the host path. In Task 1, I mounted $HOME/nginx-logs to /var/log/nginx. A Docker Volume is created and managed by Docker, so I refer to it by name instead of a host path. In Task 2, I created shared-data and mounted it at /data. In both cases, the data is stored outside the container’s writable layer.

How Task 1 proved Bind Mount persistence

I ran an Nginx container with the Bind Mount and opened the site through the VM’s public IP, which added a request to the access log. access.log and error.log appeared in $HOME/nginx-logs on the host. I then stopped and removed the myweb container and checked the directory again. Both files were still there and the access log kept its entries, which proved the logs did not depend on the container.

How Task 2 proved Docker Volume persistence

I ran separate backend and frontend containers with the same shared-data volume mounted at /data. The backend wrote to /data/message.txt, and the frontend displayed the same content in the browser. When I updated the file from the backend, the new text appeared after a browser refresh. I then removed both containers and recreated them from the same images with the same volume. The browser still showed “Test Data 2 - New Update”, which proved the data outlived both containers.

Why Docker Volumes are commonly used for application data

Docker Volumes are managed by Docker, so applications do not depend on a specific host directory structure. A container can be replaced or upgraded while its data stays in the volume, and more than one container can use the same volume. This makes volumes a good fit for databases, uploaded files, reports and other application data.
---

# Public Application URL

**Application URL:** http://16.170.218.128/

---

# LinkedIn Requirement

## Goal

Create a LinkedIn post about Docker Volumes and Bind Mounts, including one difference between them, how you verified persistent storage, and your key learning outcomes.

### Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/ginikandubuisi_devops-docker-dockervolumes-share-7530962507406950400-xMch/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

#### LinkedIn Post Screenshot

Add a screenshot of the published LinkedIn post here. Include a screenshot of the application displaying shared data.

![Output](screenshots/LinkedIn-4.png)
---

# Submission Instructions

- Complete all tasks in sequence.
- Include Screenshots 1–22 exactly as specified.
- Include the Storage Persistence Notes section.
- Include the public application URL.
- Include the LinkedIn post URL and screenshot.
- Ensure that your full name is visible in all terminal screenshots.
- Add your full name as a clear caption below every browser screenshot.
- Do not expose private keys, passwords, access keys, tokens, account IDs, or other sensitive information.

---

# Completion Checklist

- [ ] Nginx image pulled successfully
- [ ] Host log directory created
- [ ] Bind Mount configured successfully
- [ ] Nginx logs remain after container removal
- [ ] Custom Docker network created
- [ ] Docker Volume created
- [ ] Backend Dockerfile and image created
- [ ] Frontend Dockerfile and image created
- [ ] Both containers mount `shared-data`
- [ ] Backend writes data to the Docker Volume
- [ ] Frontend reads the same data from the Docker Volume
- [ ] Updated data appears after browser refresh
- [ ] Data remains after frontend and backend containers are removed and recreated
- [ ] All required screenshots included
- [ ] Storage Persistence Notes completed
- [ ] Public application URL included
- [ ] LinkedIn post URL and screenshot included
- [ ] Full name visible in terminal screenshots
- [ ] Browser screenshots have full-name captions
- [ ] No sensitive information exposed

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
