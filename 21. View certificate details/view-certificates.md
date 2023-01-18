


```bash 

controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.33.216.6:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.33.216.6
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.k8s.io/kube-apiserver:v1.26.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 10.33.216.6
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 10.33.216.6
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 10.33.216.6
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

controlplane ~ ➜  
```

Identify the Certificate file used to authenticate kube-apiserver as a client to ETCD Server - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt



Identify the key used to authenticate kubeapi-server to the kubelet server - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key


Identify the certificate file used for the kube-api server - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt


```bash

controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.33.216.6:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.33.216.6:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://10.33.216.6:2380
    - --initial-cluster=controlplane=https://10.33.216.6:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.33.216.6:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.33.216.6:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}

controlplane ~ ➜ 
```

Identify the ETCD Server Certificate used to host ETCD server  --cert-file=/etc/kubernetes/pki/etcd/server.crt

Identify the ETCD Server CA Root Certificate used to serve ETCD Server. ETCD can have its own CA. So this may be a different CA certificate than the one used by kube-api server.  --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt


What is the Common Name (CN) configured on the Kube API Server Certificate? (--tls-cert-file=/etc/kubernetes/pki/apiserver.crt) -

Subject: CN = kube-apiserver

OpenSSL Syntax: openssl x509 -in file-path.crt -text -noout

