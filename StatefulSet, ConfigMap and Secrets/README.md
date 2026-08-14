### StatefulSets

- StatefulSets are used in stateful applications like MySQL, MongoDB, etc, where the data should always be persisted and could not be compromised and the transactions like read-write should always work properly.
- StatefulSets creates the pod in a sequence and always carries a state with them meaning that the pods are always numbered in sequence like pod-0, pod-1 & so.
- If one of the pod crashes or gets deleted, then a new pod will be created with the same name and the same state of the previous pod. 

StatefulSets uses headless service which is not exposed and used for internal use.

---
### ConfigMaps & Secrets

ConfigMaps 

- ConfigMaps are used to store the environment variables and other variables and values.
- The purpose of using ConfigMaps is whenever you want to change the variables/values in the database, you don’t need to change the statefulset manifest file everytime, simply modify the configmap manifests and the changes will be applied to your stateful application.

Secrets

- Secrets are used to store the password by encoding it and storing it. It is encoded in base64 encoder. 
- Whenever this secret manifest is created, a binary file is created.
- In case of ConfigMaps, the values are in plain-text but in secrets, the values are encoded.

---
Steps to implement StatefulSet with ConfigMap and Secrets

Create a configmap manifest file
```bash
vim configmap.yml
```
Copy the contents in the pv.yml file
```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
```
Apply the manifest:
```bash
kubectl apply -f configmap.yml
```
Verify:
```bash
kubectl get configmap -n mysql
```
Expected Output
```bash
NAME               DATA   AGE
kube-root-ca.crt   1      3d23h
mysql-config-map   1      3d22h
```
---
Create a secret manifest file
```bash
vim secrets.yml
```
Copy the contents in the pvc.yml file
```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
data:
  MYSQL_ROOT_PASSWORD: cm9vdAo=
```
Apply the manifest:
```bash
kubectl apply -f secrets.yml
```
Verify:
```bash
kubectl get secrets -n mysql
```
Expected Output
```bash
NAME           TYPE     DATA   AGE
mysql-secret   Opaque   1      3d2h
```
---
Now create a StatefulSet manifest file
```bash
vim statefulset.yml
```
Copy the contents in the pvc.yml file
```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
  namespace: mysql
spec:
  serviceName: mysql-service
  replicas: 5
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-config-map
              key: MYSQL_DATABASE
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Apply the manifest:
```bash
kubectl apply -f statefulset.yml
```
Verify:
```bash
kubectl get statefulset -n mysql
```
Expected Output
```bash
NAME                READY   AGE
mysql-statefulset   5/5     3d22h
```
---
Once this is done, you can login to the database and verify the password works and the database is created.

```bash
kubectl get pods -n mysql
```
Output
```bash
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          3d2h
mysql-statefulset-1   1/1     Running   0          3d2h
mysql-statefulset-2   1/1     Running   0          3d2h
mysql-statefulset-3   1/1     Running   0          3d2h
mysql-statefulset-4   1/1     Running   0          3d2h
```
Get inside the container
```bash
 kubectl exec -it mysql-statefulset-0 -n mysql -- bash
```
Output
```bash
bash-5.1# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.46 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| devops             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.11 sec)
```
---

