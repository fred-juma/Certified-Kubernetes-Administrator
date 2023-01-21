#### kubeConfig

Located at: $HOME/.kube/config

```bash

controlplane ~ ➜  ls -la /root/.kube/config 
-rw------- 1 root root 5636 Jan 19 01:40 /root/.kube/config

controlplane ~ ➜  
```

The file is left as is, no object is created. 
kubectl command reads the file and the required values are used.


To view current config file being used

```bash
controlplane ~ ➜  kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

controlplane ~ ➜  
```


To view a different kubeconfig file:

```bash
controlplane ~ ➜  kubectl config view --kubeconfig=my-kube-config
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}

controlplane ~ ➜ 
```

To change/update the current context



use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.

```bash
controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config use-context research
Switched to context "research".

controlplane ~ ➜  
```

Once the right context is identified, use the kubectl config use-context command.

```bash
controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config current-context
research

controlplane ~ ➜  
```

We don't want to have to specify the kubeconfig file option on each command. Make the my-kube-config file the default kubeconfig.


To change the other parameters of the kubeconfig file, see the options using the command:

Replace the contents in the default kubeconfig file with the content from my-kube-config file



kubectl config -h 

Alternative to the "certificate-authority" field in the kubeconfig file, you can use "certificate-authority-data" and specifify the certificate data in base64 encoded format

cat ca.crt | base64

To decode the cert:

echo "<encoded-data>" | base64 --decode


With the current-context set to research, we are trying to access the cluster. However something seems to be wrong. Identify and fix the issue.

Try running the kubectl get pods command and look for the error. All users certificates are stored at /etc/kubernetes/pki/users.

Check the certificate locations and names

```bash
controlplane ~ ➜  ls -l /etc/kubernetes/pki/users/aws-user/
total 12
-rw-r--r-- 1 root root 1025 Jan 21 01:48 aws-user.crt
-rw-r--r-- 1 root root  924 Jan 21 01:48 aws-user.csr
-rw------- 1 root root 1679 Jan 21 01:48 aws-user.key

controlplane ~ ➜  

controlplane ~ ➜  ls -l /etc/kubernetes/pki/users/dev-user/
total 12
-rw-r--r-- 1 root root 1025 Jan 21 01:48 dev-user.crt
-rw-r--r-- 1 root root  924 Jan 21 01:48 dev-user.csr
-rw------- 1 root root 1675 Jan 21 01:48 dev-user.key

controlplane ~ ➜  


controlplane ~ ➜  ls -l /etc/kubernetes/pki/users/test-user/
total 12
-rw-r--r-- 1 root root 1025 Jan 21 01:48 test-user.crt
-rw-r--r-- 1 root root  924 Jan 21 01:48 test-user.csr
-rw------- 1 root root 1679 Jan 21 01:48 test-user.key

controlplane ~ ➜  

```

Check the corresponding configurations on the kubeconfig. You can see that the dev-user.crt client certificate is wrongly named. Rectify it and use the correct certificate name.

```bash
controlplane ~ ➜  cat /root/.kube/config | grep client
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key

controlplane ~ ➜  
```
***The End***