```bash

controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4022723326930499516 (0x37d3958d735eb7bc)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jan 17 07:51:02 2023 GMT
            Not After : Jan 17 07:51:03 2024 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:d6:e3:fc:b1:82:90:44:97:0e:69:a8:ac:2e:d6:
                    69:7b:1b:34:f6:20:9e:42:f4:1e:72:12:69:23:53:
                    39:75:a9:db:46:41:fe:a9:6d:cd:72:c8:27:91:da:
                    3d:51:3d:f7:25:62:bf:90:cc:8d:7a:bd:7b:dd:99:
                    cd:b1:48:ee:4a:dc:ba:e6:11:bb:25:9f:18:96:e0:
                    5d:bb:8e:85:34:87:5d:84:f8:af:69:a1:eb:d6:37:
                    d1:61:27:53:5b:b4:8f:6b:8d:5d:ba:9e:92:e1:4a:
                    c5:3d:e0:9f:05:5e:0d:0d:60:10:4d:54:11:95:37:
                    0d:76:da:99:97:a1:75:71:a5:dd:b4:72:69:5b:7c:
                    b5:24:6c:82:61:e8:53:ed:be:4b:01:f7:06:96:8f:
                    0d:ef:94:61:90:25:fd:69:e0:a8:94:db:8a:7e:1a:
                    15:db:24:3a:27:c5:f2:a4:1f:4c:a7:03:59:6d:94:
                    71:21:8b:0b:1d:5c:e7:bb:3e:25:21:01:bf:e8:3a:
                    d4:9c:15:27:07:67:d0:4c:88:f2:a5:08:fd:84:d9:
                    68:38:eb:1d:af:15:27:43:4d:72:3c:c5:ca:03:43:
                    03:92:3f:d9:2d:9c:c6:f1:5c:d8:2e:01:fb:5c:8c:
                    77:78:6c:93:e6:b7:9a:a1:66:90:e2:b5:1e:bf:63:
                    13:27
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:04:06:3F:93:4A:52:24:99:DE:B2:8A:76:FF:51:37:D0:D4:18:57:1F

            X509v3 Subject Alternative Name: 
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:10.33.216.6
    Signature Algorithm: sha256WithRSAEncryption
         45:fe:c8:c8:75:78:75:0c:cd:a0:aa:12:2a:90:3e:fc:b2:48:
         c8:a5:09:f6:16:ef:c1:52:2a:74:ba:ee:0c:b5:98:9a:56:a4:
         d2:34:d2:af:e8:3d:78:aa:c3:ca:42:d9:1f:c7:c1:ff:1e:38:
         e3:51:7e:54:d9:eb:a7:24:e2:d7:4e:18:2f:f4:1b:53:e4:51:
         b4:47:18:32:8b:00:63:fd:2e:19:c9:df:ae:c8:ff:9b:1c:10:
         63:2e:d7:15:cc:a9:40:fa:8a:ab:02:f1:31:7c:68:46:a8:c9:
         8e:e8:2c:72:88:7a:f2:58:4d:75:c2:f2:c9:6d:80:59:1f:e6:
         5a:e7:22:36:36:b7:87:b4:44:38:a3:e6:10:5b:c9:6b:62:5f:
         90:e0:69:b5:70:77:a8:9b:e0:3c:2b:c8:42:97:0d:6b:d8:89:
         db:bd:6c:ba:75:6b:8a:2c:de:bb:e9:ec:6d:7c:4e:08:a8:44:
         27:cc:a9:62:58:bc:8d:9a:d5:54:55:12:49:ca:cb:a6:29:74:
         53:a7:50:14:31:75:7b:89:05:80:16:36:25:3e:4a:35:1a:93:
         de:e3:41:42:9a:27:b0:ce:ae:df:93:ca:4b:14:43:70:54:45:
         45:2b:30:16:e1:bb:8f:fa:18:12:1e:01:c2:88:b1:94:d9:fc:
         a7:ce:50:56
-----BEGIN CERTIFICATE-----
MIIDjDCCAnSgAwIBAgIIN9OVjXNet7wwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yMzAxMTcwNzUxMDJaFw0yNDAxMTcwNzUxMDNaMBkx
FzAVBgNVBAMTDmt1YmUtYXBpc2VydmVyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEA1uP8sYKQRJcOaaisLtZpexs09iCeQvQechJpI1M5danbRkH+qW3N
csgnkdo9UT33JWK/kMyNer173ZnNsUjuSty65hG7JZ8YluBdu46FNIddhPivaaHr
1jfRYSdTW7SPa41dup6S4UrFPeCfBV4NDWAQTVQRlTcNdtqZl6F1caXdtHJpW3y1
JGyCYehT7b5LAfcGlo8N75RhkCX9aeColNuKfhoV2yQ6J8XypB9MpwNZbZRxIYsL
HVznuz4lIQG/6DrUnBUnB2fQTIjypQj9hNloOOsdrxUnQ01yPMXKA0MDkj/ZLZzG
8VzYLgH7XIx3eGyT5reaoWaQ4rUev2MTJwIDAQABo4HbMIHYMA4GA1UdDwEB/wQE
AwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB8GA1UdIwQY
MBaAFAQGP5NKUiSZ3rKKdv9RN9DUGFcfMIGBBgNVHREEejB4ggxjb250cm9scGxh
bmWCCmt1YmVybmV0ZXOCEmt1YmVybmV0ZXMuZGVmYXVsdIIWa3ViZXJuZXRlcy5k
ZWZhdWx0LnN2Y4Ika3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2Fs
hwQKYAABhwQKIdgGMA0GCSqGSIb3DQEBCwUAA4IBAQBF/sjIdXh1DM2gqhIqkD78
skjIpQn2Fu/BUip0uu4MtZiaVqTSNNKv6D14qsPKQtkfx8H/HjjjUX5U2eunJOLX
Thgv9BtT5FG0RxgyiwBj/S4Zyd+uyP+bHBBjLtcVzKlA+oqrAvExfGhGqMmO6Cxy
iHryWE11wvLJbYBZH+Za5yI2NreHtEQ4o+YQW8lrYl+Q4Gm1cHeom+A8K8hClw1r
2InbvWy6dWuKLN676extfE4IqEQnzKliWLyNmtVUVRJJysumKXRTp1AUMXV7iQWA
FjYlPko1GpPe40FCmiewzq7fk8pLFENwVEVFKzAW4buP+hgSHgHCiLGU2fynzlBW
-----END CERTIFICATE-----

controlplane ~ ➜  
```

What is the name of the CA who issued the Kube API Server Certificate? - Issuer: CN = kubernetes

What is the Common Name (CN) configured on the ETCD Server certificate? -  Subject: CN = controlplane

