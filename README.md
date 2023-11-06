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
Create Pod definition file
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
Create Service of type nodePort on port 30080
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

Check Service endpoint is correctly assosciated to Pod


```
blow@abra:~/docker-nginx$ k get pod -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
simplehtml   1/1     Running   0          46h   10.244.0.12   minikube   <none>           <none>
blow@abra:~/docker-nginx$ k describe svc simplehtml-svc 
Name:                     simplehtml-svc
Namespace:                default
Labels:                   env=test
Annotations:              <none>
Selector:                 env=test
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.97.222.121
IPs:                      10.97.222.121
Port:                     http  80/TCP
TargetPort:               80/TCP
NodePort:                 http  30080/TCP
Endpoints:                10.244.0.12:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
Get the node ip address
```
blow@abra:~/docker-nginx$ k get node -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane   12d   v1.27.4   192.168.49.2   <none>        Ubuntu 22.04.2 LTS   5.15.0-87-generic   docker://24.0.4
```
Access the web service via node ip on port 30080
![dockersimple](https://github.com/lyoben/docker_nginx_simplehtml/assets/81006481/b586a787-fdb2-44ed-9013-04f6ef9c0b13)


