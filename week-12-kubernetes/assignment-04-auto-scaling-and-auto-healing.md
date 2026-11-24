# Assignment 4 — Auto-Scaling & Auto-Healing: Let Kubernetes React for You

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Create Deployments, test Pod auto-healing, configure CPU requests and limits, verify Metrics Server, and create a Horizontal Pod Autoscaler.

Optionally generate CPU load to observe how the HPA responds.

---

# Task 0 — Check the Existing Cluster Resources

## Goal

Confirm that you are connected to your learning cluster and inspect existing Deployments and HPAs.

### Evidence

No screenshots required.

---

# Task 1 — Set Up Your Lab Directory

## Goal

Create and enter a dedicated directory for the auto-scaling lab.

### Evidence

#### Screenshot 1 — Terminal showing that you are working inside `~/k8s-labs/autoscaling`

![Output](screenshots/Autoscaling-Directory.png)

---

### Notes

**1. What is the difference between auto-healing and auto-scaling?**

Auto-healing replaces Pods that fail or are deleted so the replica count stays the same while auto-scaling changes the number of Pods based on load, for example CPU usage.

---

**2. Which Kubernetes feature will you use for CPU-based auto-scaling?**

The Horizontal Pod Autoscaler (HPA).

---

# Task 2 — Test Auto-Healing with a Deployment

## Goal

Create a Deployment with two replicas, delete one Pod, and observe Kubernetes create a replacement Pod.

### Evidence

#### Screenshot 2 — Completed `autohealing-deployment.yaml` manifest

![Output](screenshots/Autohealing-deployment.png)

---

#### Screenshot 3 — Output of `kubectl get pods` showing two running Pods before deletion

![Output](screenshots/Running-pods2.png)

---

#### Screenshot 4 — Output of the `kubectl delete pod` command

![Output](screenshots/Autohealing-deletion.png)

---

#### Screenshot 5 — Output of `kubectl get pods` showing the replacement Pod and two running Pods

![Output](screenshots/Replaced-pod.png)

---

### Notes

**1. What happened after you deleted one Pod?**

Kubernetes automatically created a new `demo-autohealing` Pod to replace the deleted one.

---

**2. What Kubernetes resource maintains the desired replica count for this Deployment?**

The ReplicaSet that the Deployment creates and manages.

---

**3. What proves that auto-healing worked?**

`kubectl get pods` shows the same number of `demo-autohealing` Pods running again, including a new Pod with a different name and a younger age.

---

# Task 3 — Create a Deployment for HPA

## Goal

Create a Deployment with CPU resource requests and limits so that HPA can calculate CPU utilization.

### Evidence

#### Screenshot 6 — Completed `hpa-deployment.yaml` manifest showing CPU requests and limits

![Output](screenshots/HPA-Deployment.png)

---

#### Screenshot 7 — Output showing `nginx-deployment` and its running Pods

![Output](screenshots/Outputs.png)

---

### Notes

**1. What does `cpu: "100m"` mean?**

It means 100 millicores, which is 0.1 (one-tenth) of a CPU core.

---

**2. What is the difference between a CPU request and a CPU limit?**

A request is the minimum CPU guaranteed to the container, and Kubernetes uses it to schedule the Pod. A limit is the maximum CPU the container is allowed to use.


---

**3. Why does CPU-based HPA require CPU requests?**


HPA measures CPU usage as a percentage of the request. Without a request, it has nothing to compare against, so it can't decide when to scale.

---

# Task 4 — Enable and Verify Metrics Server

## Goal

Ensure the cluster can collect resource metrics for the HPA.

### Evidence

#### Screenshot 8 — Output showing Metrics Server running, or `kubectl top pods` displaying Pod metrics

![Output](screenshots/Top-pods.png)

---

### Notes

**1. Why does HPA need Metrics Server?**

Metrics Server collects the CPU and memory usage of Pods. HPA reads this data to decide when to scale up or down.


---

**2. What issue can occur if Metrics Server is not working?**

HPA can't get metrics, so it shows `<unknown>` for CPU and doesn't scale. `kubectl top pods` also fails.


---

**3. What does `kubectl top pods` help you check?**

It shows how much CPU and memory each Pod is using right now.

---

# Task 5 — Create a Horizontal Pod Autoscaler

## Goal

Create an HPA that keeps at least two Pods running and can scale up to five Pods when average CPU utilization exceeds 50%.

