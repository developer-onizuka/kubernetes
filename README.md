```
# curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
# sudo dpkg -i minikube_latest_amd64.deb
# sudo apt install conntrack
# sudo apt install docker.io
# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
# chmod 775 kubectl 
# sudo mv kubectl /usr/local/bin/

# sudo adduser developer
# sudo groupadd docker
# sudo usermod -aG docker developer
```

```
# su - developer
Password: 

$ minikube start
😄  minikube v1.18.1 on Ubuntu 20.04 (vbox/amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.3 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v4
    ▪ Using image kubernetesui/dashboard:v2.1.0
    ▪ Using image kubernetesui/metrics-scraper:v1.0.4
🌟  Enabled addons: storage-provisioner, default-storageclass, dashboard
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

$ docker ps -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                                  NAMES
34e4788f42ac        gcr.io/k8s-minikube/kicbase:v0.0.18   "/usr/local/bin/entr…"   2 hours ago         Up 58 seconds       127.0.0.1:32772->22/tcp, 127.0.0.1:32771->2376/tcp, 127.0.0.1:32770->5000/tcp, 127.0.0.1:32769->8443/tcp, 127.0.0.1:32768->32443/tcp   minikube

$ docker exec -it $(docker ps -aq) /bin/bash

root@minikube:/# docker images
REPOSITORY                                TAG        IMAGE ID       CREATED         SIZE
k8s.gcr.io/kube-proxy                     v1.20.2    43154ddb57a8   2 months ago    118MB
k8s.gcr.io/kube-controller-manager        v1.20.2    a27166429d98   2 months ago    116MB
k8s.gcr.io/kube-apiserver                 v1.20.2    a8c2fdb8bf76   2 months ago    122MB
k8s.gcr.io/kube-scheduler                 v1.20.2    ed2c44fbdd78   2 months ago    46.4MB
kubernetesui/dashboard                    v2.1.0     9a07b5b4bfac   3 months ago    226MB
gcr.io/k8s-minikube/storage-provisioner   v4         85069258b98a   3 months ago    29.7MB
k8s.gcr.io/etcd                           3.4.13-0   0369cf4303ff   7 months ago    253MB
k8s.gcr.io/coredns                        1.7.0      bfe3a36ebd25   9 months ago    45.2MB
kubernetesui/metrics-scraper              v1.0.4     86262685d9ab   12 months ago   36.9MB
k8s.gcr.io/pause                          3.2        80d28bedfe5d   13 months ago   683kB

root@minikube:/# exit
exit

```

```
$ kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-node created

$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/hello-node-7567d9fdc9-h22jm   1/1     Running   0          86s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   95m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-node   1/1     1            1           86s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-node-7567d9fdc9   1         1         1       86s




$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
service/hello-node exposed

$ minikube service hello-node
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | hello-node |        8080 | http://192.168.49.2:30565 |
|-----------|------------|-------------|---------------------------|
🎉  Opening service default/hello-node in default browser...
👉  http://192.168.49.2:30565

$ minikube service list
|----------------------|---------------------------|--------------|---------------------------|
|      NAMESPACE       |           NAME            | TARGET PORT  |            URL            |
|----------------------|---------------------------|--------------|---------------------------|
| default              | hello-node                |         8080 | http://192.168.49.2:30565 |
| default              | kubernetes                | No node port |
| kube-system          | kube-dns                  | No node port |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |
| kubernetes-dashboard | kubernetes-dashboard      | No node port |
|----------------------|---------------------------|--------------|---------------------------|

$ curl http://192.168.49.2:30565
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.49.2:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=192.168.49.2:30565
user-agent=curl/7.68.0
BODY:
-no body in request-

```

```
$ kubectl delete service hello-node
service "hello-node" deleted

$ kubectl delete deployment hello-node
deployment.apps "hello-node" deleted

$ kubectl get all
NAME                              READY   STATUS        RESTARTS   AGE
pod/hello-node-7567d9fdc9-h22jm   0/1     Terminating   0          8m31s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   102m

$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   102m
```


```
$ kubectl create deployment test-nginx --image=nginx
deployment.apps/test-nginx created

$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/test-nginx-59ffd87f5-9tcrx   1/1     Running   0          11s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   157m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-nginx   1/1     1            1           11s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/test-nginx-59ffd87f5   1         1         1       11s

$ kubectl expose deployment test-nginx --type=LoadBalancer --port=8080
service/test-nginx exposed

$ minikube service test-nginx
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | test-nginx |        8080 | http://192.168.49.2:32126 |
|-----------|------------|-------------|---------------------------|
🎉  Opening service default/test-nginx in default browser...
👉  http://192.168.49.2:32126

$ minikube service list
|----------------------|---------------------------|--------------|---------------------------|
|      NAMESPACE       |           NAME            | TARGET PORT  |            URL            |
|----------------------|---------------------------|--------------|---------------------------|
| default              | kubernetes                | No node port |
| default              | test-nginx                |         8080 | http://192.168.49.2:32126 |
| kube-system          | kube-dns                  | No node port |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |
| kubernetes-dashboard | kubernetes-dashboard      | No node port |
|----------------------|---------------------------|--------------|---------------------------|

$ curl http://192.168.49.2:32126 
curl: (7) Failed to connect to 192.168.49.2 port 32126: Connection refused

$ kubectl delete service test-nginx
service "test-nginx" deleted

$ kubectl expose deployment test-nginx --type=LoadBalancer --port=80
service/test-nginx exposed

$ minikube service test-nginx
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | test-nginx |          80 | http://192.168.49.2:31038 |
|-----------|------------|-------------|---------------------------|
🎉  Opening service default/test-nginx in default browser...
👉  http://192.168.49.2:31038

$ curl http://192.168.49.2:31038
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```
$ kubectl delete service test-nginx
service "test-nginx" deleted

