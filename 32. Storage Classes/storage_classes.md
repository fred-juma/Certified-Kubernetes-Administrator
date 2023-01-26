#### Static Provisioning

This entails creating the persistent disk, then creating persistent volume from the persistent disk

Example below we create a persistent disk in GCP then we create a PV from the disk

```bash

gcloud beta compute disks create \
--size 1GB \
--region us-east1 \
pd-disk
```
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
    storage: 500Mi

  gcePersistentDisk:
    pdName: pd-disk
    fsType: ext4
```
#### Dynamic Provisioning
- This make use of **storage classes**
- With storage classes you can define a provisioner such as google storage that can automatically provision strorage on GC and attach that to pods when a pod is created.

*storage class definition*

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd

parameters:
  type: pd-standard
  replication-type: none
```

- With the storage class, we no longer need to create PV manually
- PV and any associated storage is going to be created automatically when the storage class is created
- For PVC to use the storage class created, we specify the storageClassName:

*pvc definition*

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
storageClassName: google-storage
  
  resources:
    requests:
      storage: 500Mi
```
- Now next time a pvc is created, the storage class associated with it uses the defined provisioner to provision a new disk with the required size on GCP and then creates a persistent volume and binds the pvc to that volume

How many StorageClasses exist in the cluster right now?

```bash
controlplane ~ ➜  kubectl get StorageClasses
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  12m
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  5s
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  5s

controlplane ~ ➜  

```

View PV

```bash
controlplane ~ ➜  kubectl get PersistentVolume
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
local-pv   500Mi      RWO            Retain           Available           local-storage            118s
```

Create a new PersistentVolumeClaim by the name of local-pvc that should bind to the volume local-pv. Inspect the pv local-pv for the specs.


- PVC: local-pvc
- Correct Access Mode?
- Correct StorageClass Used?
- PVC requests volume size = 500Mi?

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```



```bash
controlplane ~ ➜  kubectl apply -f local-pvc.yaml 
persistentvolumeclaim/claim-log-1 created

controlplane ~ ➜  
```

view created pvc

```bash
controlplane ~ ➜  kubectl get PersistentVolumeClaim
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
claim-log-1   Pending                                      local-storage   14m
local-pvc     Pending                                      local-storage   19s
```

Describe the pvc

```bash
controlplane ~ ➜  kubectl describe PersistentVolumeClaim local-pvc
Name:          local-pvc
Namespace:     default
StorageClass:  local-storage
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
  Normal  WaitForFirstConsumer  9s (x5 over 64s)  persistentvolume-controller  waiting for first consumer to be created before binding

controlplane ~ ➜  
```

- The Storage Class called local-storage makes use of VolumeBindingMode set to WaitForFirstConsumer. This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.


Create a new pod called nginx with the image nginx:alpine. The Pod should make use of the PVC local-pvc and mount the volume at the path /var/www/html.

The PV local-pv should be in a bound state.

- Pod created with the correct Image?
- Pod uses PVC called local-pvc?
- local-pv bound?
- nginx pod running?
- Volume mounted at the correct path?

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
      - name: local-persistent-storage
        mountPath: /var/www/html
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

view the pod bound to pvc

```bash
controlplane ~ ➜  kubectl get PersistentVolumeClaim
NAME          STATUS    VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
claim-log-1   Pending                                        local-storage   27m
local-pvc     Bound     local-pv   500Mi      RWO            local-storage   13m

controlplane ~ ➜  
```


Create a new Storage Class called delayed-volume-sc that makes use of the below specs:

provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer


```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```