## Node Affinity

Node Affinity is a Kubernetes scheduling rule that constrains or influences which nodes are eligible to run a Pod, based on labels assigned to those nodes.

It is defined in a Pod’s **spec.affinity.nodeAffinity** field and allows you to express:
- Required placement rules - the Pod can run only on matching nodes.
- Preferred placement rules - the scheduler tries to use matching nodes but can choose others if necessary.

For example, a Pod can require nodes labeled disktype=ssd or prefer nodes labeled region=us-east.

---

### Common types:

requiredDuringSchedulingIgnoredDuringExecution - strict rule; Pod must match.
preferredDuringSchedulingIgnoredDuringExecution - preference; scheduler tries to match but may choose another node.

Node Affinity is evaluated by the Kubernetes scheduler when placing a Pod. It does not directly select or create nodes; it uses existing node labels to make scheduling decisions.

---

### Simple Manifest

First, label a node:

```bash
kubectl label nodes worker-1 disktype=ssd
```
Then create a Pod that requires that label:
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx:latest
```
Apply it:
```bash
kubectl apply -f nginx.yml
```
The Pod can run only on nodes with:
```bash
disktype=ssd
```
If no node has that label, the Pod remains Pending.

---

### Example with Deployment

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              preference:
                matchExpressions:
                  - key: workload
                    operator: In
                    values:
                      - web
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

Here Kubernetes prefers nodes labeled:
```bash
kubectl label nodes worker-1 workload=web
```
But because this is a preference rather than a requirement, Pods may still run on other nodes if necessary.

IgnoredDuringExecution means that if a node's labels change after the Pod is scheduled, Kubernetes does not automatically evict the existing Pod.
