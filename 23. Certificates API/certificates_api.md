#### Certificates workflow & API

- K8s admin sets up the cluster, the CA server and the various certificates. The admin also has private/public key pair for authenticating into the cluster

When a new New admin joins the team:

- New admin generate their own private/public key pair
- Generate Certificate Signing (CSR) Request and sends to the admin
- The admin get the CSR to the CA server for signing using the CA server private key hence generating a valid certificate
- The admin sends the certificate to the new admin
- The nwe admin now have valid certificate and key to authenticate to the cluster


The CA server
- Stores the certificate key file.
- The CA server is the master node
- All the certificate related operations are carried out by the **Controller Manager**
- The controller manager has controllers in it called **CSR-APPROVING** and **CSR-SIGNING** etc. that are responsible for the tasks
- The controller manager has 2 options for specifying the CA servers root certiticate and key 

cat /etc/kubernetes/manifests/kube-controller-manager.yaml

    --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    --cluster-signing-key-file=/etc/kubernetes/pki/ca.crt

Automating the Certificate Signing workfolw and the certifcates rotation
- Using builtin kubernetes certificates API
- CSR is sent directly to the CA through an API call
- When admin receives the CSR fom the new admin, they create a CertificateSigningRequest Object
- Requests can the reviewed and approved by the admin using kubectl commands

#### The Process

A new member akshay joined our team. He requires access to our cluster. The Certificate Signing Request is at the /root location.

```bash
controlplane ~ ➜  ls -l /root/
total 8
-rw-r--r-- 1 root root  887 Jan 18 07:41 akshay.csr
-rw------- 1 root root 1679 Jan 18 07:41 akshay.key
```
controlplane ~ ➜  


Create a **CertificateSigningRequest** object with the name akshay with the contents of the **akshay.csr** file

As of kubernetes 1.19, the API to use for CSR is **certificates.k8s.io/v1**

Please note that an additional field called **signerName** should also be added when creating CSR. For client authentication to the API server we will use the built-in signer **kubernetes.io/kube-apiserver-client**.

openssl genrsa -out jane.key 2048

They then generate CSR with the key and sends the CSR to the admin

*openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr*


The admin takes the CSR and generates CertificateSigningRequest Object using a manifest file

The csr is encoded in base64,  generate the base64 encoded format as following: -


// *cat akshar.csr | base64 | tr -d "\n"*



```bash

controlplane ~ ➜  cat akshay.csr | base64 -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTdmM1p5YXFqM255UlF6WEZUK3ZsY01MSllUVWM2a20rSEJCN2RTVWJ0VTZqCm11UmlDcytoLzluOU5UK2VYdG9DT0RwclpNY2szSnk2UXB0NWE5dGZLcWl2ZlVzU2JIa0JWekxTcWZSaVpzYUEKZnZGaGpYN3RsTzJ2TURIWFlqWC9FbEJUb01OcFJNYkl6NlBWR05TQk4vQjFCVWZ5VnlpWldHamUvSWdwSDBlNQprT3UvaUhocVVzQ1ExZktXTjJnaFY5bWttME5PbzBxTTRMeWdPN3pRdzlFallwai9FS1lyVDJlSTVlSVRxSHBVCnRZeHVKSmh2bW9ydXFZc3pSamdlVU9NalBUS0FsdGROeFpMaHRHMCtNWVIvS2VTWEpJQkhtU2podlMxQ3VEck8Kc2l3RnBOMW84ZFA1UmhBTDVNR1BxL21wNngxNzQzMHplT0Z3N0dBSFZRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBRU80SWl3MUMrMlk1R2wzTW5aV1FzaU9nb0FRcHFDVHJJaDhFQU83azdTZ3lOajFXcnJKClFuYVM3ZnpKYi8xZ0E5S2FHeGlKUDhQdUQ1bS85Z2xUNk1BbCtFYk80RGhNSXd2b1NnTndRM3Q0dUUvNXAya2UKaHAvT2MvZUxreStsbjBVbDZFQVJ6YVVKcGhhZTgvQ2NnZlJLWjEzQVVzWlcyVW51eHNLMDlSajhNc2FLVHVBLwp3UzFzZzdabUpKWTRnM1A4ZHBWZmZIdzNucW10dXkyelAxZ01XU2o2MnhtcHg1WVg5WXY1U3FKRWRBdWJraytZClBsUFhtUGZsUnl2cTM2RHM2alppVG9VT0RpNWpiTHh6SGoyUGlCY2lyLzFmNDZNUDNxZGY0ano5akxQVHJVVUsKaHBRT2x4WFF0M3RYQlFmUUY1a0loMmRRWW9GVng1ZUMzTmc9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
controlplane ~ ➜  
```

