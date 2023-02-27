#### Troubleshooting Test 1:

A simple 2 tier application is deployed in the alpha namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

Application Architecture              |  
:-------------------------:|
![ApplicationArchitecture](blob/main/images/application_failure.JPG)

View all resources in the namespace

```bash
controlplane ~ ➜  kubectl get all -n alpha
NAME                                READY   STATUS    RESTARTS   AGE
pod/webapp-mysql-7cc9dcdffd-jnpkv   1/1     Running   0          4m53s
pod/mysql                           1/1     Running   0          4m53s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.43.214.229   <none>        3306/TCP         4m53s
service/web-service   NodePort    10.43.37.139    <none>        8080:30081/TCP   4m53s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           4m53s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-7cc9dcdffd   1         1         1       4m53s

controlplane ~ ➜ 
```

Check the pod logs

```bash
controlplane ~ ➜ kubectl logs webapp-mysql-7cc9dcdffd-jnpkv -n alpha
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
10.42.0.1 - - [16/Feb/2023 06:28:33] "GET / HTTP/1.1" 200 -
10.42.0.1 - - [16/Feb/2023 06:28:34] "GET /static/img/failed.png HTTP/1.1" 200 -

controlplane ~ ➜  
```


Check the web pod label and ensure it matches with the web-service label

web pod label is *name=webapp-mysql*

```bash
controlplane ~ ➜ kubectl describe pod webapp-mysql-7cc9dcdffd-jnpkv -n alpha
Name:             webapp-mysql-7cc9dcdffd-jnpkv
Namespace:        alpha
Priority:         0
Service Account:  default
Node:             controlplane/172.25.0.9
Start Time:       Thu, 16 Feb 2023 06:22:57 +0000
Labels:           name=webapp-mysql
                  pod-template-hash=7cc9dcdffd
```

web service label is also *name=webapp-mysql* 

```bash
controlplane ~ ➜  kubectl describe service web-service -n alpha
Name:                     web-service
Namespace:                alpha
Labels:                   <none>
Annotations:              <none>
Selector:                 name=webapp-mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.37.139
IPs:                      10.43.37.139
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30081/TCP
Endpoints:                10.42.0.10:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

controlplane ~ ➜  
```

Next check the mysql pod logs

```bash
2023-02-16 06:23:19 1 [Note] Server hostname (bind-address): '*'; port: 3306
2023-02-16 06:23:19 1 [Note] IPv6 is available.
2023-02-16 06:23:19 1 [Note]   - '::' resolves to '::';
2023-02-16 06:23:19 1 [Note] Server socket created on IP: '::'.
2023-02-16 06:23:19 1 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2023-02-16 06:23:19 1 [Warning] 'proxies_priv' entry '@ root@mysql' ignored in --skip-name-resolve mode.
2023-02-16 06:23:19 1 [Note] Event Scheduler: Loaded 0 events
2023-02-16 06:23:19 1 [Note] mysqld: ready for connections.
Version: '5.6.51'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

Check the label on mysql pod which is *name=mysql*

```bash
controlplane ~ ➜  kubectl describe pod mysql -n alpha
Name:             mysql
Namespace:        alpha
Priority:         0
Service Account:  default
Node:             controlplane/172.25.0.9
Start Time:       Thu, 16 Feb 2023 06:22:57 +0000
Labels:           name=mysql
Annotations:      <none>
Status:           Running
IP:               10.42.0.9
IPs:
  IP:  10.42.0.9
 ```

 Check the mysql service log, the label is *name=mysql* same as the pod

 ```bash
 controlplane ~ ➜  kubectl describe service mysql -n alpha
Name:              mysql
Namespace:         alpha
Labels:            <none>
Annotations:       <none>
Selector:          name=mysql
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.214.229
IPs:               10.43.214.229
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.42.0.9:3306
Session Affinity:  None
Events:            <none>

controlplane ~ ➜  
 ```

Going back to the architectural diagram, we note that the database service is labelled as *mysql-service* whereas the corresponding object is named mysql.

Let us amend the name to *mysql-service*

```bash
controlplane ~ ➜  kubectl edit service mysql -n alpha

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-02-16T06:22:57Z"
  name: mysql-service
  namespace: alpha
  resourceVersion: "754"
  uid: 36784a99-894d-401f-8fd1-695ca70e8c8f
spec:
  clusterIP: 10.43.214.229
  clusterIPs:
  - 10.43.214.229
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Delete the service
```bash

controlplane ~ ✖ kubectl delete service mysql -n alpha
service "mysql" deleted
```

Recreate the service with the new configs

