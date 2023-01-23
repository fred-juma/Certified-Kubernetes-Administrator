#### Image security

image: nginx

Actual path:

image: docker.io/library/nginx

- docker.io is the registry
- library is the user account
- nginx is the image/repository

Private registry

First log in to the docker registry with your credentials

docker login private-registry.io

Then once successful run the application using the image from the registry

docker run private-registry.io/apps/internal-app

To use an image from the private registry in kubernetes: 

- First create a secret with the credentials in it.
- The secret is of type docker-registry 

kubectl create secret docker-registry regcred \
--docker-server=private-registry.io \
--docker-username=registry-user \
--docker-password=registry-password \
--docker-email=registry-user@org.com 


- Then specify the secret in the pod definition to pass the credentials

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
  imagePullSecrets:
    - name: regcred
```

We have an application running on our cluster. Let us explore it first. What image is the application using? **nginx:alpine**

```bash
root@controlplane ~ ➜  kubectl get deployment -o wide
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
web    2/2     2            2           80s   nginx        nginx:alpine   app=web

root@controlplane ~ ➜  
```
We decided to use a modified version of the application from an internal private registry. Update the image of the deployment to use a new image from myprivateregistry.com:5000

The registry is located at myprivateregistry.com:5000. Don't worry about the credentials for now. We will configure them in the upcoming steps.

First describe the deployment to get the image name **nginx**

```bash
root@controlplane ~ ➜  kubectl describe deployment web
Name:                   web
Namespace:              default
CreationTimestamp:      Mon, 23 Jan 2023 08:42:11 +0000
Labels:                 app=web
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx:
    Image:        nginx:alpine
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-fd5cb786 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  4m3s  deployment-controller  Scaled up replica set web-fd5cb786 to 2
  ```

  then update the container image 

  ```bash
  root@controlplane ~ ➜  kubectl set image deployment/web nginx=myprivateregistry.com:5000/nginx:alpine
deployment.apps/web image updated

root@controlplane ~ ➜  
```

View pods. We note that the pod created with the new image is not running. The error is image pull error. 

```bash
root@controlplane ~ ➜  kubectl get pods
NAME                  READY   STATUS         RESTARTS   AGE
web-87bb989dc-thdm2   0/1     ErrImagePull   0          104s
web-fd5cb786-h2fdf    1/1     Running        0          9m24s
web-fd5cb786-zkxx6    1/1     Running        0          9m24s

root@controlplane ~ ➜  
```

Create a secret object with the credentials required to access the registry.

Name: private-reg-cred
Username: dock_user
Password: dock_password
Server: myprivateregistry.com:5000
Email: dock_user@myprivateregistry.com

```bash
root@controlplane ~ ➜  kubectl create secret docker-registry private-reg-cred \
> --docker-server=myprivateregistry.com:5000 \
> --docker-username=dock_user \
> --docker-password=dock_password \
> --docker-email=dock_user@myprivateregistry.com 
secret/private-reg-cred created

root@controlplane ~ ➜  

```

Configure the deployment to use credentials from the new secret to pull images from the private registry

root@controlplane ~ ➜ kubectl get deployment/web -o yaml > deployment.yaml

root@controlplane ~ ➜  vi deployment.yaml 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: myprivateregistry.com:5000/nginx:alpine
        name: nginx
      imagePullSecrets:
      - name: private-reg-cred
      restartPolicy: Always
      schedulerName: default-scheduler
      ```

Delete the previous deployment

```bash
root@controlplane ~ ➜  kubectl delete deployment web
deployment.apps "web" deleted
```

Apply the updated deployment

```bash
root@controlplane ~ ➜  kubectl apply -f deployment.yaml 
deployment.apps/web created

root@controlplane ~ ➜  
```
Check pod status

```bash
root@controlplane ~ ➜  kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-844574b8cd-fk7ln   1/1     Running   0          20s
web-844574b8cd-vqsx7   1/1     Running   0          20s
```

***The End***