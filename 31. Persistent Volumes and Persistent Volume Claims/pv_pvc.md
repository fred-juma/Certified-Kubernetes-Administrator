#### Docker storage

Docker host file system - Where docker stores files by default including files related to images and containers running on the docker host.

/var/lib/docker
	aufs
	containers
	image
	volumens


Docker layered architecture
- Each line of instructions in Dockerfile, creates a new layer in the docker image with just the changes in the previous layer

FROM ubuntu ..........................................Layer 1 - Base ubuntu layer (Bottom)

RUN apt-get update && apt-get -y install python.......Layer 2 - changes in apt packages

RUN pip install flask-mysql...........................Layer 3 - changes in pip packages

COPY . /opt/source-code...............................Layer 4 - source code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run..Layer 5 - update entrypoint (Top)

Then you build the image:

docker build Dockerfile -t my-app

- Once image is build, no more changes can be made, the image is now read only

- Then you create container:

docker run my-app

- This creates the Layer 6 - Container layer. A writeable  (read, write) layer used to store data created by the container including log files,  temp files
- The life of this layer is as long as it is powered on.
- When the container is destroyed, the layer and all the changes in it are destroyed
- The image layer though is shared by all containers created using this image

COPY-ON-WRITE - if the source code is edited in the conatiner layer, a copy of the file will be created in the container, while the initial one on the image layer 4 will not be changed until the image is rebuild.
- However, if the container is destroyed, all the source code / app changes will be destroyed.
- If you would like to preserve the data created by the container, use persistent volume.

#### Volumes

Volume Mount:

docker volume create data_volume

- This creates volume under /var/lib/docker/volumes/data_volume
- Mount the volume during container creation

docker run -v data_volume:/var/lib/mysql mysql

- This creates a new container and mounts the data_volume inside the counter at /var/lib/mysql
- All data written on this volume is stored on the host directory data_volume
- That way even if the container is destroyed, the data is persisted

Bind Mount:

Create a container and mount the container volume to an existing directory which is not within the docker file system on the docker host

docker run -v /data/mysql:/var/lib/mysql mysql (deprecated) 

docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql

- Docker uses storage drivers to enable the layered architecture. The storage drivers include:

AUFS
ZFS
BTRFS
Device Mapper
Overlay
Overlay2

- Choice of storage driver is dependent on the OS used.
- The storage drivers help manage storage on images and containers

#### Volume Drivers
- Volumes are handled by volume driver plugins
- The default volume driver plugin is *Local*
- The Local volume driver plugin help create a volume on the docker host and store its data in /var/lib/docker/volumes/
- There are other  volume driver plugin tha help create volumes on 3rd party storage solutions  

To provision a volume on AWS EBS and attach to a docker container:

docker run -it \
--name mysql \
--volume-driver rexray/ebs \
--mount src=ebs-vol,target=/var/lib/mysql mysql

- When the container exits, the data is saved in the cloud

#### Container Storage Interface

- Container Runtime Interface (CRI) is a standard that defines how k8s communicate with container runtimes e.g. docker, cri-o, rkt etc.
- Container Networking Interface (CNI) - Extend support for different networking solutions
- Container Storage Interface (CSI) - A universal standard to support multiple storage solutions


#### Kubernetes Volumes
- Like docker containers, data in pods are transient, deleted when the pod is deleted
- For persistence volume is used.
- Create avolume on the pod node and mount it on the pod

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: random-number-geberator
spec: 
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data

```

- This is not recommended in a multi-node cluster.

#### Persistent Volumes
- This is a cluster wide pool of volumes configured by admin to be used by users deploying apps on the cluster
- The users cannow select storage from the cluster using persstent volume claims
- Access Modes include:

ReadOnlyMany
ReadWriteOnce
ReadWriteMany

persistent volume definition

```yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-vol1
spec: 
  accessModes: 
    - ReadWriteOnce
  capacity: 
    storage: 1Gi
  hostPath:
    path: /tmd/data
```

Persistent Volume Claims
- Request properties include:
sufficient capacity
Access modes
Volume modes
Storage class

pvc definition

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: myclaim
spec: 
  accessModes: 
    - ReadWriteOnce
  resources:
    requests: 
      storage: 500Mi
  
```

persistentVolumeReclaimPolicy:
- Retain
- Delete
- Recycle

We have deployed a POD. Inspect the POD and wait for it to start running.

```bash
controlplane ~ ➜  kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          21s

controlplane ~ ➜  
```

The application stores logs at location /log/app.log. View the logs. You can exec in to the container and open the file:
kubectl exec webapp -- cat /log/app.log

```bash

controlplane ~ ➜  kubectl exec webapp -- cat /log/app.log
[2023-01-25 11:02:04,622] INFO in event-simulator: USER4 is viewing page3
[2023-01-25 11:02:05,623] INFO in event-simulator: USER1 logged out
[2023-01-25 11:02:06,625] INFO in event-simulator: USER3 is viewing page2
```

Configure a volume to store these logs at /var/log/webapp on the host. 
Use the spec provided below.

- Name: webapp
- Image Name: kodekloud/event-simulator
- Volume HostPath: /var/log/webapp
- Volume Mount: /log

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    
    volumeMounts:
    -  mountPath: /log
       name: log-volume
 
  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
      type: Directory
```

Apply to create the resources

```bash
controlplane ~ ➜  kubectl apply -f web.yaml 
pod/webapp created

controlplane ~ ➜
```  

Create a Persistent Volume with the given specification.

- Volume Name: pv-log
- Storage: 100Mi
- Access Modes: ReadWriteMany
- Host Path: /pv/log
- Reclaim Policy: Retain


```yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-log
spec: 
  accessModes: 
    - ReadWriteMany
  capacity: 
    storage: 100Mi
  hostPath:
    path: /pv/log
  persistentVolumeReclaimPolicy: Retain
 ```

Apply the manifest

```bash
controlplane ~ ➜  kubectl apply -f pv.yaml 
persistentvolume/pv-log created

controlplane ~ ➜  
```

Let us claim some of that storage for our application. Create a Persistent Volume Claim with the given specification.

- Volume Name: claim-log-1
- Storage Request: 50Mi
- Access Modes: ReadWriteOnce


  
```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  
  resources:
    requests:
      storage: 50Mi
```
Apply the manifest

```bash
controlplane ~ ➜  kubectl apply -f pvc.yaml 
persistentvolumeclaim/claim-log-1 created

controlplane ~ ➜  
````
What is the state of the Persistent Volume Claim?


```bash

controlplane ~ ➜  kubectl get persistentvolumeclaim
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           8s

controlplane ~ ➜  
```


What is the state of the Persistent Volume?

```bash


controlplane ~ ➜  kubectl get persistentvolume
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1                           3m10s

controlplane ~ ➜  
```

Update the webapp pod to use the persistent volume claim as its storage. 
Replace hostPath configured earlier with the newly created PersistentVolumeClaim.

- Name: webapp
- Image Name: kodekloud/event-simulator
- Volume: PersistentVolumeClaim=claim-log-1
- Volume Mount: /log


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    
    volumeMounts:
    -  mountPath: /log
       name: log-volume
 
  volumes:
  - name: log-volume
    persistentVolumeClaim:
    	claimName: claim-log-1

```

What is the Reclaim Policy set on the Persistent Volume pv-log? **Retain** Therefore if the PVC was destroyed, the PV is not deleted but will not be available for binding


If the pod and the pvc are deleted, the status of the PV will be **Released**

```bash
controlplane ~ ➜  kubectl get persistentvolume
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Released   default/claim-log-1                           10m

controlplane ~ ➜ 
```