#### Imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. 


#### Create an NGINX Pod

```bash
kubectl run nginx --image=nginx
```
#### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

#### Create a deployment

```bash
kubectl create deployment --image=nginx nginx
```

#### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

#### Generate Deployment with 4 Replicas

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

#### Scale a deployment

```bash
kubectl scale deployment nginx --replicas=4
```

#### Another way to do this is to save the YAML definition to a file and modify. You can then update the YAML file with the replicas or any other field before creating the deployment.

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

#### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379. 

#### This will automatically use the pod's labels as selectors

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

#### This option below will not use the pods labels as selectors, instead it will assume selectors as **app=redis**. 

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

 #### You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service

 #### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

 ```bash
 kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

 #### This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod


 ### OR

 ```bash
 kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

#### This will not use the pods labels as selectors


Create a new namespace called dev-ns

```bash

controlplane ~ ➜  kubectl create namespace dev-ns
namespace/dev-ns created

controlplane ~ ➜  kubectl get namespaces
NAME              STATUS   AGE
default           Active   18m
kube-system       Active   18m
kube-public       Active   18m
kube-node-lease   Active   18m
dev-ns            Active   69s
```

#### List pod(s) that with the following labels: bu=finance, env=prod, tier=frontend

```bash
controlplane ~ ➜  kubectl get pods --selector bu=finance,env=prod,tier=frontend
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          14m

controlplane ~ ➜  
```

#### List all objects with label bu=finance

```bash
controlplane ~ ➜  kubectl get all --selector bu=finance
NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-ksdmv   1/1     Running   0          6m24s
pod/db-2-8wvp2    1/1     Running   0          6m24s
pod/app-1-4zvqw   1/1     Running   0          6m24s
pod/auth          1/1     Running   0          6m24s
pod/app-1-9dsb2   1/1     Running   0          6m24s
pod/app-1-zzxdf   1/1     Running   0          6m23s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.43.238.199   <none>        3306/TCP   6m23s
Note:
```

//To list the count only: kubectl get all --selector bu=finance --no-headers | wc -l



--dry-run - By default as soon as the command is run, the resource will be created

--dry-run=client - Does not  create the resource, instead, tell you whether the resource can be created and if your command is right

-o yaml - output the resource definition in YAML format on screen

***End***