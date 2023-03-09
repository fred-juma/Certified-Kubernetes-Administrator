#### Task

Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

#### Create deployment manifest

```bash
controlplane ~ ➜  kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1 -o yaml --dry-run=client > nginx-deploy.yaml

controlplane ~ ➜  
```

Update the manifest file to add *update strategy*

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deploy
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```

Apply the deployment definition

```bash

controlplane ~ ➜  kubectl apply -f nginx-deploy.yaml 
deployment.apps/nginx-deploy created

controlplane ~ ➜  
```

View all resources created

```bash

controlplane ~ ➜  kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/gold-nginx-69d8d76f55-5rmhx    1/1     Running   0          12m
pod/nginx-deploy-648567495-kkm99   1/1     Running   0          46s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gold-nginx     1/1     1            1           12m
deployment.apps/nginx-deploy   1/1     1            1           46s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/gold-nginx-69d8d76f55    1         1         1       12m
replicaset.apps/nginx-deploy-648567495   1         1         1       46s

controlplane ~ ➜  
```

Set the new image

```bash
controlplane ~ ➜  kubectl set image deployment nginx-deploy nginx=nginx:1.17 --record

deployment.apps/nginx-deploy image updated

controlplane ~ ➜   
```

View rollout history

```bash
controlplane ~ ➜  kubectl rollout history deployment nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deploy nginx=nginx:1.17 --record=true

controlplane ~ ➜   
```


***The End***