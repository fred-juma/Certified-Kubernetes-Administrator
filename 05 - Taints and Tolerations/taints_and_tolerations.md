
#### List all nodes
```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   10m     v1.24.0
node01  
```
#### Show taint on a node
```bash
controlplane ~ ➜  kubectl describe node node01 | grep taint
```

#### Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule

```bash
controlplane ~ ➜  kubectl taint nodes node01 spray=mortein:NoSchedule
node/node01 tainted

controlplane ~ ➜  
```

#### Apply label to a node

```bash
controlplane ~ ➜ kubectl label node node01 color=blue
node/node01 labeled

controlplane ~ ➜  
```

#### Create a new pod with the nginx image and pod name as mosquito

```bash
controlplane ~ ➜  kubectl run mosquito --image=nginx
pod/mosquito created 

controlplane ~ ➜  
```

#### pod status is pending because pod mosquito cannot tolerate taint mortein on node01

```bash
controlplane ~ ➜  kubectl get pods 
NAME       READY   STATUS    RESTARTS   AGE
mosquito   0/1     Pending   0          23s

controlplane ~ ➜  
```
#### Create another pod named bee with the nginx image, which has a toleration set to the taint mortein

```bash

controlplane ~ ➜  kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml


controlplane ~ ➜  vi bee.yaml 

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee

  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```

#### Apply the pod definition manifest

```bash
controlplane ~ ➜  kubectl apply -f bee.yaml 
pod/bee created

controlplane ~ ➜  
```

***The End***