### Evidence

#### Screenshot 9 — The imperative command output OR completed `hpa-nginx.yaml` manifest

![Output](screenshots/Imperative-command-output.png)

---

#### Screenshot 10 — Output of `kubectl get hpa` and the relevant `kubectl describe hpa` command

![Output](screenshots/HPA.png)

---

### Notes

**1. What does `minReplicas: 2` mean?**

HPA will never scale the Deployment below two Pods, even when load is low.

---

**2. What does `maxReplicas: 5` mean?**

HPA will never scale the Deployment above five Pods, even when load is high.

---

**3. What does `averageUtilization: 50` mean?**

HPA tries to keep average CPU usage across the Pods at 50% of their CPU request. It adds Pods when usage goes above 50% and removes Pods when usage drops below it.

---

**4. Why should only one HPA manage a Deployment at a time?**

Two HPAs on the same Deployment can give conflicting scaling decisions, so the Pod count would keep changing back and forth.

---

# Task 6 — Optional: Generate CPU Load and Observe Scaling

## Goal

Generate CPU load in one NGINX Pod and observe how HPA responds.

### Evidence

If you skip this task, write “Not completed — optional” and leave the remaining Task 6 placeholders blank.

Task status: Add your status here.

#### Screenshot 11 — Terminal showing the CPU-load command running inside the selected Pod

![Output](screenshots/CPU-load-command.png)

---

#### Screenshot 12 — Output of `kubectl get hpa` and `kubectl get pods` while HPA evaluates or changes the replica count

![Output](screenshots/get-hpa-w.png)
![Output](screenshots/get-hpa.png)

---

### Notes

Complete these answers only if you perform the optional test.

**1. Did the HPA change the number of Pods? Explain what you observed.**

Yes. When I ran a CPU load, CPU usage rose to 100% against the 50% target, and the HPA scaled the Deployment from 2 to 4 replicas. CPU then settled at 50%/50% because the load was spread across more Pods.

---

**2. If scaling did not happen, what are two possible reasons?**

Not applicable — scaling occurred.

---

**3. Why is automatic scale-down useful after traffic decreases?**

It removes Pods that are no longer needed, which saves cluster resources and cost while keeping the minimum number of Pods running.

---

# LinkedIn Requirement

### Evidence

#### Screenshot 13 — Published LinkedIn post showing your name, the required explanation, and the attached Kubernetes screenshot

![Output](screenshots)

#### Linkedin Post Link

Add your link here

---

# Submission Instructions

- Include required Screenshots 1–10 and Screenshot 13.
- Include Screenshots 11–12 only if you complete Task 6.
- Full Name must be visible in required screenshots.
- Ensure screenshots are clear and readable.
- Include the completed `autohealing-deployment.yaml` and `hpa-deployment.yaml` files.
- Include `hpa-nginx.yaml` if you selected the declarative HPA method.
- Complete the Notes sections for Tasks 1–5 in your own words.
- Complete Task 6 notes only if you perform the optional test.
- Do not expose passwords, tokens, private keys, kubeconfig files, or account IDs.

---

# Completion Checklist

- [ ] Checked the active context and existing cluster resources
- [ ] Created the autoscaling working directory (Screenshot 1)
- [ ] Created `autohealing-deployment.yaml` (Screenshot 2)
- [ ] Confirmed two auto-healing Pods were running (Screenshot 3)
- [ ] Deleted one managed Pod (Screenshot 4)
- [ ] Confirmed that the Pod was automatically replaced (Screenshot 5)
- [ ] Created `hpa-deployment.yaml` with CPU requests and limits (Screenshot 6)
- [ ] Verified `nginx-deployment` and its running Pods (Screenshot 7)
- [ ] Verified Metrics Server and confirmed Pod metrics are available (Screenshot 8)
- [ ] Created one HPA using the selected method (Screenshot 9)
- [ ] Verified minimum replicas of 2, maximum replicas of 5, and a CPU target of 50% (Screenshot 10)
- [ ] Completed Notes for Tasks 1–5
- [ ] Completed Task 6 evidence and notes, or marked it “Not completed — optional”
- [ ] Stopped the CPU-load command and exited the container shell, if Task 6 was completed
- [ ] Published the LinkedIn post with the required explanation and attached screenshot (Screenshot 13)
- [ ] Included the required manifest files
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
