### Instructions
#### Create deployment with the following propertis:
+ image - httpd:2.4-alpine
+ name - httpd-frontend
+ replicas - 3

### Solution
#### Generate a yaml template file with *-o yaml* option but don't create it and that is specified with the option *--dry-run*. This helps eliminate yaml indentation errors.

```bash
controlplane ~ ✖ kubectl create deployment --image=httpd:2.4-alpine httpd-frontend --replicas=3 --dry-run=client -o yaml > httpd-frontend.yaml
```

#### Confirm that the deployment file has been created

```bash
controlplane ~ ➜  ls -l
total 8
-rw-rw-rw-    1 root     root           387 Nov 20 08:43 deployment-definition-1.yaml
-rw-r--r--    1 root     root           431 Nov 20 08:44 httpd-frontend.yaml
-rw-rw-rw-    1 root     root             0 Nov 10 15:21 sample.yaml
```

#### Inspect the created deployment definition file to ensure it is as desired.

```bash
controlplane ~ ➜  vi httpd-frontend.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: httpd-frontend
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd-frontend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd
        resources: {}
status: {}

```

#### Now apply the deployment definition file to create the specified resources

```bash

controlplane ~ ➜  kubectl apply -f httpd-frontend.yaml 
deployment.apps/httpd-frontend created
```

#### View all the resources created: deployment, replicaset and pods

```bash
controlplane ~ ➜  kubectl get all
NAME                                       READY   STATUS             RESTARTS   AGE
pod/deployment-1-7c65c4d9dd-795b6          1/1     Running            0          2m59s
pod/deployment-1-7c65c4d9dd-rg2dt          1/1     Running            0          2m59s
pod/frontend-deployment-6d8c45b946-7tv7v   0/1     ImagePullBackOff   0          7m11s
pod/frontend-deployment-6d8c45b946-96ncj   0/1     ImagePullBackOff   0          7m11s
pod/frontend-deployment-6d8c45b946-6kwws   0/1     ImagePullBackOff   0          7m11s
pod/frontend-deployment-6d8c45b946-pffzq   0/1     ImagePullBackOff   0          7m11s
pod/httpd-frontend-6f67496c45-84tzm        1/1     Running            0          9s
pod/httpd-frontend-6f67496c45-t8r5l        1/1     Running            0          9s
pod/httpd-frontend-6f67496c45-j4k9m        1/1     Running            0          9s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   12m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend-deployment   0/4     4            0           7m11s
deployment.apps/deployment-1          2/2     2            2           2m59s
deployment.apps/httpd-frontend        3/3     3            3           9s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend-deployment-6d8c45b946   4         4         0       7m11s
replicaset.apps/deployment-1-7c65c4d9dd          2         2         2       2m59s
replicaset.apps/httpd-frontend-6f67496c45        3         3         3       9s

controlplane ~ ➜  
```

***The End***