#### Ingress

- Ingress help users to access your application using a single externally accessible URL that is configurable to route traffic to different services within the cluster based on URL path while implementing SSL security as well.
- Ingress is a L7 Load balancer built into the cluster that can be configured using native kubernetes primitives like any other k8s objects.
- Deployment of ingress comprises ingress controller (reverse proxy solution e.g. nginx, HAProxy, traefik etc.) and ingress resources which are a set of rules 

#### Ingress Controller
- Can be a implemented as GCP HTT(S) Load Balancer, Nginx, Contour, HAProxy, traefik or Istio

Nginx Ingress Controller definition

```yaml

apiVersion: v1
kind: Deployment
metadata:
	name: nginx-ingress-controller
spec:
	replicas: 1
	selector:
		matchLabels:
			name: nginx-ingress
	template:
		metadata:
			labels:
				name: nginx-ingress
		spec:
			containers:
				- name: nginx-ingress-controller
				image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
			args:
				- /nginx-ingress-controller
				- --configmap=$(POD_NAMESPACE)/nginx-configuration
			env:
				- name: POD_NAME
				  valueFrom:
				  	fieldRef:
				  		fieldPath: metadata.name
				- name: POD_NAMESPACE
				  valueFrom:
				  	fieldRef:
				  		fieldPath: metadata.namespace
			ports:
			  - name: http
			    containerPort: 80
			  - name: https
			    containerPort: 443	  		
```

ConfigMap definition to feed the nginx with configuration data

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: nginx-configuration
```

Service configuration to expose the deployment 

```yaml
apiVersion: v1
kind: Service
metadata:
	name: nginx-ingress
spec:
	type: NodePort
	selector:
	  name: nginx-ingress
	ports:
		- port: 80
		  targetPort: 80
		  protocol: TCP
		  name: http 
		- port: 443
		  targetPort: 443
		  protocol: TCP
		  name: https
```

Create a service account with correct permissions, roles and role bindings to access all of these objects

#### Ingress Resources
-  an ingress resource is a set of rules and configurations applied on the ingress controller

Example 1: 1 Rule (www.my-online-store.com), 1 path (/wear)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	name: Ingress-wear
spec:
	backend:
		serviceName: wear-service
		servicePort: 80

```

Example 2: 1 Rule (www.my-online-store.com), 2 paths (/wear and /watch)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	name: Ingress-wear-watch
spec:
	rules:
	- http:
		paths:
			- path: /wear
			  pathType: Prefix
			  backend:
			  	service:
		          name: wear-service
		          port: 
		          	number: 80


			- path: /watch
			  pathType: Prefix
			  backend:
			  	service:
		          name: watch-service
		          port:
		            number: 80
	
```

Example 3: 2 Rules (wear.my-online-store.com & watch.my-online-store.com), 1 path for each URL  (WEAR and VIDEO)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	name: Ingress-wear-watch
spec:
	rules:
	- host: wear.my-online-store.com
	  http:
		paths: 
		- backend:
		    serviceName: wear-service
		    servicePort: 80


	- host: watch.my-online-store.com
	  http:
		paths: 
		- backend:
		    serviceName: wear-service
		    servicePort: 80	
	
```

Imperative way of creating ingress resource

```bash
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```


We have deployed Ingress Controller, resources and applications. Explore the setup.

