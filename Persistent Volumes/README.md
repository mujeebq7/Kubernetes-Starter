### Persistent Volume(PV) and Persistent Volume(PVC)

In Kubernetes, storage is in two layers.

Persistent Volume (PV)
- A PV is the actual storage resource in the cluster.
- Represents real backing storage (EBS, Azure Disk, NFS, Ceph, local-path, etc.).
- Has capacity, access modes, reclaim policy, and storage class.

PersistentVolumeClaim (PVC)
- A PVC is a Pod/user request claiming for storage.
- You ask for size, access mode, and optionally storageClass.
- Kubernetes finds/binds a matching PV, or dynamically provisions one.
- Pods mount the PVC, not the PV directly (in most cases).

Simple analogy:
- PV = assigns storage from the disk
- PVC = request/reservation for that storage
- Pod = app that mounts and uses that reserved storage

Typical flow:
- Pod uses a PVC. 
- PVC binds to a PV (existing or newly provisioned).
- Pod mounts claim path and reads/writes data.
- If Pod is deleted and recreated, data remains on PV.
---
### Important fields to know :
1. Access modes:
- ReadWriteOnce(RWO): mounted read-write by one node
- ReadOnlyMany(ROX): read-only by many nodes
- ReadWriteMany(RWX): read-write by many nodes
  
2. Reclaim policy (on PV):

- Delete: delete underlying storage when released
- Retain: keep data for manual recovery

3. StorageClass: 
- It defines provisioner and storage parameters. 
- It can be a local storage (host machine), cloud storage, network storage, etc 
---

### Steps to implement Persistent Volume and Persistent Volume Claim

Create a persistent volume manifest file
```bash
vim pv.yml
```
Copy the contents in the pv.yml file
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
  namespace: nginx
  labels:
    app: local
spec:
  capacity: 
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  hostPath:
    path: /mnt/data
```
Apply the manifest:
```bash
kubectl apply -f pv.yml
```
Verify:
```bash
kubectl get pv -n nginx
```
Expected Output
```bash
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   1Gi        RWO            Retain           Available           local-storage   <unset>                          9s
```
---
Let's create a persistent volume claim manifest file
```bash
vim pvc.yml
```
Copy the contents in the pvc.yml file
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```
Apply the manifest:
```bash
kubectl apply -f pvc.yml
```
Verify:
```bash
kubectl get pvc -n nginx
```
Expected Output
```bash
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
local-pvc   Bound    local-pv   1Gi        RWO            local-storage   <unset>                 51s
```
---
Now, let's create a pod with a volume that the pvc will be attached to
```bash
vim pod.yml
```
Copy the contents in the pod.yml file
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
      - mountPath: /usr/share/nginx/html/
        name: nginx-storage
  volumes:
  - name: nginx-storage
    persistentVolumeClaim:
      claimName: local-pvc
```
Apply the manifest:
```bash
kubectl apply -f pod.yml
```
Verify:
```bash
kubectl get pods -n nginx
```
---
If you want to verify if the data is persistent from the pod to the host machine

Check on which worker node is your pod running
```bash
kubectl get pods -n nginx -o wide
```
In this example, the pod is running on test-cluster-worker
```bash
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE                  NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          13s   10.244.1.14   test-cluster-worker   <none>           <none>
```
Get the container ID of the test-cluster-worker
```bash
docker ps
```
You will see something similar
```bash
CONTAINER ID   IMAGE                                 COMMAND                  CREATED        STATUS        PORTS                                                                                                          NAMES
409c56d807a9   kindest/node:v1.35.0                  "/usr/local/bin/entr…"   46 hours ago   Up 46 hours                                                                                                                  test-cluster-worker
6f2d34a890cb   kindest/node:v1.35.0                  "/usr/local/bin/entr…"   46 hours ago   Up 46 hours                                                                                                                  test-cluster-worker2
5560c9c7cbd6   kindest/node:v1.35.0                  "/usr/local/bin/entr…"   46 hours ago   Up 46 hours   127.0.0.1:46339->6443/tcp, 0.0.0.0:8080->30080/tcp                                                             test-cluster-control-plane
```
Get inside the container and verify if the mounted path exists
```bash
docker exec -it <container-id> bash
```
Once inside the container, verify if the mounted path exists and data is stored there.
