Configure Applications

Configuring applications comprises of understanding the following concepts:

- Configuring Command and Arguments on applications

- Configuring Environment Variables

- Configuring Secrets

*command* field in pod specification -> overrides to *ENTRYPOINT* field in docker file
*args* field in pod specification -> overrides to *CMD* field in docker file

FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. 

apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["5000"]

controlplane ~ ➜  kubectl apply -f ubuntu-sleeper-2.yaml 
pod/ubuntu-sleeper-2 created

controlplane ~ ➜  

Update pod ubuntu-sleeper-3 to sleep for 2000 seconds

apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["2000"]


controlplane ~ ➜ kubectl replace --force -f ubuntu-sleeper-3.yaml 
pod "ubuntu-sleeper-3" deleted
pod/ubuntu-sleeper-3 replaced

Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green.

controlplane ~ ➜  kubectl run webapp-green --image=kodekloud/webapp-color -- --color green
pod/webapp-green created

controlplane ~ ➜  