```bash
controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 793245835379272166 (0xb022ce26d56e9e6)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = etcd-ca
        Validity
            Not Before: Jan 17 11:21:51 2023 GMT
            Not After : Jan 17 11:21:51 2024 GMT
        Subject: CN = controlplane
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:ca:61:39:1e:0f:10:50:67:82:a4:cf:95:20:a1:
                    7b:96:9c:74:97:bd:45:0c:a7:70:bf:0e:29:71:c6:
                    e5:4d:f6:fb:92:1e:e2:58:a5:d3:d7:e3:08:f8:5f:
                    7f:59:ab:eb:33:8b:d7:8e:25:5e:3d:01:6d:50:53:
                    16:e5:b7:47:01:49:50:17:89:c8:94:29:df:35:4c:
                    ab:f7:b5:99:6c:da:6c:16:47:95:d8:f0:78:32:0f:
                    3c:a3:6c:92:28:a6:a0:29:55:98:30:20:04:c2:6d:
                    0f:f1:4b:46:2f:c8:14:cb:b0:c2:5e:8c:f7:82:5d:
                    10:be:67:6c:bb:81:b7:82:6f:05:33:57:ad:3d:42:
                    d0:66:0d:69:0b:9e:16:44:10:67:6e:60:11:c9:7f:
                    77:26:09:80:7b:37:29:44:b8:44:8b:47:36:b7:e2:
                    29:6f:6b:ad:19:00:03:19:1f:37:05:4d:df:b1:f0:
                    b1:4c:dd:8b:0a:ca:76:a1:5a:22:2d:25:63:34:9c:
                    5e:79:52:03:45:23:7a:ff:cf:96:50:c4:57:dd:82:
                    a1:15:10:fe:46:02:71:e8:e7:ea:c2:46:bf:6c:76:
                    62:41:64:e7:fa:db:88:fb:f3:ff:5e:ba:d1:5c:01:
                    89:0b:5c:f9:3b:80:50:b0:f7:2c:84:1c:b0:f2:2e:
                    b7:0b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:C1:DB:E0:D7:3B:E6:55:03:39:B4:EF:52:B1:19:B7:3B:41:A8:86:B0

            X509v3 Subject Alternative Name: 
                DNS:controlplane, DNS:localhost, IP Address:10.51.13.9, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
    Signature Algorithm: sha256WithRSAEncryption
         7f:dd:83:ba:a8:60:fc:51:59:3f:38:51:db:ae:5d:07:6a:82:
         30:e9:75:c0:ed:e3:7d:15:e7:c7:fd:33:5e:f6:22:98:21:45:
         ba:b7:38:5e:c8:2f:99:54:e5:95:27:12:6e:c6:c0:a4:40:f9:
         f6:8d:44:8f:57:34:c0:b7:05:f7:a6:29:b5:b0:a8:10:3e:8d:
         fa:02:51:cf:1a:16:12:81:d1:57:37:c3:d6:41:03:03:5b:e9:
         ce:a6:ae:d1:99:24:0a:96:94:2c:9f:a4:e2:0c:0e:de:0f:61:
         05:e0:e8:b9:c4:8c:8c:38:62:81:4d:75:4e:a1:c0:53:cd:5b:
         84:68:d1:af:e0:7c:66:10:6b:7d:11:26:07:30:dc:16:54:d5:
         15:31:58:80:ff:50:9a:31:26:05:1a:0c:14:b4:3a:29:dd:56:
         1d:1c:24:c5:9f:25:3b:39:b3:f7:75:f7:46:17:c8:00:40:80:
         9d:6b:a0:f3:e0:9a:1e:9e:31:f9:38:a1:e2:37:2d:72:7f:4b:
         43:16:08:a2:eb:a0:1e:36:da:ea:dc:5d:ac:56:1f:ea:56:67:
         e2:a3:6d:b6:ba:e7:28:0e:3d:4f:23:c2:8f:30:d0:fc:76:51:
         db:4d:c8:73:43:0e:fb:b5:1f:e3:fe:ad:75:9f:06:5d:79:99:
         a7:8e:88:bb
-----BEGIN CERTIFICATE-----
MIIDTzCCAjegAwIBAgIICwIs4m1W6eYwDQYJKoZIhvcNAQELBQAwEjEQMA4GA1UE
AxMHZXRjZC1jYTAeFw0yMzAxMTcxMTIxNTFaFw0yNDAxMTcxMTIxNTFaMBcxFTAT
BgNVBAMTDGNvbnRyb2xwbGFuZTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAMphOR4PEFBngqTPlSChe5acdJe9RQyncL8OKXHG5U32+5Ie4lil09fjCPhf
f1mr6zOL144lXj0BbVBTFuW3RwFJUBeJyJQp3zVMq/e1mWzabBZHldjweDIPPKNs
kiimoClVmDAgBMJtD/FLRi/IFMuwwl6M94JdEL5nbLuBt4JvBTNXrT1C0GYNaQue
FkQQZ25gEcl/dyYJgHs3KUS4RItHNrfiKW9rrRkAAxkfNwVN37HwsUzdiwrKdqFa
Ii0lYzScXnlSA0Ujev/PllDEV92CoRUQ/kYCcejn6sJGv2x2YkFk5/rbiPvz/166
0VwBiQtc+TuAULD3LIQcsPIutwsCAwEAAaOBozCBoDAOBgNVHQ8BAf8EBAMCBaAw
HQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHwYD
VR0jBBgwFoAUwdvg1zvmVQM5tO9SsRm3O0GohrAwQAYDVR0RBDkwN4IMY29udHJv
bHBsYW5lgglsb2NhbGhvc3SHBAozDQmHBH8AAAGHEAAAAAAAAAAAAAAAAAAAAAEw
DQYJKoZIhvcNAQELBQADggEBAH/dg7qoYPxRWT84UduuXQdqgjDpdcDt430V58f9
M172IpghRbq3OF7IL5lU5ZUnEm7GwKRA+faNRI9XNMC3BfemKbWwqBA+jfoCUc8a
FhKB0Vc3w9ZBAwNb6c6mrtGZJAqWlCyfpOIMDt4PYQXg6LnEjIw4YoFNdU6hwFPN
W4Ro0a/gfGYQa30RJgcw3BZU1RUxWID/UJoxJgUaDBS0OindVh0cJMWfJTs5s/d1
90YXyABAgJ1roPPgmh6eMfk4oeI3LXJ/S0MWCKLroB422urcXaxWH+pWZ+Kjbba6
5ygOPU8jwo8w0Px2UdtNyHNDDvu1H+P+rXWfBl15maeOiLs=
-----END CERTIFICATE-----

controlplane ~ ➜  
```