```bash
controlplane ~ ➜  kubectl get all -A
NAMESPACE       NAME                                            READY   STATUS      RESTARTS   AGE
app-space       pod/default-backend-b46b9989-mnzpl              1/1     Running     0          68s
app-space       pod/webapp-video-57cddcc65-rlqtc                1/1     Running     0          68s
app-space       pod/webapp-wear-658fc8dbb4-mwn4n                1/1     Running     0          68s
ingress-nginx   pod/ingress-nginx-admission-create-p6z78        0/1     Completed   0          64s
ingress-nginx   pod/ingress-nginx-admission-patch-7dv82         0/1     Completed   0          64s
ingress-nginx   pod/ingress-nginx-controller-5f8964959d-jlpq2   1/1     Running     0          64s
kube-flannel    pod/kube-flannel-ds-hq2gh                       1/1     Running     0          112s
kube-system     pod/coredns-787d4945fb-qvcfn                    1/1     Running     0          112s
kube-system     pod/coredns-787d4945fb-zhwfk                    1/1     Running     0          112s
kube-system     pod/etcd-controlplane                           1/1     Running     0          2m7s
kube-system     pod/kube-apiserver-controlplane                 1/1     Running     0          2m7s
kube-system     pod/kube-controller-manager-controlplane        1/1     Running     0          2m2s
kube-system     pod/kube-proxy-gjpxp                            1/1     Running     0          112s
kube-system     pod/kube-scheduler-controlplane                 1/1     Running     0          2m2s

NAMESPACE       NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
app-space       service/default-backend-service              ClusterIP   10.105.234.63    <none>        80/TCP                       68s
app-space       service/video-service                        ClusterIP   10.102.218.249   <none>        8080/TCP                     68s
app-space       service/wear-service                         ClusterIP   10.104.116.224   <none>        8080/TCP                     68s
default         service/kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      2m7s
ingress-nginx   service/ingress-nginx-controller             NodePort    10.107.77.59     <none>        80:30080/TCP,443:32103/TCP   65s
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   10.110.232.180   <none>        443/TCP                      65s
kube-system     service/kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       2m4s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   2m1s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   2m4s

NAMESPACE       NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
app-space       deployment.apps/default-backend            1/1     1            1           68s
app-space       deployment.apps/webapp-video               1/1     1            1           68s
app-space       deployment.apps/webapp-wear                1/1     1            1           68s
ingress-nginx   deployment.apps/ingress-nginx-controller   1/1     1            1           65s
kube-system     deployment.apps/coredns                    2/2     2            2           2m4s

NAMESPACE       NAME                                                  DESIRED   CURRENT   READY   AGE
app-space       replicaset.apps/default-backend-b46b9989              1         1         1       68s
app-space       replicaset.apps/webapp-video-57cddcc65                1         1         1       68s
app-space       replicaset.apps/webapp-wear-658fc8dbb4                1         1         1       68s
ingress-nginx   replicaset.apps/ingress-nginx-controller-5f8964959d   1         1         1       65s
kube-system     replicaset.apps/coredns-787d4945fb                    2         2         2       112s

NAMESPACE       NAME                                       COMPLETIONS   DURATION   AGE
ingress-nginx   job.batch/ingress-nginx-admission-create   1/1           13s        66s
ingress-nginx   job.batch/ingress-nginx-admission-patch    1/1           12s        65s

controlplane ~ ➜  
```


What is the Host configured on the Ingress Resource? The host entry defines the domain name that users use to reach the application like www.google.com *All hosts (*)*

```bash
controlplane ~ ➜  kubectl describe ingress --namespace app-space
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          10.107.77.59
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (10.244.0.4:8080)
              /watch   video-service:8080 (10.244.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    14m (x2 over 14m)  nginx-ingress-controller  Scheduled for sync

controlplane ~ ➜  
```

What backend is the /wear path on the Ingress configured with? **wear-service:8080 (10.244.0.4:8080)**

At what path is the video streaming application made available on the Ingress? **/watch**

If the requirement does not match any of the configured paths what service are the requests forwarded to? **No service**


You are requested to change the URLs at which the applications are made available. Make the video application available at */stream*


```bash
controlplane ~ ➜  kubectl edit ingress --namespace app-space
ingress.networking.k8s.io/ingress-wear-watch edited



# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2023-02-05T12:09:50Z"
  generation: 1
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "714"
  uid: bc58dc07-4b5d-4bb1-9a60-39c94584ed74
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /stream
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 10.107.77.59
 ```

A user is trying to view the /eat URL on the Ingress Service. Which page would he see?


If not open already, click on the Ingress tab above your terminal, and append /eat to the URL in the browser. **The 404 Error Page**

Due to increased demand, your business decides to take on a new venture. You acquired a food delivery company. Their applications have been migrated over to your cluster.


Inspect the new deployments in the app-space namespace.

```bash
controlplane ~ ➜  kubectl get all -n app-space
NAME                                 READY   STATUS    RESTARTS   AGE
pod/default-backend-b46b9989-mnzpl   1/1     Running   0          27m
pod/webapp-food-dcd846f95-r92lg      1/1     Running   0          39s
pod/webapp-video-57cddcc65-rlqtc     1/1     Running   0          27m
pod/webapp-wear-658fc8dbb4-mwn4n     1/1     Running   0          27m

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/default-backend-service   ClusterIP   10.105.234.63    <none>        80/TCP     27m
service/food-service              ClusterIP   10.102.105.52    <none>        8080/TCP   39s
service/video-service             ClusterIP   10.102.218.249   <none>        8080/TCP   27m
service/wear-service              ClusterIP   10.104.116.224   <none>        8080/TCP   27m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/default-backend   1/1     1            1           27m
deployment.apps/webapp-food       1/1     1            1           39s
deployment.apps/webapp-video      1/1     1            1           27m
deployment.apps/webapp-wear       1/1     1            1           27m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/default-backend-b46b9989   1         1         1       27m
replicaset.apps/webapp-food-dcd846f95      1         1         1       39s
replicaset.apps/webapp-video-57cddcc65     1         1         1       27m
replicaset.apps/webapp-wear-658fc8dbb4     1         1         1       27m

controlplane ~ ➜ 
```

