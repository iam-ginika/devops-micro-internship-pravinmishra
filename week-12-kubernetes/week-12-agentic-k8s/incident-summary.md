# Kubernetes Incident Summary

**Full Name:**GINIKA NDUBUISI

## 1. Reported Symptom

The Book Review backend became unhealthy after a new container image was rolled out. The backend Pod stopped being ready, and the backend-service Service had no ready backend, so the application could not reach its API.

## 2. Evidence Collected

The k8s-triage.sh script produced a failure report (reports/incident-failure-report.txt). It showed the backend Pod in ErrImagePull or ImagePullBackOff, readiness at 0/1, restart and event warnings from the failed image pull, and no ready endpoints behind backend-service. The report was saved before any recovery action.

## 3. Most Likely Cause

The backend Deployment was pointed at an image tag that does not exist (joy098/book-review-backend:does-not-exist), so the node could not pull the image and the new Pod never started. This is supported by the Pod status and the image pull events in the report.

## 4. Human-Approved Recovery Action

After reviewing Claude's suggested command, I restored the original working image myself with kubectl set image deployment/backend backend=joy098/book-review-backend:v1 -n book-review, then waited for the rollout to finish. Claude did not run any cluster-changing command.

## 5. Verification

The rollout completed successfully, one backend Pod returned to Running and 1/1 Ready, and the backend-service EndpointSlice listed a ready endpoint again. A second /k8s-triage run saved as reports/recovery-report.txt showed no FAIL results and an overall status of HEALTHY.

## 6. Safety Decision

Claude was limited to reading the report and suggesting a command as text. I reviewed every tool request in manual permission mode, denied anything that could change a file or the cluster, and ran the failure and the recovery myself in a regular terminal on my personal lab cluster.

## 7. Agentic Loop Mapping

- Gather: the read-only k8s-triage.sh script collected Pod status, readiness, restarts, Warning events and Service backends, and wrote the report.
- Analyze: Claude read the report through the /k8s-triage skill and explained the failure with evidence.
- Human Act: I reviewed the suggested command and ran the image restore myself.
- Verify: I ran the same script again and compared the recovery report with the failure report.