How long, from the issued date, is the Kube-API Server Certificate valid for? - 

			Not Before: Jan 17 11:21:50 2023 GMT
            Not After : Jan 17 11:21:50 2024 GMT


File: /etc/kubernetes/pki/apiserver.crt

```bash

controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2602209403090526459 (0x241ce7aa3e6460fb)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jan 17 11:21:50 2023 GMT
            Not After : Jan 17 11:21:50 2024 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:d5:1a:60:a9:35:5a:c6:a1:06:6e:71:03:0c:0b:
                    00:4e:30:62:40:34:8b:cb:c9:11:e1:5a:5a:1e:ac:
                    02:d7:99:87:12:59:fd:00:aa:c3:b3:c0:e7:9c:40:
                    6e:7f:bc:fc:2c:cc:bb:5e:04:d6:40:d9:82:64:9e:
                    50:17:0f:56:fb:f4:c5:91:b6:fe:09:11:8d:44:d2:
                    0f:af:b0:b0:27:01:7e:92:58:cf:53:bc:0a:8e:95:
                    3e:78:2b:e1:c7:33:4a:07:95:cb:31:58:f5:d0:0b:
                    43:ef:df:67:12:2f:29:fc:7d:69:7c:84:98:ae:41:
                    8b:ce:ce:9b:e2:d2:03:bd:31:5a:4c:18:da:e5:dc:
                    a1:dc:59:16:c3:94:1a:b7:04:6f:24:37:44:ea:21:
                    d7:3b:4d:6f:cd:57:2d:ba:4e:21:22:60:cb:22:e2:
                    af:38:5b:f7:61:72:f5:b5:04:8f:41:e1:42:11:6d:
                    33:93:b2:39:86:1b:a1:8c:e5:8c:44:83:70:cb:9a:
                    e7:6e:b2:22:71:e0:3a:5d:3d:e1:13:89:f0:92:18:
                    1d:bf:77:c5:54:72:bd:15:7b:a6:5d:03:4f:32:e0:
                    ac:97:22:9d:1e:2d:26:bd:23:86:a9:d6:0d:12:d7:
                    81:b6:f5:7e:4c:ab:05:12:b4:db:cf:92:71:14:4e:
                    ea:39
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:C5:EE:D7:B7:90:2E:B9:E3:33:82:F2:3D:3D:B2:20:88:7D:D9:E8:FB

            X509v3 Subject Alternative Name: 
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:10.51.13.9
    Signature Algorithm: sha256WithRSAEncryption
         80:85:00:1f:fc:1a:36:f6:6e:b9:e3:27:80:ff:ee:62:bd:32:
         28:a2:89:08:57:bb:5d:3c:d6:71:70:c4:ac:ca:14:ea:81:9e:
         39:9d:cf:ec:06:68:6c:df:a6:fc:9c:d5:de:0c:02:b2:41:0a:
         8a:fb:96:59:b8:72:15:35:a7:90:2a:12:40:08:d3:a4:04:80:
         78:85:4e:01:a0:75:33:a0:7f:ba:90:2f:2a:0a:e1:49:6d:12:
         d6:61:10:2f:33:16:16:79:0a:c4:26:ce:b1:3e:a7:73:1f:d6:
         48:21:33:42:b7:c4:ea:99:31:38:74:48:e6:f9:53:66:6b:4b:
         b7:bb:b4:98:c1:2a:9e:34:76:10:e4:4d:78:28:cf:4a:38:d7:
         a8:b7:55:54:cf:d1:f1:f6:35:eb:7d:d3:c7:08:b5:af:47:da:
         23:91:90:08:31:76:8e:17:8e:11:d9:ae:b7:6f:f3:31:d9:cf:
         fd:f6:e9:56:e2:6b:7b:c4:52:23:ea:96:21:19:7d:67:9e:8e:
         8d:a6:34:98:21:80:d2:fa:66:79:dd:d3:f8:2c:d4:a8:1f:3b:
         47:63:b4:d7:81:11:7c:53:89:9e:37:c5:0d:94:be:74:83:ba:
         d1:4f:1f:d6:2f:76:88:55:93:a9:4a:96:ad:09:9f:18:e4:2e:
         f4:52:ab:97
-----BEGIN CERTIFICATE-----
MIIDjDCCAnSgAwIBAgIIJBznqj5kYPswDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yMzAxMTcxMTIxNTBaFw0yNDAxMTcxMTIxNTBaMBkx
FzAVBgNVBAMTDmt1YmUtYXBpc2VydmVyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEA1RpgqTVaxqEGbnEDDAsATjBiQDSLy8kR4VpaHqwC15mHEln9AKrD
s8DnnEBuf7z8LMy7XgTWQNmCZJ5QFw9W+/TFkbb+CRGNRNIPr7CwJwF+kljPU7wK
jpU+eCvhxzNKB5XLMVj10AtD799nEi8p/H1pfISYrkGLzs6b4tIDvTFaTBja5dyh
3FkWw5QatwRvJDdE6iHXO01vzVctuk4hImDLIuKvOFv3YXL1tQSPQeFCEW0zk7I5
hhuhjOWMRINwy5rnbrIiceA6XT3hE4nwkhgdv3fFVHK9FXumXQNPMuCslyKdHi0m
vSOGqdYNEteBtvV+TKsFErTbz5JxFE7qOQIDAQABo4HbMIHYMA4GA1UdDwEB/wQE
AwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB8GA1UdIwQY
MBaAFMXu17eQLrnjM4LyPT2yIIh92ej7MIGBBgNVHREEejB4ggxjb250cm9scGxh
bmWCCmt1YmVybmV0ZXOCEmt1YmVybmV0ZXMuZGVmYXVsdIIWa3ViZXJuZXRlcy5k
ZWZhdWx0LnN2Y4Ika3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2Fs
hwQKYAABhwQKMw0JMA0GCSqGSIb3DQEBCwUAA4IBAQCAhQAf/Bo29m654yeA/+5i
vTIoookIV7tdPNZxcMSsyhTqgZ45nc/sBmhs36b8nNXeDAKyQQqK+5ZZuHIVNaeQ
KhJACNOkBIB4hU4BoHUzoH+6kC8qCuFJbRLWYRAvMxYWeQrEJs6xPqdzH9ZIITNC
t8TqmTE4dEjm+VNma0u3u7SYwSqeNHYQ5E14KM9KONeot1VUz9Hx9jXrfdPHCLWv
R9ojkZAIMXaOF44R2a63b/Mx2c/99ulW4mt7xFIj6pYhGX1nno6NpjSYIYDS+mZ5
3dP4LNSoHztHY7TXgRF8U4meN8UNlL50g7rRTx/WL3aIVZOpSpatCZ8Y5C70UquX
-----END CERTIFICATE-----

controlplane ~ ➜ 
```

