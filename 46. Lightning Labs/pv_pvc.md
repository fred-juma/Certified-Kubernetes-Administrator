#### Task

A new deployment called alpha-mysql has been deployed in the alpha namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume alpha-pv to be mounted at /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 to make use of an empty root password.

Important: Do not alter the persistent volume.

#### Solution

View created resources


```bash
controlplane ~ ➜  kubectl get all -n alpha
NAME                               READY   STATUS    RESTARTS   AGE
pod/alpha-mysql-6d945ffc78-hnpp9   0/1     Pending   0          18m

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/alpha-mysql   0/1     1            0           18m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/alpha-mysql-6d945ffc78   1         1         0       18m

controlplane ~ ➜  
```

Describe the pending pod to view events. It returns the error *mysql-alpha-pvc* not found.

```bash
controlplane ~ ➜  kubectl describe pod alpha-mysql-6d945ffc78-hnpp9 -n alpha

Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  4m19s (x5 over 19m)  default-scheduler  0/2 nodes are available: 2 persistentvolumeclaim "mysql-alpha-pvc" not found. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.

```

List created PVs

```bash

controlplane ~ ➜  kubectl get persistentvolume
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
alpha-pv   1Gi        RWO            Retain           Available           slow                    23m

controlplane ~ ➜  
```

List created PVCs. No PVC is found

```bash
controlplane ~ ➜  kubectl get persistentvolumeclaim
No resources found in default namespace.

controlplane ~ ➜  
```

Create the PVC with same capacity *1Gi* as the PV.

```bash
controlplane ~ ➜  kubectl vi alpha-pvc.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpha-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

Apply the pvc manifest

```bash
controlplane ~ ➜  kubectl apply -f alpha-pvc.yaml 
persistentvolumeclaim/alpha-pvc created

controlplane ~ ➜  

```

Update the deployment definition and specify the 

volume mounts and volume for the PV and PVC



```bash
...
...
spec:
      containers:
      - env:
        - name: MYSQL_ALLOW_EMPTY_PASSWORD
          value: "1"
        image: mysql:5.6
        imagePullPolicy: Always
        name: mysql
        ports:
        - containerPort: 3306
          protocol: TCP

 		volumeMounts:
        - mountPath: /var/lib/mysql
          name: mysql-data
        - mountPath: /var/lib/mysql
          name: alpha-pv
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
volumes:
- name: mysql-data
  persistentVolumeClaim:
    claimName: mysql-alpha-pvc
- name: alpha-pv
  persistentVolumeClaim:
    claimName: alpha-pvc
```

```bash
controlplane ~ ➜  kubectl edit deployment alpha-mysql -n alpha
deployment.apps/alpha-mysql edited

controlplane ~ ➜  
```

View all resources and confirm that the deployment, replicaset and pod is ready

```bash

controlplane ~ ➜  kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/gold-nginx-69d8d76f55-5rmhx     1/1     Running   0          53m
pod/nginx-deploy-6555d89bcd-m99n7   1/1     Running   0          37m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   111m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gold-nginx     1/1     1            1           53m
deployment.apps/nginx-deploy   1/1     1            1           41m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/gold-nginx-69d8d76f55     1         1         1       53m
replicaset.apps/nginx-deploy-648567495    0         0         0       41m
replicaset.apps/nginx-deploy-6555d89bcd   1         1         1       37m
replicaset.apps/nginx-deploy-d95bf7756    0         0         0       39m

controlplane ~ ➜  
```

***The End***