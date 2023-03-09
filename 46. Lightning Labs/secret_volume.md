#### Task

Create a pod called secret-1401 in the admin1401 namespace using the busybox image. The container within the pod should be called secret-admin and should sleep for 4800 seconds.

The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.

#### Solution

View the secret

```bash
controlplane ~ ➜  kubectl get secret -n admin1401
NAME             TYPE     DATA   AGE
dotfile-secret   Opaque   1      34m

controlplane ~ ➜  
```

Create the pod

```bash
controlplane ~ ➜  kubectl run secret-1401 -n admin1401 --image=busybox -o yaml --dry-run=client > secret-1401.yaml
```

Edit the pod manifest with the secret volume and volumeMount

```bash
controlplane ~ ➜  vi secret-1401.yaml 

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401
spec:
  volumes:
  - name: secret-volume
    secret: 
      secretName: dotfile-secret
  containers:
  - image: busybox
    name: secret-admin
    command: ["sleep"]
    args: ["4800"]
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: /etc/secret-volume
```

Apply the pod definition

```bash
controlplane ~ ➜  kubectl apply -f secret-1401.yaml 
pod/secret-1401 created

controlplane ~ ➜  
```

Confirm the pod is created and is running

```bash
controlplane ~ ➜  kubectl get pod -n admin1401
NAME          READY   STATUS    RESTARTS   AGE
secret-1401   1/1     Running   0          14s

controlplane ~ ➜ 
``` 