How long, from the issued date, is the Root CA Certificate valid for? -

Validity
            Not Before: Jan 17 11:21:50 2023 GMT
            Not After : Jan 14 11:21:50 2033 GMT


File: /etc/kubernetes/pki/ca.crt

```bash
controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/ca.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jan 17 11:21:50 2023 GMT
            Not After : Jan 14 11:21:50 2033 GMT
        Subject: CN = kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:ca:1e:97:9e:1a:e1:9c:e2:0b:f3:ae:69:7c:04:
                    b7:8d:d0:91:70:b2:8b:00:74:6b:33:5a:cc:c6:e6:
                    bf:cf:2d:28:b1:7a:1c:8d:3d:76:cf:d3:29:2d:9d:
                    d6:06:98:f7:75:ce:b9:ba:25:0c:f6:e2:f6:db:c8:
                    9d:2b:51:9c:21:06:ec:0e:64:21:3f:77:d7:84:ac:
                    7e:9a:46:25:7a:f0:08:1b:3c:d2:0f:0f:7c:8f:28:
                    c4:15:3e:b1:e6:db:6e:2b:00:92:d1:dc:04:ac:89:
                    7f:52:c9:ef:3f:cb:67:92:e0:ee:b3:f2:5d:d2:18:
                    72:ec:92:41:17:dd:30:0b:45:a4:c3:7d:3d:cd:cd:
                    d9:19:6f:01:f9:28:dc:cf:c5:1b:83:d6:a8:9b:f4:
                    d1:51:70:50:90:b2:6d:7a:e7:54:cb:fb:20:b3:b6:
                    9b:f3:07:83:a3:43:bf:30:be:48:d4:0c:17:c8:29:
                    47:fd:ae:0b:20:a9:09:b8:50:d3:61:00:6d:81:24:
                    69:94:11:ab:49:87:0b:73:2a:30:08:12:56:cb:fe:
                    9e:98:77:78:86:89:94:ee:6d:58:41:03:6a:c5:fa:
                    02:ea:17:e2:e1:e9:e3:e0:9e:aa:d7:a7:1b:65:c8:
                    9a:99:04:08:79:e0:3a:e6:9b:5d:6c:a3:70:69:ea:
                    23:97
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                C5:EE:D7:B7:90:2E:B9:E3:33:82:F2:3D:3D:B2:20:88:7D:D9:E8:FB
            X509v3 Subject Alternative Name: 
                DNS:kubernetes
    Signature Algorithm: sha256WithRSAEncryption
         6b:7f:dd:ee:fe:13:f6:6a:86:5c:e2:f8:7f:6f:f6:87:03:d0:
         79:28:fc:f3:8a:d1:64:19:6c:ea:da:92:96:38:ed:9c:fe:c1:
         b6:88:d8:af:54:41:8a:96:1e:08:e1:2e:6b:fe:fd:76:47:cb:
         21:17:18:ac:b9:b4:88:64:3a:a9:37:3d:d1:46:b9:d0:e7:e7:
         d1:9f:5c:17:60:19:6e:b2:b2:37:6b:8a:e1:48:fa:e6:bd:6a:
         db:31:b2:99:74:c5:0e:1d:a6:91:32:0a:d3:59:5c:fd:0e:23:
         74:40:3e:51:f4:c4:af:ab:aa:fc:29:52:4f:c6:84:fe:b0:53:
         5c:d3:1a:e4:cf:99:4d:c8:bf:59:b6:98:ea:bc:68:fd:0a:72:
         a8:b3:b0:9d:69:1e:1d:9f:af:2d:3e:fc:dc:bb:c5:d6:a9:69:
         f6:d2:a9:c8:ff:74:0c:35:ca:2e:ae:dd:85:eb:3c:e3:63:e9:
         41:45:b7:f4:ba:d3:8d:59:41:c2:4c:db:96:b0:9c:fb:06:5b:
         83:8e:05:68:41:39:87:f1:6e:38:76:f1:21:d9:93:6e:57:a0:
         31:75:14:21:37:aa:3a:78:f1:2c:bb:ff:1d:d9:28:04:ce:d5:
         ef:38:94:86:0a:a9:fd:33:6b:4e:87:77:49:1b:ea:1c:f3:5d:
         bc:49:ba:db
-----BEGIN CERTIFICATE-----
MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTIzMDExNzExMjE1MFoXDTMzMDExNDExMjE1MFowFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMoe
l54a4ZziC/OuaXwEt43QkXCyiwB0azNazMbmv88tKLF6HI09ds/TKS2d1gaY93XO
ubolDPbi9tvInStRnCEG7A5kIT9314SsfppGJXrwCBs80g8PfI8oxBU+sebbbisA
ktHcBKyJf1LJ7z/LZ5Lg7rPyXdIYcuySQRfdMAtFpMN9Pc3N2RlvAfko3M/FG4PW
qJv00VFwUJCybXrnVMv7ILO2m/MHg6NDvzC+SNQMF8gpR/2uCyCpCbhQ02EAbYEk
aZQRq0mHC3MqMAgSVsv+nph3eIaJlO5tWEEDasX6AuoX4uHp4+CeqtenG2XImpkE
CHngOuabXWyjcGnqI5cCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wHQYDVR0OBBYEFMXu17eQLrnjM4LyPT2yIIh92ej7MBUGA1UdEQQO
MAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBAGt/3e7+E/Zqhlzi+H9v
9ocD0Hko/POK0WQZbOrakpY47Zz+wbaI2K9UQYqWHgjhLmv+/XZHyyEXGKy5tIhk
Oqk3PdFGudDn59GfXBdgGW6ysjdriuFI+ua9atsxspl0xQ4dppEyCtNZXP0OI3RA
PlH0xK+rqvwpUk/GhP6wU1zTGuTPmU3Iv1m2mOq8aP0KcqizsJ1pHh2fry0+/Ny7
xdapafbSqcj/dAw1yi6u3YXrPONj6UFFt/S6041ZQcJM25awnPsGW4OOBWhBOYfx
bjh28SHZk25XoDF1FCE3qjp48Sy7/x3ZKATO1e84lIYKqf0za06Hd0kb6hzzXbxJ
uts=
-----END CERTIFICATE-----

controlplane ~ ➜  
```