You are requested to add a new path to your ingress to make the food delivery application available to your customers.

Make the new application available at /eat.

Edit the ingress resource to add the new /eat backend, service and path 

```bash
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2023-02-05T12:09:50Z"
  generation: 2
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "2673"
  uid: bc58dc07-4b5d-4bb1-9a60-39c94584ed74
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /stream
        pathType: Prefix
      - backend:
          service:
            name: food-service
            port:
              number: 8080
        path: /eat
        pathType: Prefix  
status:
  loadBalancer:
    ingress:
    - ip: 10.107.77.59
```

A new payment service has been introduced. Since it is critical, the new application is deployed in its own namespace.


Identify the namespace in which the new application is deployed. **critical-space**

```bash
controlplane ~ ➜  kubectl get all -n critical-space
NAME                              READY   STATUS    RESTARTS   AGE
pod/webapp-pay-58cdc69889-cxmxn   1/1     Running   0          47s

NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/pay-service   ClusterIP   10.96.155.71   <none>        8282/TCP   47s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-pay   1/1     1            1           47s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-pay-58cdc69889   1         1         1       47s

controlplane ~ ➜  
```

You are requested to make the new application available at /pay.

Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.

Create a new Ingress for the new pay application in the critical-space namespace.
Use the command kubectl get svc -n critical-space to know the service and port details.

```bash
controlplane ~ ➜  kubectl get svc -n critical-space
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
pay-service   ClusterIP   10.96.155.71   <none>        8282/TCP   9m21s

controlplane ~ ➜  
```


```yaml

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282
```
Apply the ingress manifest

```bash
controlplane ~ ➜  kubectl apply -f pay-ingress.yaml 
ingress.networking.k8s.io/test-ingress created

controlplane ~ ➜  
```

We have deployed two applications. Explore the setup.

Ingress Deployment              |  
:-------------------------:|
![deployment](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/ingress.JPG)

```bash
controlplane ~ ➜  kubectl get all -A
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
app-space      pod/default-backend-b46b9989-9jk8m         1/1     Running   0          90s
app-space      pod/webapp-video-57cddcc65-6pmg5           1/1     Running   0          91s
app-space      pod/webapp-wear-658fc8dbb4-wbdph           1/1     Running   0          91s
kube-flannel   pod/kube-flannel-ds-fjhjl                  1/1     Running   0          2m28s
kube-system    pod/coredns-787d4945fb-2d9bj               1/1     Running   0          2m27s
kube-system    pod/coredns-787d4945fb-h9wh2               1/1     Running   0          2m27s
kube-system    pod/etcd-controlplane                      1/1     Running   0          2m36s
kube-system    pod/kube-apiserver-controlplane            1/1     Running   0          2m36s
kube-system    pod/kube-controller-manager-controlplane   1/1     Running   0          2m40s
kube-system    pod/kube-proxy-rqxts                       1/1     Running   0          2m28s
kube-system    pod/kube-scheduler-controlplane            1/1     Running   0          2m36s

NAMESPACE     NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
app-space     service/default-http-backend   ClusterIP   10.111.33.3      <none>        80/TCP                   90s
app-space     service/video-service          ClusterIP   10.104.172.197   <none>        8080/TCP                 91s
app-space     service/wear-service           ClusterIP   10.109.105.3     <none>        8080/TCP                 91s
default       service/kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP                  2m41s
kube-system   service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   2m38s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   2m34s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   2m38s

NAMESPACE     NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
app-space     deployment.apps/default-backend   1/1     1            1           90s
app-space     deployment.apps/webapp-video      1/1     1            1           91s
app-space     deployment.apps/webapp-wear       1/1     1            1           91s
kube-system   deployment.apps/coredns           2/2     2            2           2m38s

NAMESPACE     NAME                                       DESIRED   CURRENT   READY   AGE
app-space     replicaset.apps/default-backend-b46b9989   1         1         1       90s
app-space     replicaset.apps/webapp-video-57cddcc65     1         1         1       91s
app-space     replicaset.apps/webapp-wear-658fc8dbb4     1         1         1       91s
kube-system   replicaset.apps/coredns-787d4945fb         2         2         2       2m27s

controlplane ~ ➜  
```

