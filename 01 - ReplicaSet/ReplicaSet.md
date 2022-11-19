Create and configure replicaset named replicaset-1, container image nginx, label nginx , replicas 2

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
controlplane ~ ➜  kubectl apply -f /root/replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```

```bash
controlplane ~ ➜  kubectl get replicasets
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       10m
replicaset-1      2         2         2       82s

controlplane ~ ➜  
```

```bash
controlplane ~ ➜  kubectl describe replicasets replicaset-1
Name:         replicaset-1
Namespace:    default
Selector:     tier=frontend
Labels:       <none>
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  3m41s  replicaset-controller  Created pod: replicaset-1-ds4st
  Normal  SuccessfulCreate  3m41s  replicaset-controller  Created pod: replicaset-1-wmmlb

controlplane ~ ➜  
```

```bash

controlplane ~ ➜  kubectl delete replicasets replicaset-1
replicaset.apps "replicaset-1" deleted

controlplane ~ ➜  
```