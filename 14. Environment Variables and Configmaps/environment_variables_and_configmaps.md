#### Configuring Environment Variables in Applications

Using docker run

```bash

docker run -e APP_COLOR=pink simple-webapp-color
```

In pod specification yaml file

```yaml

apiVersion: v1
kind: Pod 
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    command: ["sleep"]
    args: ["2000"]
    env:
      - name: APP_COLOR
        value: pink
      - name: APP_MODE
        value: prod
```
 Configuring ConfigMaps in Applications

 - create configMaps
 - Then inject them into the pods

Step 1

2 ways of creating config map:

1) imperative way of creating configMaps:

```bash
 kubectl create configmap \
 app-config --from-literal=APP_COLOR=blue \
            --from-literal=APP_MODE=prod
```

yet another way is:

```bash
 kubectl create configmap \
 app-config --from-file=app_config.properties
```

2) Declarative way

Create a definition file:

config-map.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR=blue
  APP_MODE=prod
```

Then apply the definition file

```bash
kubectl create -f config-map.yaml
```



Step 2:

Then inject config maps in a pod

```yaml

apiVersion: v1
kind: Pod 
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
          name: app-config
```

Listing config maps

```bash

 controlplane ~ ➜  kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      10m
db-config          3      12s
```

Describing configmaps

```bash
controlplane ~ ➜  kubectl describe configmaps db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_NAME:
----
SQL01
DB_PORT:
----
3306
DB_HOST:
----
SQL01.example.com

BinaryData
====

Events:  <none>

controlplane ~ ➜  
```


Create a new ConfigMap for the webapp-color POD. Use the spec given below.


ConfigName Name: webapp-config-map

Data: APP_COLOR=darkblue

```bash
controlplane ~ ➜  kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
configmap/webapp-config-map created
```

Update the environment variable on the POD to use the newly created ConfigMap

Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.


Pod Name: webapp-color

EnvFrom: webapp-config-map

```bash
controlplane ~ ➜  kubectl edit pods webapp-color



......
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: webapp-config-map
.......        
```

Delete the pod

```bash
controlplane ~ ➜ kubectl delete pods webapp-color
pod "webapp-color" deleted
```

Recreate the pod

```bash
controlplane ~ ➜  kubectl apply -f /tmp/kubectl-edit-1420084020.yaml
pod/webapp-color created

controlplane ~ ➜  
```