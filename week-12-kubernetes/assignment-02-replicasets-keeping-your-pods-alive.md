# Assignment 02 — ReplicaSets: Keeping Your Pods Alive

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will create three NGINX Pods using a ReplicaSet, test auto-healing by deleting one Pod, and scale the ReplicaSet from three to five Pods.

---

# Task 0 — Pre-check: Remove the Pod from Assignment 01

## Goal

Confirm that you are connected to your learning cluster and remove the previous standalone Pod to prevent the ReplicaSet from adopting it.

### Evidence

No screenshots required.

---

# Task 1 — Set Up Your Working Directory

## Goal

Create and enter a dedicated directory for this ReplicaSet lab.

### Evidence

#### Screenshot 01 — Output of `pwd` showing the working directory ending in `/k8s-labs/replicasets`

![Output](screenshots/Workking-directory.png)

---

### Notes

**1. Why is it useful to keep Kubernetes manifests in organized directories?**

It keeps files organized, easy to find and version-controlled, and lets you apply related resources together. 

---

**2. What file will you create in this directory for this assignment?**

`nginx-replicaset.yaml`, a manifest that defines a ReplicaSet running three NGINX Pods.

---

# Task 2 — Create and Apply the NGINX ReplicaSet

## Goal

Create a ReplicaSet that maintains three NGINX Pods.

### Evidence

#### Screenshot 02 — Completed `nginx-replicaset.yaml` manifest showing `replicas: 3`

![Output](screenshots/nginx-replicaset.png)

---

#### Screenshot 03 — Output of `kubectl apply -f nginx-replicaset.yaml`

![Output](screenshots/nginx-replicaset-created.png)

---

#### Screenshot 04 — Output of `kubectl get pods` showing three NGINX Pods with `1/1` under `READY` and `Running` under `STATUS`

![Output](screenshots/get-pod-output.png)

---

### Notes

**1. What does `replicas: 3` mean in this manifest?**

It tells Kubernetes to keep exactly three NGINX Pods running at all times.

---

**2. Why must `selector.matchLabels` match `template.metadata.labels`?**

`template.metadata.labels` sets the labels on the Pods the ReplicaSet creates, and `selector.matchLabels` tells the ReplicaSet which Pods to manage. They must match so the ReplicaSet can find and control its own Pods.


---

**3. What image and image tag are used for the NGINX container?**

The image is `nginx` and the tag is `1.21.1` (`nginx:1.21.1`).

---

# Task 3 — Test ReplicaSet Auto-Healing

## Goal

Delete one Pod managed by the ReplicaSet and observe Kubernetes automatically create a replacement.

### Evidence

#### Screenshot 05 — Output of the command deleting one ReplicaSet-managed Pod

![Output](screenshots/Deletion.png)

---

#### Screenshot 06 — Output of `kubectl get pods` showing the replacement Pod with a different name from the deleted Pod

![Output](screenshots/Replacement.png)

---

#### Screenshot 07 — Output showing three NGINX Pods with `1/1` under `READY` and `Running` under `STATUS` again

![Output](screenshots/Running-pods.png)

---

### Notes

**1. What happened after you deleted one Pod?**

The ReplicaSet automatically created a new Pod (`nginx-replicaset-ckmcc`) to replace the deleted one (`nginx-replicaset-fggtj`).

---

**2. How does the ReplicaSet know that a replacement Pod is needed?**

It keeps comparing the number of running Pods with matching labels against `replicas: 3`. When the count dropped to two, it created a new Pod.

---

**3. What proves that auto-healing worked successfully?**

`kubectl get pods` shows three Pods running again, including a new Pod with a different name and a younger age than the others.

---

# Task 4 — Manually Scale the ReplicaSet

## Goal

Scale the ReplicaSet from three NGINX Pods to five NGINX Pods by changing the YAML manifest.

### Evidence

#### Screenshot 08 — Updated `nginx-replicaset.yaml` showing `replicas: 5`

![Output](screenshots/5-replicas.png)

---

#### Screenshot 09 — Output of `kubectl get pods` showing five NGINX Pods with `1/1` under `READY` and `Running` under `STATUS`

![Output](screenshots/5-pods.png)

---

### Notes

**1. What change did you make to scale the ReplicaSet?**

I changed `replicas: 3` to `replicas: 5` in `nginx-replicaset.yaml` and re-applied it with `kubectl apply -f nginx-replicaset.yaml`.

---

**2. Did you manually create the additional Pods? Explain why or why not.**

No. The ReplicaSet saw that the desired count was five and created the two extra Pods automatically.

---

**3. What would happen if you changed the replica count from five back to three?**

The ReplicaSet would delete two Pods automatically, leaving three running.

---

# LinkedIn Requirement

Create a LinkedIn post that includes:

- A short explanation of what a Kubernetes ReplicaSet does.
- What happened when you deleted one NGINX Pod.
- How Kubernetes automatically created a replacement Pod to maintain the desired replica count.
- What you learned by changing the replica count from three to five.
- One screenshot showing either:
  - Three Pods running after the auto-healing test, or
  - Five Pods running after scaling the ReplicaSet.

Do not share sensitive information, cluster credentials, tokens, or kubeconfig details in your post.

### Evidence

#### Screenshot 10 — Published LinkedIn post showing your name, the required explanation, and the attached Pod screenshot

![Output](screenshots/LinkedIn-2.png)

---

### Post Link

https://www.linkedin.com/posts/ginikandubuisi_dmi-kubernetes-devops-share-7531689819320545280-wyFX/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

---

# Submission Instructions

- Add all required screenshots, numbered 01–10, to this template.
- Full Name must be visible in required screenshots.
- Ensure screenshots are clear and readable.
- Submit the final `nginx-replicaset.yaml` file showing `replicas: 5`. Screenshot 02 provides evidence of the initial manifest with `replicas: 3`.
- Complete the Notes sections in Tasks 1–4 in your own words.
- Include the published LinkedIn post URL.
- Do not expose tokens, passwords, private keys, kubeconfig files, or account IDs.

---

# Completion Checklist

- [ ] Completed Task 0 and confirmed that the previous `nginx-pod` is no longer listed
- [ ] Created and entered the ReplicaSet working directory (Screenshot 01)
- [ ] Created `nginx-replicaset.yaml` with an initial replica count of three (Screenshot 02)
- [ ] Applied the ReplicaSet manifest successfully (Screenshot 03)
- [ ] Confirmed three NGINX Pods are running (Screenshot 04)
- [ ] Deleted one ReplicaSet-managed Pod (Screenshot 05)
- [ ] Observed the automatically created replacement Pod (Screenshot 06)
- [ ] Confirmed three NGINX Pods are running again (Screenshot 07)
- [ ] Changed the replica count from three to five (Screenshot 08)
- [ ] Confirmed five NGINX Pods are running (Screenshot 09)
- [ ] Submitted the final `nginx-replicaset.yaml` file with `replicas: 5`
- [ ] Completed the Notes sections in Tasks 1–4
- [ ] Published the LinkedIn post with the required explanation and Pod screenshot (Screenshot 10)
- [ ] Included the published LinkedIn post URL
- [ ] Included all required screenshots
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
