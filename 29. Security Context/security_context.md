- Docker container processes runs in a separate namespace as the host processes that also runs in a separate namespace
- The docker process is seen in the host processes list as a process
- By default docker containers run with root user privileges
- To run docker with another user:

```bash

docker run --user=1000
```
- This can also be configured in Dockerfile as:

```bash
FROM ubuntu

USER 1000
```

After which you can build the custom image:

```bash
docker build -t custom-image .
```

Then run the image without specifying the user id

```bash
docker run custom-image sleep 3600
```

Viwe linux root user capablities

```bash
cat /usr/include/linuc/capabilities.h
```

- Docker root user has limited set of capabilities unlike the host root user
- Provide additional capabilities to docker container using the *--cap-add* option

```bash
docker run --cap-add MAC_ADMIN ubuntu
```

- Drop capabilities using *--cap-drop* option

```bash
docker run --cap-drop KILL ubuntu
```

- To run container with all the privileges enabled

```bash
docker run --privileged ubuntu
```

- These capabilities can be configured in k8s as well.
- The securities can be configured at the container level or at the pod level
- If configured at the pod level, the settings will apply to all containers in the pod
- If configured at both the pod and container levels, the container security settings will override the pod security settings
- Pod level security context:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1010

  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

- Container level security context:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities: 
        	add ["MAC_ADMIN"]
```

- Note: capabilities are only supported at the container level and not at the pod level

***The End***