The kube-api server suddenly stops responding to your commands. Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.

Run crictl ps -a command to identify the kube-api server container. Run crictl logs container-id command to view the logs.

```bash
controlplane ~ ➜  crictl ps -a | grep kube-apiserver
a4af247061d2f       a31e1d84401e6       6 seconds ago        Running             kube-apiserver            1                   b7c4f3bc53355       kube-apiserver-controlplane
1e83f521cfefc       a31e1d84401e6       30 seconds ago       Exited              kube-apiserver            0                   b7c4f3bc53355       kube-apiserver-controlplane

controlplane ~ ➜  
```

If we inspect the kube-apiserver container on the controlplane, we can see that it is frequently exiting.

If we now inspect the logs of this exited container, we would see the following errors:

```bash

root@controlplane:~# crictl logs --tail=2 1fb242055cff8  
W0916 14:19:44.771920       1 clientconn.go:1331] [core] grpc: addrConn.createTransport failed to connect to {127.0.0.1:2379 127.0.0.1 <nil> 0 <nil>}. Err: connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting...
E0916 14:19:48.689303       1 run.go:74] "command failed" err="context deadline exceeded"
```

This indicates an issue with the ETCD CA certificate used by the kube-apiserver. Correct it to use the file /etc/kubernetes/pki/etcd/ca.crt.

Once the YAML file has been saved, wait for the kube-apiserver pod to be Ready. This can take a couple of minutes.

***The End***