$ kubectl delete deployment test-nginx
deployment.apps "test-nginx" deleted

$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   165m
```


```
$ cat deployment/nginx_v1.14.2.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-nginx
spec:
  selector:
    matchLabels:
      run: test-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: test-nginx
    spec:
      containers:
      - name: test-nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

$ cat service/nginx.yaml 
apiVersion: v1
kind: Service
metadata:
  name: test-nginx
  labels:
    run: test-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: test-nginx
  type: LoadBalancer
  
$ kubectl apply -f deployment/nginx_v1.14.2.yaml 
deployment.apps/test-nginx created

$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/test-nginx-7ccdf46987-m8k6c   1/1     Running   0          8s
pod/test-nginx-7ccdf46987-q5txt   1/1     Running   0          8s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-nginx   2/2     2            2           8s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/test-nginx-7ccdf46987   2         2         2       8s

$ kubectl apply -f service/nginx.yaml 
service/test-nginx created

developer@hisayuki:~/k8s$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/test-nginx-7ccdf46987-m8k6c   1/1     Running   0          25s
pod/test-nginx-7ccdf46987-q5txt   1/1     Running   0          25s

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        20h
service/test-nginx   LoadBalancer   10.96.168.116   <pending>     80:30120/TCP   4s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-nginx   2/2     2            2           25s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/test-nginx-7ccdf46987   2         2         2       25s

$ minikube service list
|----------------------|---------------------------|--------------|---------------------------|
|      NAMESPACE       |           NAME            | TARGET PORT  |            URL            |
|----------------------|---------------------------|--------------|---------------------------|
| default              | kubernetes                | No node port |
| default              | test-nginx                |           80 | http://192.168.49.2:30120 |
| kube-system          | kube-dns                  | No node port |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |
| kubernetes-dashboard | kubernetes-dashboard      | No node port |
|----------------------|---------------------------|--------------|---------------------------|

$ curl http://192.168.49.2:30120
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
```
$ diff nginx_v1.14.2.yaml nginx_v1.16.1.yaml 
17c17
<         image: nginx:1.14.2
---
>         image: nginx:1.16.1

$ kubectl get pods -l run=test-nginx
NAME                          READY   STATUS    RESTARTS   AGE
test-nginx-7ccdf46987-m8k6c   1/1     Running   0          7m4s
test-nginx-7ccdf46987-q5txt   1/1     Running   0          7m4s

$ kubectl apply -f nginx_v1.16.1.yaml 
deployment.apps/test-nginx configured

$ kubectl get pods -l run=test-nginx
NAME                          READY   STATUS              RESTARTS   AGE
test-nginx-6d4f7fd59b-r2pmj   0/1     ContainerCreating   0          3s
test-nginx-7ccdf46987-m8k6c   1/1     Running             0          7m51s
test-nginx-7ccdf46987-q5txt   1/1     Running             0          7m51s

$ kubectl get pods -l run=test-nginx
NAME                          READY   STATUS              RESTARTS   AGE
test-nginx-6d4f7fd59b-l2tmq   0/1     ContainerCreating   0          0s
test-nginx-6d4f7fd59b-r2pmj   1/1     Running             0          24s
test-nginx-7ccdf46987-m8k6c   1/1     Terminating         0          8m12s
test-nginx-7ccdf46987-q5txt   1/1     Running             0          8m12s

$ kubectl get pods -l run=test-nginx
NAME                          READY   STATUS        RESTARTS   AGE
test-nginx-6d4f7fd59b-l2tmq   1/1     Running       0          4s
test-nginx-6d4f7fd59b-r2pmj   1/1     Running       0          28s
test-nginx-7ccdf46987-m8k6c   0/1     Terminating   0          8m16s
test-nginx-7ccdf46987-q5txt   0/1     Terminating   0          8m16s

$ kubectl get pods -l run=test-nginx
NAME                          READY   STATUS    RESTARTS   AGE
test-nginx-6d4f7fd59b-l2tmq   1/1     Running   0          10s
test-nginx-6d4f7fd59b-r2pmj   1/1     Running   0          34s

$ kubectl describe deployment test-nginx
Name:                   test-nginx
Namespace:              default
CreationTimestamp:      Sun, 28 Mar 2021 07:36:02 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               run=test-nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=test-nginx
  Containers:
   test-nginx:
    Image:        nginx:1.16.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   test-nginx-6d4f7fd59b (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  11m    deployment-controller  Scaled up replica set test-nginx-7ccdf46987 to 2
  Normal  ScalingReplicaSet  3m40s  deployment-controller  Scaled up replica set test-nginx-6d4f7fd59b to 1
  Normal  ScalingReplicaSet  3m16s  deployment-controller  Scaled down replica set test-nginx-7ccdf46987 to 1
  Normal  ScalingReplicaSet  3m16s  deployment-controller  Scaled up replica set test-nginx-6d4f7fd59b to 2
  Normal  ScalingReplicaSet  3m14s  deployment-controller  Scaled down replica set test-nginx-7ccdf46987 to 0
```
