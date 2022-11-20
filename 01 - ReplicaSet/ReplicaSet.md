### Create and configure **ReplicaSet** named replicaset-1, container image nginx, label -> tier: frontend and number of replicas is 2.

#### Create the *yaml* defination file as *replicaset-1* 

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

#### Create the replicaSet from the yaml defineition file

```bash
controlplane ~ ➜  kubectl apply -f /root/replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```

#### Show the ReplicaSets in the default cluster
```bash
controlplane ~ ➜  kubectl get replicasets
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       10m
replicaset-1      2         2         2       82s

controlplane ~ ➜  
```

#### Get the properties of the ReplicaSet *replicaset-1*

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

#### Update the replica set to use another image, busybox image. Use the *kubectl* utility with *edit* option to edit the ReplicaSet definition file and update the image

```bash
controlplane ~ ➜  kubectl edit replicaset replicaset-1
replicaset.apps/replicaset-1 edited
```

#### Once you update the image, delete the old pods one by one so that new pods are created with the updated image

#### Scale the ReplicaSet to 5 PODs. Use kubectl scale command and specify the numba od replicas, type (ReplicaSet) and the name (new-replica-set)

```bash
controlplane ~ ➜  kubectl scale --replicas=5 replicaset replicaset-1
replicaset.apps/replicaset-1 scaled

controlplane ~ ➜ 
```

#### Delete the ReplicaSet

```bash

controlplane ~ ➜  kubectl delete replicasets replicaset-1
replicaset.apps "replicaset-1" deleted

controlplane ~ ➜  
```




***The End***