create a CSR name akshay.yaml as follows: 

template:

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
     <encoded-csr>

```yaml

---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTdmM1p5YXFqM255UlF6WEZUK3ZsY01MSllUVWM2a20rSEJCN2RTVWJ0VTZqCm11UmlDcytoLzluOU5UK2VYdG9DT0RwclpNY2szSnk2UXB0NWE5dGZLcWl2ZlVzU2JIa0JWekxTcWZSaVpzYUEKZnZGaGpYN3RsTzJ2TURIWFlqWC9FbEJUb01OcFJNYkl6NlBWR05TQk4vQjFCVWZ5VnlpWldHamUvSWdwSDBlNQprT3UvaUhocVVzQ1ExZktXTjJnaFY5bWttME5PbzBxTTRMeWdPN3pRdzlFallwai9FS1lyVDJlSTVlSVRxSHBVCnRZeHVKSmh2bW9ydXFZc3pSamdlVU9NalBUS0FsdGROeFpMaHRHMCtNWVIvS2VTWEpJQkhtU2podlMxQ3VEck8Kc2l3RnBOMW84ZFA1UmhBTDVNR1BxL21wNngxNzQzMHplT0Z3N0dBSFZRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBRU80SWl3MUMrMlk1R2wzTW5aV1FzaU9nb0FRcHFDVHJJaDhFQU83azdTZ3lOajFXcnJKClFuYVM3ZnpKYi8xZ0E5S2FHeGlKUDhQdUQ1bS85Z2xUNk1BbCtFYk80RGhNSXd2b1NnTndRM3Q0dUUvNXAya2UKaHAvT2MvZUxreStsbjBVbDZFQVJ6YVVKcGhhZTgvQ2NnZlJLWjEzQVVzWlcyVW51eHNLMDlSajhNc2FLVHVBLwp3UzFzZzdabUpKWTRnM1A4ZHBWZmZIdzNucW10dXkyelAxZ01XU2o2MnhtcHg1WVg5WXY1U3FKRWRBdWJraytZClBsUFhtUGZsUnl2cTM2RHM2alppVG9VT0RpNWpiTHh6SGoyUGlCY2lyLzFmNDZNUDNxZGY0ano5akxQVHJVVUsKaHBRT2x4WFF0M3RYQlFmUUY1a0loMmRRWW9GVng1ZUMzTmc9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400 # one day
  usages:
  - client auth
```

Apply the csr object

```bash
controlplane ~ ➜  kubectl apply -f akshay.yaml 
certificatesigningrequest.certificates.k8s.io/akshay created

controlplane ~ ➜  
```


Once the object is created, all the certificate signing requests can  be seen by the admin

What is the Condition of the newly created Certificate Signing Request object? - **Pending**

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      58s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Pending
csr-qlrzn   23m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

controlplane ~ ➜ 
```

Approve the CSR request

```bash
controlplane ~ ➜  kubectl certificate approve akshay
certificatesigningrequest.certificates.k8s.io/akshay approved

controlplane ~ ➜ 
```

Kubernetes signs the certificate with the CA key pair. The certificate is then extracted and shared with the new user
 
You can view the generated certificate in a yaml format, under section "status". The certificate is in base64 format

You are not aware of a request coming in. What groups is this CSR requesting access to?

```bash
controlplane ~ ➜  kubectl get csr agent-smith -o yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  creationTimestamp: "2023-01-18T13:05:49Z"
  name: agent-smith
  resourceVersion: "2423"
  uid: ea3c0298-4a96-4caf-8e77-736897ff216e
