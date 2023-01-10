
#### Cluster Maintenance: Cordoning and uncordoning nodes

Let us explore the environment first

List nodes

```bash

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   9m26s   v1.24.0
node01         Ready    <none>          8m47s   v1.24.0

controlplane ~ ➜  
```

Which nodes are the applications hosted on?

```bash
controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-797fc567b4-85wzg   1/1     Running   0          43s   10.244.1.2   node01         <none>           <none>
blue-797fc567b4-g4bxf   1/1     Running   0          43s   10.244.0.4   controlplane   <none>           <none>
blue-797fc567b4-tkqv8   1/1     Running   0          43s   10.244.1.3   node01         <none>           <none>

controlplane ~ ➜ 
```

List the applications / deployments

```bash
controlplane ~ ➜  kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   3/3     3            3           13s

controlplane ~ ➜  
```

We need to take node01 out for maintenance. Empty the node of all applications and mark it unschedulable.

Drain node01

```bash
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-r95jj, kube-system/kube-proxy-4hqrt
evicting pod default/blue-797fc567b4-qdk9g
evicting pod default/blue-797fc567b4-pxksk
pod/blue-797fc567b4-qdk9g evicted
pod/blue-797fc567b4-pxksk evicted
node/node01 drained

controlplane ~ ➜  
```
Note: When pod is not part of a replicaSet, the draining will not be effected unless --force option is used
Now checking to ensure all pods have been evicted from node01

```bash
controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-797fc567b4-4ltqn   1/1     Running   0          16s     10.244.0.5   controlplane   <none>           <none>
blue-797fc567b4-6zx8f   1/1     Running   0          16s     10.244.0.6   controlplane   <none>           <none>
blue-797fc567b4-g4bxf   1/1     Running   0          2m51s   10.244.0.4   controlplane   <none>           <none>

controlplane ~ ➜  
```

The maintenance tasks have been completed. Configure the node node01 to be schedulable again.

Uncordon node01

```bash
controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

controlplane ~ ➜  
```
hr-app is a critical app and we do not want it to be removed and we do not want to schedule any more pods on node01.
Mark node01 as unschedulable so that no new pods are scheduled on this node.

Make sure that hr-app is not affected.

Cordon node01

```bash
controlplane ~ ➜  kubectl cordon node01
node/node01 cordoned
```

Cordoning node does not affect pods already scheduled on the node, however, no new pods will be scheduled on the node

```bash
controlplane ~ ➜  kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-797fc567b4-4ltqn     1/1     Running   0          8m3s   10.244.0.5   controlplane   <none>           <none>
blue-797fc567b4-6zx8f     1/1     Running   0          8m3s   10.244.0.6   controlplane   <none>           <none>
blue-797fc567b4-g4bxf     1/1     Running   0          10m    10.244.0.4   controlplane   <none>           <none>
hr-app-5ff955d598-wgp5d   1/1     Running   0          70s    10.244.1.5   node01         <none>           <none>

controlplane ~ ➜  
```

***The End***