Let us now deploy an Ingress Controller. First, create a namespace called ingress-nginx.

We will isolate all ingress related objects into its own namespace.


```bash
controlplane ~ ➜  kubectl create namespace ingress-nginx
namespace/ingress-nginx created

controlplane ~ ➜  
```

The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object with name ingress-nginx-controller in the ingress-nginx namespace.

Use the spec given below. No data needs to be configured in the ConfigMap.

- Name: ingress-nginx-controller

```bash
controlplane ~ ➜  kubectl create configmap ingress-nginx-controller -n ingress-nginx
configmap/ingress-nginx-controller created

controlplane ~ ➜  
```

Confirm that the configmap is created

```bash
controlplane ~ ➜  kubectl get configmap -n ingress-nginxNAME                       DATA   AGE
ingress-nginx-controller   0      34s
kube-root-ca.crt           1      4m32s

controlplane ~ ➜  
```

The NGINX Ingress Controller requires two ServiceAccounts. Create both ServiceAccount with name ingress-nginx and ingress-nginx-admission in the ingress-nginx namespace.

Use the spec provided below.
- Name: ingress-serviceaccount
- Name: ingress-nginx-admission

Creating the first service account

```bash
controlplane ~ ➜  kubectl create serviceaccount ingress-nginx -n ingress-nginx
serviceaccount/ingress-nginx created

controlplane ~ ➜  
```

Creating the second service account

```bash
controlplane ~ ➜  kubectl create serviceaccount ingress-nginx-admission -n ingress-nginx
serviceaccount/ingress-nginx-admission created

controlplane ~ ➜  
```

We have created the Roles and RoleBindings for the ServiceAccount. Check it out!!

```bash
controlplane ~ ➜  kubectl get role -n ingress-nginx
NAME                      CREATED AT
ingress-nginx             2023-02-06T11:45:46Z
ingress-nginx-admission   2023-02-06T11:45:46Z

controlplane ~ ➜  
```

Describe *ingress-nginx* role
```bash
controlplane ~ ➜  kubectl describe role ingress-nginx -n ingress-nginx
Name:         ingress-nginx
Labels:       app.kubernetes.io/component=controller
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  <none>
PolicyRule:
  Resources                           Non-Resource URLs  Resource Names               Verbs
  ---------                           -----------------  --------------               -----
  events                              []                 []                           [create patch]
  configmaps                          []                 []                           [get list watch create]
  endpoints                           []                 []                           [get list watch]
  pods                                []                 []                           [get list watch]
  secrets                             []                 []                           [get list watch]
  services                            []                 []                           [get list watch]
  ingressclasses.networking.k8s.io    []                 []                           [get list watch]
  ingresses.networking.k8s.io         []                 []                           [get list watch]
  configmaps                          []                 [ingress-controller-leader]  [get update]
  namespaces                          []                 []                           [get]
  ingresses.networking.k8s.io/status  []                 []                           [update]

controlplane ~ ➜ 
```

Describe *ingress-nginx-admission* role

```bash
controlplane ~ ➜  kubectl describe role ingress-nginx-admission -n ingress-nginx
Name:         ingress-nginx-admission
Labels:       app.kubernetes.io/component=admission-webhook
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
              helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  secrets    []                 []              [get create]

controlplane ~ ➜ 
```

View created rolebindings

```bash
controlplane ~ ➜  kubectl get rolebinding -n ingress-nginxNAME                      ROLE                           AGE
ingress-nginx             Role/ingress-nginx             4m44s
ingress-nginx-admission   Role/ingress-nginx-admission   4m44s

controlplane ~ ➜  
```

Describe *ingress-nginx* rolebinding

```bash
controlplane ~ ➜  kubectl describe rolebinding ingress-nginx -n ingress-nginx
Name:         ingress-nginx
Labels:       app.kubernetes.io/component=controller
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  <none>
Role:
  Kind:  Role
  Name:  ingress-nginx
Subjects:
  Kind            Name           Namespace
  ----            ----           ---------
  ServiceAccount  ingress-nginx  ingress-nginx

controlplane ~ ➜ 
```

Describe *ingress-nginx-admission* rolebinding

