### Service in Kubernetes

- The Service API is an abstraction to help you expose groups of pods over a network.
- In short, service is used to expose your pods to the outer world.
---
There are different types of services 

**NodePort** : The cluster will assign the IP address with a port ranging from 30000

**ClusterIP** : IP address will be assigned by the cluster

**Load Balancer** : It is used when the service is in cloud

**Headless Service** : It is a service used for internal use, not exposed to the outer world

**External IP** : This service is used when you want to assign a specific IP address

---
### Steps to implement Services in Kubernetes

Create a service manifest file
```bash
vim service.yml
```
Copy the contents in the pv.yml file
```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
  ```
Apply the manifest:
```bash
kubectl apply -f service.yml
```
Verify:
```bash
kubectl get svc -n nginx
```
Expected Output
```bash
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.96.213.19   <none>        80/TCP    30m
```

If you are using kind-cluster and your cluster is a Docker container, then you will need to port-forward the port of the container to the host machine
```bash
kubectl port-forward service/nginx-service -n nginx 80:80 --address=0.0.0.0
```
You should see something similar if the port-forwarding works fine
```bash
Forwarding from 0.0.0.0:80 -> 80
```
Once you get this output, open the browser and verify
```bash
Welcome to nginx!
If you see this page, nginx is successfully installed and working. Further configuration is required for the web server, reverse proxy, API gateway, load balancer, content cache, or other features.

For online documentation and support please refer to nginx.org.
To engage with the community please visit community.nginx.org.
For enterprise grade support, professional services, additional security features and capabilities please refer to f5.com/nginx.

Thank you for using nginx.
```
