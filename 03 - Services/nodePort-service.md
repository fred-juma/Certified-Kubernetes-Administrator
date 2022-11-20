#### Instructions

Create a new service to access the web application using the service-definition-1.yaml file.


Name: webapp-service
Type: NodePort
targetPort: 8080
port: 8080
nodePort: 30080
selector:
  name: simple-webapp

#### Solution

#### Using vi editor, create the service definition template yaml file 

```bash

controlplane ~ ➜  vi service-definition-1.yaml

controlplane ~ ➜  

```

#### Within the yaml file, enter the details as below:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30080
  selector:
    name: simple-webapp
~                           
```


#### use *kubectl* utility to create the service from the tenplate yaml file

```bash
controlplane ~ ➜  kubectl apply -f service-definition-1.yaml 
service/webapp-service created

controlplane ~ ➜  
```

#### Confirm that the service was created

```bash
controlplane ~ ➜  kubectl get services
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.43.0.1    <none>        443/TCP          17m
webapp-service   NodePort    10.43.21.3   <none>        8080:30080/TCP   22s

controlplane ~ ➜  
```

***The End***