```bash
controlplane ~ ➜  kubectl describe rolebinding ingress-nginx-admission -n ingress-nginx
Name:         ingress-nginx-admission
Labels:       app.kubernetes.io/component=admission-webhook
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
              helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
Role:
  Kind:  Role
  Name:  ingress-nginx-admission
Subjects:
  Kind            Name                     Namespace
  ----            ----                     ---------
  ServiceAccount  ingress-nginx-admission  ingress-nginx

controlplane ~ ➜  
```

Describe the *ingress-nginx* clusterrole

```bash
controlplane ~ ➜  kubectl describe clusterrole ingress-nginx
Name:         ingress-nginx
Labels:       app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  <none>
PolicyRule:
  Resources                           Non-Resource URLs  Resource Names  Verbs
  ---------                           -----------------  --------------  -----
  events                              []                 []              [create patch]
  services                            []                 []              [get list watch]
  ingressclasses.networking.k8s.io    []                 []              [get list watch]
  ingresses.networking.k8s.io         []                 []              [get list watch]
  nodes                               []                 []              [list watch get]
  configmaps                          []                 []              [list watch]
  endpoints                           []                 []              [list watch]
  namespaces                          []                 []              [list watch]
  pods                                []                 []              [list watch]
  secrets                             []                 []              [list watch]
  ingresses.networking.k8s.io/status  []                 []              [update]

controlplane ~ ➜ 
```


Describe the *ingress-nginx-admission* clusterrole

```bash
controlplane ~ ➜  kubectl describe clusterrole ingress-nginx-admission
Name:         ingress-nginx-admission
Labels:       app.kubernetes.io/component=admission-webhook
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
              helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
PolicyRule:
  Resources                                                     Non-Resource URLs  Resource Names  Verbs
  ---------                                                     -----------------  --------------  -----
  validatingwebhookconfigurations.admissionregistration.k8s.io  []                 []              [get update]

controlplane ~ ➜  
```
Describe the *ingress-nginx* clusterrolebinding

```bash
controlplane ~ ➜  kubectl describe clusterrolebinding ingress-nginx
Name:         ingress-nginx
Labels:       app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  ingress-nginx
Subjects:
  Kind            Name           Namespace
  ----            ----           ---------
  ServiceAccount  ingress-nginx  ingress-nginx

controlplane ~ ➜  
```


Describe the *ingress-nginx-admission* clusterrolebinding

```bash
controlplane ~ ➜  kubectl describe clusterrolebinding ingress-nginx-admission
Name:         ingress-nginx-admission
Labels:       app.kubernetes.io/component=admission-webhook
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.1.2
              helm.sh/chart=ingress-nginx-4.0.18
Annotations:  helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
              helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
Role:
  Kind:  ClusterRole
  Name:  ingress-nginx-admission
Subjects:
  Kind            Name                     Namespace
  ----            ----                     ---------
  ServiceAccount  ingress-nginx-admission  ingress-nginx

controlplane ~ ➜ 
```

Let us now deploy the Ingress Controller. Create the Kubernetes objects using the given file.

The Deployment and it's service configuration is given at /root/ingress-controller.yaml. There are several issues with it. Try to fix them.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
	replicas: 1
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --watch-ingress-without-class=true
        - --default-backend-service=app-space/default-http-backend
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
          	fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - name: http
            containerPort: 80
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission


---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodeport: 30080
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort
  ```

  Corrected manifest

  ```yaml
  apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --watch-ingress-without-class=true
        - --default-backend-service=app-space/default-http-backend
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: k8s.gcr.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission

---

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort
  ```


Apply the deployment manifest

```bash
controlplane ~ ➜  kubectl apply -f ingress.yaml 
deployment.apps/ingress-nginx-controller created
service/ingress-nginx-controller created

controlplane ~ ➜  
```

Create the ingress resource to make the applications available at /wear and /watch on the Ingress service.
Also, make use of rewrite-target annotation field: -nginx.ingress.kubernetes.io/rewrite-target: /

Ingress resource comes under the namespace scoped, so don't forget to create the ingress in the app-space namespace.


- Path: /wear
- Path: /watch
- Configure correct backend service for /wear
- Configure correct backend service for /watch
- Configure correct backend port for /wear service
- Configure correct backend port for /watch service


```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
           name: wear-service
           port: 
            number: 8080
      - path: /watch
        pathType: Prefix
        backend:
          service:
           name: video-service
           port:
            number: 8080
```

Apply the manifest file

```bash
controlplane ~ ➜  kubectl apply -f ingress-resource.yaml 
ingress.networking.k8s.io/ingress-wear-watch created

controlplane ~ ➜ 
``` 

***The End***