```bash
controlplane ~ ➜  kubectl apply -f /tmp/kubectl-edit-402886320.yaml
service/mysql-service created

controlplane ~ ➜
```  

Confirm the change is made

```bash
controlplane ~ ➜  kubectl get service -n alpha
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web-service     NodePort    10.43.37.139    <none>        8080:30081/TCP   47m
mysql-service   ClusterIP   10.43.214.229   <none>        3306/TCP         90s

controlplane ~ ➜  
```


#### Troubleshooting Test 2

The same 2 tier application is deployed in the beta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


```bash
controlplane ~ ➜  kubectl get all -n beta
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          26s
pod/webapp-mysql-7cc9dcdffd-74srf   1/1     Running   0          26s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql-service   ClusterIP   10.43.193.230   <none>        3306/TCP         26s
service/web-service     NodePort    10.43.92.87     <none>        8080:30081/TCP   26s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           26s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-7cc9dcdffd   1         1         1       26s

controlplane ~ ➜  
```

If you inspect the *mysql-service* in the beta namespace, you will notice that the targetPort used to create this service *8080* is incorrect.
Compare this to the Architecture diagram and change it to *3306*. Update the mysql-service as per the below YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-02-16T06:22:57Z"
  name: mysql-service
  namespace: beta
  resourceVersion: "754"
  uid: 36784a99-894d-401f-8fd1-695ca70e8c8f
spec:
  clusterIP: 10.43.214.229
  clusterIPs:
  - 10.43.214.229
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

#### Troubleshooting Test 3: 

The same 2 tier application is deployed in the gamma namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed or unresponsive. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

List of all resources

```bash
controlplane ~ ➜  kubectl get all -n gamma
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          106s
pod/webapp-mysql-7cc9dcdffd-h7jjc   1/1     Running   0          106s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql-service   ClusterIP   10.43.97.75     <none>        3306/TCP         106s
service/web-service     NodePort    10.43.215.203   <none>        8080:30081/TCP   106s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           106s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-7cc9dcdffd   1         1         1       106s

controlplane ~ ➜  
```

The selctor name configured in mysql service *sql00001* is wrong. edit the pod with correct slector name which is equivalent to the name configured in the mysql *name=mysql*


#### Troubleshooting Test 4:

The same 2 tier application is deployed in the delta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

Application Architecture              |  
:-------------------------:|
![ApplicationArchitecture](blob/main/images/application_failure4.JPG)

View all resources in the namespace

```bash
controlplane ~ ➜  kubectl get all -n delta
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          108s
pod/webapp-mysql-78c8f7998b-8bfgr   1/1     Running   0          108s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql-service   ClusterIP   10.43.103.70    <none>        3306/TCP         108s
service/web-service     NodePort    10.43.215.227   <none>        8080:30081/TCP   108s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           108s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-78c8f7998b   1         1         1       108s

controlplane ~ ➜  
```

There is a problem in the value of environment variables configuration for key DB_User. The give user is *root* whereas the in the pod definition, the value for the DB_User key is *root*. Edit the deployment definition and correct the name



#### Troubleshooting Test 5:

The same 2 tier application is deployed in the epsilon namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


View all resources in the epsilon namespace

```bash
controlplane ~ ➜  kubectl get all -n epsilon
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          37s
pod/webapp-mysql-78c8f7998b-9z6xn   1/1     Running   0          37s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql-service   ClusterIP   10.43.218.161   <none>        3306/TCP         37s
service/web-service     NodePort    10.43.10.215    <none>        8080:30081/TCP   37s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           37s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-78c8f7998b   1         1         1       37s

controlplane ~ ➜  
```


Correct the deployment environment variable key DB_User with correct value *root*

Correct the mysql pod environment variable key MYSQL_ROOT_PASSWORD with correct value *paswrd*


#### Troubleshooting Test 6:

The same 2 tier application is deployed in the zeta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

View all resources in the zeta namespace

```bash
controlplane ~ ✖ kubectl get all -n zeta
NAME                                READY   STATUS    RESTARTS   AGE
pod/webapp-mysql-78c8f7998b-kd787   1/1     Running   0          3m54s
pod/mysql                           1/1     Running   0          3m54s

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/mysql-service   ClusterIP   10.43.41.68    <none>        3306/TCP         3m54s
service/web-service     NodePort    10.43.190.37   <none>        8080:30088/TCP   3m53s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           3m54s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-78c8f7998b   1         1         1       3m54s

controlplane ~ ➜  
```

orrect the mysql pod environment variable key MYSQL_ROOT_PASSWORD with correct value *paswrd*
Correct the deployment environment variable key DB_User with correct value *root*
Correct the websivce nodePort with correct value *30081*


***The End***


