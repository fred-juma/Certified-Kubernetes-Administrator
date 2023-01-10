#### Multi-Container pods

Create a multi-container pod with 2 containers.

Use the spec given below.

If the pod goes into the crashloopbackoff then add the command sleep 1000 in the lemon container.


Name: yellow

Container 1 Name: lemon

Container 1 Image: busybox

Container 2 Name: gold

Container 2 Image: redis


```bash

controlplane ~ ➜  vi yellow.yaml

controlplane ~ ➜  
```

```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command: ["sleep"]
    args: ["1000"]
  - name: gold
    image: redis
```

Apply the pod manifest

```bash

controlplane ~ ➜  kubectl apply -f yellow.yaml 
pod/yellow created

controlplane ~ ➜  
```

We have deployed an application logging stack in the elastic-stack namespace. Inspect it.


Elastic-stack Deployment              |  
:-------------------------:|
![deployment](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/elastic-stack.JPG)


```bash

controlplane ~ ➜  kubectl get all --namespace elastic-stack
NAME                 READY   STATUS    RESTARTS   AGE
pod/app              1/1     Running   0          16m
pod/elastic-search   1/1     Running   0          16m
pod/kibana           1/1     Running   0          16m

NAME                    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
service/elasticsearch   NodePort   10.96.192.45     <none>        9200:30200/TCP,9300:30300/TCP   16m
service/kibana          NodePort   10.105.149.215   <none>        5601:30601/TCP                  16m

controlplane ~ ➜  
```


We will configure a sidecar container for the application to send logs to Elastic Search.

NOTE: It can take a couple of minutes for the Kibana UI to be ready after the Kibana pod is ready.

You can inspect the Kibana logs by running:

kubectl -n elastic-stack logs kibana

```bash
controlplane ~ ➜  kubectl -n elastic-stack logs kibana
{"type":"log","@timestamp":"2023-01-09T12:49:31Z","tags":["status","plugin:kibana@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2023-01-09T12:49:31Z","tags":["status","plugin:elasticsearch@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2023-01-09T12:49:31Z","tags":["status","plugin:xpack_main@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
```
Inspect the app pod and identify the number of containers in it.


It is deployed in the elastic-stack namespace.

```bash

controlplane ~ ➜  kubectl describe pod app --namespace elastic-stack
Name:             app
Namespace:        elastic-stack
Priority:         0
Service Account:  default
Node:             controlplane/10.54.151.3
Start Time:       Mon, 09 Jan 2023 07:47:45 -0500
Labels:           name=app
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  app:
    Container ID:   containerd://6e58ec90fbbaa911fe36f1b7b451ac8c50a9c0c7964d849ef53b3d5a2f8de3f0
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 09 Jan 2023 07:47:52 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /log from log-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2268b (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  log-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log/webapp
    HostPathType:  DirectoryOrCreate
  kube-api-access-2268b:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20m   default-scheduler  Successfully assigned elastic-stack/app to controlplane
  Normal  Pulling    20m   kubelet            Pulling image "kodekloud/event-simulator"
  Normal  Pulled     20m   kubelet            Successfully pulled image "kodekloud/event-simulator" in 4.71283805s (4.712950457s including waiting)
  Normal  Created    20m   kubelet            Created container app
  Normal  Started    20m   kubelet            Started container app

controlplane ~ ➜  
```

The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login.


Inspect the log file inside the pod.

```bash

controlplane ~ ➜  kubectl -n elastic-stack exec -it app -- cat /log/app.log | less

controlplane ~ ➜ 
```

Edit the pod to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.

Only add a new container. Do not modify anything else. Use the spec provided below.




Note: State persistence concepts are discussed in detail later in this course. For now please make use of the below documentation link for updating the concerning pod.


https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/


Name: app

Container Name: sidecar

Container Image: kodekloud/filebeat-configured

Volume Mount: log-volume

Mount Path: /var/log/event-simulator/

Existing Container Name: app

Existing Container Image: kodekloud/event-simulator


Elastic-stack Deployment with Multi-Container              |  
:-------------------------:|
![deployment](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/multi-container-pod.JPG)


```yaml

apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume

  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: DirectoryOrCreate
```

Apply the pod manifest

```bash
controlplane ~ ➜  kubectl apply -f pod.yaml 
pod/pod created

controlplane ~ ➜  
```

***The End***