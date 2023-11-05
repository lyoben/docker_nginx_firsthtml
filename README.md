# docker-nginx-simple-html

### Contents:

### Description
Introductory to deploy a web service as a Kubernetes pod. First by creating a nginx docker image. Then deploy Kubernetes pod and service to make the service available. Kubernetes implmentation using Minikube single node cluster on Ubuntu VM.

### Pre-requisite
Ubuntu 20.04 Desktop

### Installation
1. Install Docker by following Step 1 — Installing Docker and Step 2 — Executing the Docker Command Without Sudo in this [article](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04/).
2. Install Minikube by following Step 1- 5 in this [article](https://www.linuxtechi.com/how-to-install-minikube-on-ubuntu/)
3. Change

### Setting up
Create a folder
> mkdir -p ~/docker-nginx
>
> cd ~/docker-nginx

Create a static html page
```
cat << EOF > index.html
 <html>
  <head>
    <title>Docker nginx simple html</title>
  </head>

  <body>
    <div class="container">
      <h1>Welcome to Docker nginx simple html!</h1>
      <p> Visit <a href="url">https://github.com/lyoben</a> for more info.
      </p>
    </div>
  </body>
</html>
EOF
```

Create a dockerfile
```
cat << EOF > dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install nginx curl -y
COPY index.html /var/www/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"
EOF
```
Build docker image
> docker build -t simplehtml-nginx:v1 .
```
blow@abra:~/docker-nginx$ docker image ls
REPOSITORY                                           TAG       IMAGE ID       CREATED          SIZE
simplehtml-nginx                                     v1        bf85b8e3e951   49 minutes ago   184MB
---snipped---
```
Create pod definition file
```
cat << EOF > simplehtml-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    env: test
  name: simplehtml
spec:
  containers:
  - image: simplehtml-nginx:v1
    name: simplehtml
    imagePullPolicy: Never
EOF
```
Create service of type nodePort on port 30080
```
cat << EOF > simplehtml-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    env: test
  name: simplehtml-svc
spec:
  ports:
  - name: http
    nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    env: test
  type: NodePort
EOF
```

```
blow@abra:~/docker-nginx$ ip link show type bridge
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:03:98:a4:0b brd ff:ff:ff:ff:ff:ff
5: br-3d7af3f93112: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:b7:e4:e2:d9 brd ff:ff:ff:ff:ff:ff

blow@abra:~/docker-nginx$ ip addr show br-3d7af3f93112
5: br-3d7af3f93112: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:b7:e4:e2:d9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.49.1/24 brd 192.168.49.255 scope global br-3d7af3f93112
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b7ff:fee4:e2d9/64 scope link 
       valid_lft forever preferred_lft forever

```



