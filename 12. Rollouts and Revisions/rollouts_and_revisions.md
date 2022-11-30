
#### Rollouts and revisions

View esources in the cluster

```bash
controlplane ~ ➜  kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/frontend-7494d64f7-kmrn6   1/1     Running   0          15m
pod/frontend-7494d64f7-wcgvs   1/1     Running   0          15m
pod/frontend-7494d64f7-ldccs   1/1     Running   0          15m
pod/frontend-7494d64f7-8zk2t   1/1     Running   0          15m

NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1    <none>        443/TCP          21m
service/webapp-service   NodePort    10.43.92.7   <none>        8080:30080/TCP   15m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend   4/4     4            4           15m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend-7494d64f7   4         4         4       15m

Rollout status

controlplane ~ ➜  kubectl rollout status deployment.apps/frontend
deployment "frontend" successfully rolled out

controlplane ~ ➜  
```

View rollout revisions and history

```bash
controlplane ~ ➜  kubectl rollout history deployment.apps/frontend
deployment.apps/frontend 
REVISION  CHANGE-CAUSE
1         <none>


controlplane ~ ➜   
```
**Deployment strategies**



**Recreate** - replicaSet scaled down to zero. then bring up new replicaset. There is application downtime involved

**Rolling update** - Pods are upgraded few ata time. seamless, no downtime.

Describe deployment

```bash
controlplane ~ ➜  kubectl describe deployment frontend
Name:                   frontend
Namespace:              default
CreationTimestamp:      Wed, 30 Nov 2022 07:11:43 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        20
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/webapp-color:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-7494d64f7 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set frontend-7494d64f7 to 4

kubectl get replicasets

Changing deployment image

View old image, and the container name

controlplane ~ ✖ kubectl describe pod/frontend-7494d64f7-kmrn6
Name:         frontend-7494d64f7-kmrn6
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.83
Start Time:   Wed, 30 Nov 2022 07:11:43 +0000
Labels:       name=webapp
              pod-template-hash=7494d64f7
Annotations:  <none>
Status:       Running
IP:           10.42.0.10
IPs:
  IP:           10.42.0.10
Controlled By:  ReplicaSet/frontend-7494d64f7
Containers:
  simple-webapp:
    Container ID:   containerd://b8db0ce331bebfaa35126dccdae383c40588dd66467b6a29065bd5ebf369af17
    Image:          kodekloud/webapp-color:v1
    Image ID:       docker.io/kodekloud/webapp-color@sha256:27b1e0cbd55a646824c231c896bf77f8278f2d335c4f2b47cbb258edf8281ceb
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
```

Upgrade the deployment image to *kodekloud/webapp-color:v2*

```bash
controlplane ~ ➜  kubectl set image deployment.apps/frontend simple-webapp=kodekloud/webapp-color:v2
deployment.apps/frontend image updated
```

View replicaSets, notice the previous replicaSet drained of all pods, while the rolledout replicaSet having all the pods

```bash
kubectl get replicasets

controlplane ~ ➜  kubectl get replicasets
NAME                 DESIRED   CURRENT   READY   AGE
frontend-b9c6c8d8    4         4         4       29m
frontend-7494d64f7   0         0         0       47m

controlplane ~ ➜  
```
Note that the deployment definition file remains unchanged

Rollback update to previous revision

```bash
controlplane ~ ➜  kubectl rollout undo deployment.apps/frontend
```

***The End***




