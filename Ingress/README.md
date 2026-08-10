### Ingress

- Ingress is used to re-route the traffic in your cluster. It is used to manage multiple microservices to re-route the traffic.
- It is a resource that allows you to manage traffic and routes.

Ingress Controller : It is used to help routing on a cluster-level.

---
Steps to implement Ingress

Run this to apply the Ingress Controller manifest file
```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```
Once the manifest is applied, a namespace named 'ingress-nginx' will be created.
Also pods and service will also be created

```bash
kubectl get ns
```
Expected output:
```bash
NAME                   STATUS   AGE
default                Active   2d18h
ingress-nginx          Active   30s
kube-node-lease        Active   2d18h
kube-public            Active   2d18h
kube-system            Active   2d18h
kubernetes-dashboard   Active   47h
local-path-storage     Active   2d18h
nginx                  Active   2d
```
```bash
kubectl get pods -n ingress-nginx
```
Expected output:
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-mmrnx        0/1     Completed   0          3m44s
ingress-nginx-admission-patch-bx774         0/1     Completed   1          3m44s
ingress-nginx-controller-7b9b9dc95d-czmvk   1/1     Running     0          3m44s
```
```bash
kubectl get svc -n ingress-nginx
```
Expected output
```bash
NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.239.98   <pending>     80:30668/TCP,443:31387/TCP   4m21s
ingress-nginx-controller-admission   ClusterIP      10.96.13.74    <none>        443/TCP                      4m21s
```
---
We have provided the manifests file for Nginx, Apache and Ingress in this section, apply those manifests file.
```bash
kubectl apply -f nginx.yml
```
```bash
kubectl apply -f apache.yml
```
```bash
kubectl apply -f ingress.yml
```
Verify:
```bash
kubectl get all -n nginx
```
---
You will need to expose the ingress which is running through the Ingress Controller
```bash
kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 80:80 --address=0.0.0.0
```
Once done, verify it on the browser
- IP address + /apache should open the Apache web page
- IP address + /nginx should open the Nginx web page


