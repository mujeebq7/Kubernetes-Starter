## Jobs & Cron Jobs

Jobs:

- A job is a task that the container will run and exit. It can be a backup task, patching or update or creating a folder
- You can run the job in a single thread or parallel as well.

CronJobs:

- A cron job is a job that will run on a scheduled time.
- You need to specify the schedule in the manifest file.
---

Let's create a sample Job

```bash
vim job.yml
```
Copy this manifest file in job.yml
```bash
kind: Job
apiVersion: batch/v1
metadata:
  name: demo-job
  namespace: nginx
spec:
  completions: 1
  parallelism: 1
  template:
    metadata:
      name: demo-job-pod
      labels:
        app: demo-job
    spec:
      containers:
      - name: demo-job
        image: busybox:latest
        command: ["/bin/sh", "-c", "echo Hello World! && sleep 5"]
      restartPolicy: Never
```
Once done, apply the manifest file
```bash
kubectl apply -f job.yml
```
Verify:
```bash
kubectl get job -n nginx
```
```bash
kubectl get pods -n nginx
```
You can also verify the logs for the pod to check if the logs displayed the command you ran
```bash
kubectl logs pod/pod_name -n nginx
```
Expeced output:
```bash
Hello World!
```
---
Let's create a sample CronJob

```bash
vim cronjob.yml
```

Copy this manifest file in cronjob.yml
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
  namespace: nginx
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
Once done, apply the manifest file
```bash
kubectl apply -f cronjob.yml
```
Verify:
```bash
kubectl get cronjob -n nginx
```
```bash
kubectl get pods -n nginx
```
You can also verify the logs for the pod to check if the logs displayed the command you ran
```bash
kubectl logs pod/pod_name -n nginx
```
Expeced output:
```bash
Sat Aug  8 11:54:00 UTC 2026
Hello from the Kubernetes cluster
```