spec:
  groups:
  - system:masters
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dEQ0NBVUFDQVFBd0V6RVJNQThHQTFVRUF3d0libVYzTFhWelpYSXdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRE8wV0pXK0RYc0FKU0lyanBObzV2UklCcGxuemcrNnhjOStVVndrS2kwCkxmQzI3dCsxZUVuT041TXVxOTlOZXZtTUVPbnJEVU8vdGh5VnFQMncyWE5JRFJYall5RjQwRmJtRCs1eld5Q0sKeTNCaWhoQjkzTUo3T3FsM1VUdlo4VEVMcXlhRGtuUmwvanYvU3hnWGtvazBBQlVUcFdNeDRCcFNpS2IwVSt0RQpJRjVueEF0dE1Wa0RQUTdOYmVaUkc0M2IrUVdsVkdSL3o2RFdPZkpuYmZlek90YUF5ZEdMVFpGQy93VHB6NTJrCkVjQ1hBd3FDaGpCTGt6MkJIUFI0Sjg5RDZYYjhrMzlwdTZqcHluZ1Y2dVAwdEliT3pwcU52MFkwcWRFWnB3bXcKajJxRUwraFpFV2trRno4MGxOTnR5VDVMeE1xRU5EQ25JZ3dDNEdaaVJHYnJBZ01CQUFHZ0FEQU5CZ2txaGtpRwo5dzBCQVFzRkFBT0NBUUVBUzlpUzZDMXV4VHVmNUJCWVNVN1FGUUhVemFsTnhBZFlzYU9SUlFOd0had0hxR2k0CmhPSzRhMnp5TnlpNDRPT2lqeWFENnRVVzhEU3hrcjhCTEs4S2czc3JSRXRKcWw1ckxaeTlMUlZyc0pnaEQ0Z1kKUDlOTCthRFJTeFJPVlNxQmFCMm5XZVlwTTVjSjVURjUzbGVzTlNOTUxRMisrUk1uakRRSjdqdVBFaWM4L2RoawpXcjJFVU02VWF3enlrcmRISW13VHYybWxNWTBSK0ROdFYxWWllKzBIOS9ZRWx0K0ZTR2poNUw1WVV2STFEcWl5CjRsM0UveTNxTDcxV2ZBY3VIM09zVnBVVW5RSVNNZFFzMHFXQ3NiRTU2Q0M1RGhQR1pJcFVibktVcEF3a2ErOEUKdndRMDdqRytocGtueG11RkFlWHhnVXdvZEFMYUo3anUvVERJY3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - server auth
  username: agent-x
status: {}

controlplane ~ ➜  
```


To decode the certificate into plain text format

```bash
echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dEQ0NBVUFDQVFBd0V6RVJNQThHQTFVRUF3d0libVYzTFhWelpYSXdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRE8wV0pXK0RYc0FKU0lyanBObzV2UklCcGxuemcrNnhjOStVVndrS2kwCkxmQzI3dCsxZUVuT041TXVxOTlOZXZtTUVPbnJEVU8vdGh5VnFQMncyWE5JRFJYall5RjQwRmJtRCs1eld5Q0sKeTNCaWhoQjkzTUo3T3FsM1VUdlo4VEVMcXlhRGtuUmwvanYvU3hnWGtvazBBQlVUcFdNeDRCcFNpS2IwVSt0RQpJRjVueEF0dE1Wa0RQUTdOYmVaUkc0M2IrUVdsVkdSL3o2RFdPZkpuYmZlek90YUF5ZEdMVFpGQy93VHB6NTJrCkVjQ1hBd3FDaGpCTGt6MkJIUFI0Sjg5RDZYYjhrMzlwdTZqcHluZ1Y2dVAwdEliT3pwcU52MFkwcWRFWnB3bXcKajJxRUwraFpFV2trRno4MGxOTnR5VDVMeE1xRU5EQ25JZ3dDNEdaaVJHYnJBZ01CQUFHZ0FEQU5CZ2txaGtpRwo5dzBCQVFzRkFBT0NBUUVBUzlpUzZDMXV4VHVmNUJCWVNVN1FGUUhVemFsTnhBZFlzYU9SUlFOd0had0hxR2k0CmhPSzRhMnp5TnlpNDRPT2lqeWFENnRVVzhEU3hrcjhCTEs4S2czc3JSRXRKcWw1ckxaeTlMUlZyc0pnaEQ0Z1kKUDlOTCthRFJTeFJPVlNxQmFCMm5XZVlwTTVjSjVURjUzbGVzTlNOTUxRMisrUk1uakRRSjdqdVBFaWM4L2RoawpXcjJFVU02VWF3enlrcmRISW13VHYybWxNWTBSK0ROdFYxWWllKzBIOS9ZRWx0K0ZTR2poNUw1WVV2STFEcWl5CjRsM0UveTNxTDcxV2ZBY3VIM09zVnBVVW5RSVNNZFFzMHFXQ3NiRTU2Q0M1RGhQR1pJcFVibktVcEF3a2ErOEUKdndRMDdqRytocGtueG11RkFlWHhnVXdvZEFMYUo3anUvVERJY3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K" | base64 --decode
```

To deny the request

```bash
controlplane ~ ➜  kubectl certificate deny agent-smith
certificatesigningrequest.certificates.k8s.io/agent-smith denied

controlplane ~ ➜  
```

Status of the CSR

```bash
controlplane ~ ➜  kubectl get csr
NAME          AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
agent-smith   5m49s   kubernetes.io/kube-apiserver-client           agent-x                    <none>              Denied
akshay        8m47s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Approved,Issued
csr-qlrzn     31m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

controlplane ~ ➜  
```

Delete the new CSR

```bash
controlplane ~ ➜  kubectl delete csr agent-smith 
certificatesigningrequest.certificates.k8s.io "agent-smith" deleted

controlplane ~ ➜  
```