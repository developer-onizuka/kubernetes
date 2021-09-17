
```
$ mkdir containers
$ cd containers/

$ cat pvc.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ubuntu-pv
spec:
  storageClassName: local-storage
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/data"
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ubuntu-pvc
spec:
  storageClassName: local-storage
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi


$ kubectl apply -f sc.yaml 
storageclass.storage.k8s.io/local-storage created


$ kubectl get sc
NAME            PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
local-storage   kubernetes.io/no-provisioner   Retain          Immediate           false                  11s

$ ls
mongo-pvc.yaml  mongo.yaml  nginx-pvc.yaml  sc.yaml

$ kubectl apply -f mongo-pvc.yaml 
persistentvolume/mongo-pv created
persistentvolumeclaim/mongo-pvc created

$ kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongo-pvc   Bound    mongo-pv   1Gi        RWO            local-storage   9s

$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    REASON   AGE
mongo-pv   1Gi        RWO            Retain           Bound    default/mongo-pvc   local-storage            11s

$ kubectl apply -f mongo.yaml 
service/mongo-test created
deployment.apps/mongo-test created

$ kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
mongo-test-67f5dd84b7-4rssk   0/1     ContainerCreating   0          8s

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
mongo-test-67f5dd84b7-4rssk   1/1     Running   0          84s

$ kubectl get services
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP     4d17h
mongo-test   ClusterIP   10.98.25.12   <none>        27017/TCP   99s

$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  labels:
    name: dnsutils
spec:
  containers:
  - name: dnsutils
    image: tutum/dnsutils
    command:
    - sleep
    - "3600"
EOF
pod/dnsutils created

$ kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
dnsutils                      0/1     ContainerCreating   0          7s
mongo-test-67f5dd84b7-4rssk   1/1     Running             0          3m26s

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
dnsutils                      1/1     Running   0          93s
mongo-test-67f5dd84b7-4rssk   1/1     Running   0          4m52s

$ kubectl describe pods |grep ^Node:
Node:         worker2/192.168.122.219
Node:         worker2/192.168.122.219

$ kubectl describe pods mongo-test-67f5dd84b7-4rssk
Name:         mongo-test-67f5dd84b7-4rssk
Namespace:    default
Priority:     0
Node:         worker2/192.168.122.219
Start Time:   Fri, 17 Sep 2021 14:12:54 +0900
Labels:       pod-template-hash=67f5dd84b7
              run=mongo-test
Annotations:  cni.projectcalico.org/containerID: 62146e8b4331d812e0963a8642b6708ed69f4f358c203936ecb50c48cbf18012
              cni.projectcalico.org/podIP: 192.168.189.66/32
              cni.projectcalico.org/podIPs: 192.168.189.66/32
Status:       Running
IP:           192.168.189.66
IPs:
  IP:           192.168.189.66
Controlled By:  ReplicaSet/mongo-test-67f5dd84b7
Containers:
  mongodb:
    Container ID:   docker://4a1d737a5f82db4ccd132ef06fa7c2acaa0cf5a17cf7be5b44018f2716570c67
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:58ea1bc09f269a9b85b7e1fae83b7505952aaa521afaaca4131f558955743842
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 17 Sep 2021 14:14:16 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data/db from mongo-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lprzv (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  mongo-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mongo-pvc
    ReadOnly:   false
  kube-api-access-lprzv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  6m14s  default-scheduler  Successfully assigned default/mongo-test-67f5dd84b7-4rssk to worker2
  Normal  Pulling    6m13s  kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled     4m56s  kubelet            Successfully pulled image "docker.io/mongo" in 1m16.668285964s
  Normal  Created    4m52s  kubelet            Created container mongodb
  Normal  Started    4m52s  kubelet            Started container mongodb
  
$ kubectl describe pods dnsutils
Name:         dnsutils
Namespace:    default
Priority:     0
Node:         worker2/192.168.122.219
Start Time:   Fri, 17 Sep 2021 14:16:13 +0900
Labels:       name=dnsutils
Annotations:  cni.projectcalico.org/containerID: 4ec98edc076b0de1cdc3403032a1b09573ed6eb1daeec510abca651942ecda2f
              cni.projectcalico.org/podIP: 192.168.189.67/32
              cni.projectcalico.org/podIPs: 192.168.189.67/32
Status:       Running
IP:           192.168.189.67
IPs:
  IP:  192.168.189.67
Containers:
  dnsutils:
    Container ID:  docker://70787f2f5adcf52ae8f88bfb5b850f437595e819cd7390b10f130980812c4d32
    Image:         tutum/dnsutils
    Image ID:      docker-pullable://tutum/dnsutils@sha256:d2244ad47219529f1003bd1513f5c99e71655353a3a63624ea9cb19f8393d5fe
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Fri, 17 Sep 2021 14:16:43 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vbs76 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-vbs76:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m23s  default-scheduler  Successfully assigned default/dnsutils to worker2
  Normal  Pulling    3m22s  kubelet            Pulling image "tutum/dnsutils"
  Normal  Pulled     2m54s  kubelet            Successfully pulled image "tutum/dnsutils" in 28.486919248s
  Normal  Created    2m53s  kubelet            Created container dnsutils
  Normal  Started    2m53s  kubelet            Started container dnsutils


$ kubectl exec -it dnsutils -- /bin/bash
root@dnsutils:/# 
root@dnsutils:/# nslookup mongo-test
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	mongo-test.default.svc.cluster.local
Address: 10.98.25.12

root@dnsutils:/# curl mongo-test:27017
It looks like you are trying to access MongoDB over HTTP on the native driver port.


$ kubectl apply -f employee.yaml 
service/employee-test created
deployment.apps/employee-test created

$ kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
dnsutils                         1/1     Running             0          22m
employee-test-85cf649cf6-4mzp6   0/1     ContainerCreating   0          10s
employee-test-85cf649cf6-7mjdc   0/1     ContainerCreating   0          10s
employee-test-85cf649cf6-hx2jh   0/1     ContainerCreating   0          10s
employee-test-85cf649cf6-hz9hz   0/1     ContainerCreating   0          10s
mongo-test-67f5dd84b7-4rssk      1/1     Running             0          25m

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
dnsutils                         1/1     Running   0          24m
employee-test-85cf649cf6-4mzp6   1/1     Running   0          2m30s
employee-test-85cf649cf6-7mjdc   1/1     Running   0          2m30s
employee-test-85cf649cf6-hx2jh   1/1     Running   0          2m30s
employee-test-85cf649cf6-hz9hz   1/1     Running   0          2m30s
mongo-test-67f5dd84b7-4rssk      1/1     Running   0          28m

$ kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
employee-test   ClusterIP   10.97.233.212   <none>        5001/TCP,5000/TCP   2m40s
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP             4d18h
mongo-test      ClusterIP   10.98.25.12     <none>        27017/TCP           28m



$ kubectl describe pod employee-test
Name:         employee-test-85cf649cf6-4mzp6
Namespace:    default
Priority:     0
Node:         worker1/192.168.122.18
Start Time:   Fri, 17 Sep 2021 14:38:38 +0900
Labels:       pod-template-hash=85cf649cf6
              run=employee-test
Annotations:  cni.projectcalico.org/containerID: adef28e082b6fcaaf7b6052f30811303178ef6c2e1d9326179bd9ab782c2a778
              cni.projectcalico.org/podIP: 192.168.235.135/32
              cni.projectcalico.org/podIPs: 192.168.235.135/32
Status:       Running
IP:           192.168.235.135
IPs:
  IP:           192.168.235.135
Controlled By:  ReplicaSet/employee-test-85cf649cf6
Containers:
  employee:
    Container ID:  docker://7623a1690d6c456f6c8bb2c4ff53346d8dacd46d5f6824dad266f3bd510d28de
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Fri, 17 Sep 2021 14:40:56 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  mongo-test
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jshhm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-jshhm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/employee-test-85cf649cf6-4mzp6 to worker1
  Normal  Pulling    3m17s  kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     61s    kubelet            Successfully pulled image "developeronizuka/employee" in 2m15.89478361s
  Normal  Created    61s    kubelet            Created container employee
  Normal  Started    61s    kubelet            Started container employee

Name:         employee-test-85cf649cf6-7mjdc
Namespace:    default
Priority:     0
Node:         worker2/192.168.122.219
Start Time:   Fri, 17 Sep 2021 14:38:38 +0900
Labels:       pod-template-hash=85cf649cf6
              run=employee-test
Annotations:  cni.projectcalico.org/containerID: 0f236a7d9afc40259d91b01fab9f69300ce6e058ac38c88cb75a12cee259f440
              cni.projectcalico.org/podIP: 192.168.189.69/32
              cni.projectcalico.org/podIPs: 192.168.189.69/32
Status:       Running
IP:           192.168.189.69
IPs:
  IP:           192.168.189.69
Controlled By:  ReplicaSet/employee-test-85cf649cf6
Containers:
  employee:
    Container ID:  docker://8d22b300e1ba8c95a5b8abae646721628b614d07d7b333b673872cf1003cfa2b
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Fri, 17 Sep 2021 14:40:56 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  mongo-test
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-szstm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-szstm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/employee-test-85cf649cf6-7mjdc to worker2
  Normal  Pulling    3m17s  kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     61s    kubelet            Successfully pulled image "developeronizuka/employee" in 2m16.389641627s
  Normal  Created    61s    kubelet            Created container employee
  Normal  Started    61s    kubelet            Started container employee

Name:         employee-test-85cf649cf6-hx2jh
Namespace:    default
Priority:     0
Node:         worker1/192.168.122.18
Start Time:   Fri, 17 Sep 2021 14:38:38 +0900
Labels:       pod-template-hash=85cf649cf6
              run=employee-test
Annotations:  cni.projectcalico.org/containerID: 3538000fe1ff42d065b8632e6cf7c5012f6f3fe6dbe66fcbc68d0adba3a084ef
              cni.projectcalico.org/podIP: 192.168.235.136/32
              cni.projectcalico.org/podIPs: 192.168.235.136/32
Status:       Running
IP:           192.168.235.136
IPs:
  IP:           192.168.235.136
Controlled By:  ReplicaSet/employee-test-85cf649cf6
Containers:
  employee:
    Container ID:  docker://22fc9172fa7ca34f2cb03e73f06fbca0d2f99d8c7a180a9350dd7d1f53ea44d0
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Fri, 17 Sep 2021 14:40:58 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  mongo-test
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lc7z8 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-lc7z8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/employee-test-85cf649cf6-hx2jh to worker1
  Normal  Pulling    3m17s  kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     59s    kubelet            Successfully pulled image "developeronizuka/employee" in 2m18.299517584s
  Normal  Created    59s    kubelet            Created container employee
  Normal  Started    59s    kubelet            Started container employee

Name:         employee-test-85cf649cf6-hz9hz
Namespace:    default
Priority:     0
Node:         worker2/192.168.122.219
Start Time:   Fri, 17 Sep 2021 14:38:38 +0900
Labels:       pod-template-hash=85cf649cf6
              run=employee-test
Annotations:  cni.projectcalico.org/containerID: 5389aa976424fd025fb5e9af52f542c0903dc12a9d1cb071e7e3fc54efc2a4b1
              cni.projectcalico.org/podIP: 192.168.189.68/32
              cni.projectcalico.org/podIPs: 192.168.189.68/32
Status:       Running
IP:           192.168.189.68
IPs:
  IP:           192.168.189.68
Controlled By:  ReplicaSet/employee-test-85cf649cf6
Containers:
  employee:
    Container ID:  docker://d6c9ffa923e6cdc59153f4f39aff973faeab99dd200d54d19dab6047c22b126e
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Fri, 17 Sep 2021 14:40:54 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  mongo-test
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dpkbm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-dpkbm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m19s  default-scheduler  Successfully assigned default/employee-test-85cf649cf6-hz9hz to worker2
  Normal  Pulling    3m17s  kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     64s    kubelet            Successfully pulled image "developeronizuka/employee" in 2m13.755453227s
  Normal  Created    63s    kubelet            Created container employee
  Normal  Started    63s    kubelet            Started container employee

$ kubectl describe pod employee-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219


root@dnsutils:/# curl https://employee-test:5001 -k
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title> - Employee</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" href="/">Employee</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/Home/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            
<h1>List of Employees</h1>

<h2></h2>

<a href="/Home/Insert"> Add New Employee</a>

<br /><br />


<table border="1" cellpadding="10">
</table>

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2020 - Employee - <a href="/Home/Privacy">Privacy</a>
        </div>
    </footer>
    <script src="/lib/jquery/dist/jquery.min.js"></script>
    <script src="/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/site.js"></script>
    
</body>
</html>
root@dnsutils:/#


<Worker1 and Worker2>
$ sudo mkdir -p /var/data/nginx
$ sudo vi /var/data/nginx/default.conf


$ kubectl apply -f nginx.yaml 
$ kubectl describe pod nginx-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219

$ kubectl get services
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
employee-test   ClusterIP      10.97.233.212   <none>        5001/TCP,5000/TCP   18m
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP             4d18h
mongo-test      ClusterIP      10.98.25.12     <none>        27017/TCP           43m
nginx-test      LoadBalancer   10.96.221.84    <pending>     8080:31241/TCP      2m19s

$ kubectl describe services nginx-test
Name:                     nginx-test
Namespace:                default
Labels:                   run=nginx-test
Annotations:              <none>
Selector:                 run=nginx-test
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.221.84
IPs:                      10.96.221.84
Port:                     http  8080/TCP
TargetPort:               80/TCP
NodePort:                 http  31241/TCP
Endpoints:                192.168.189.70:80,192.168.235.137:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>


$ kubectl describe pod nginx-test
Name:         nginx-test-6998749496-dfqjc
Namespace:    default
Priority:     0
Node:         worker1/192.168.122.18
Start Time:   Fri, 17 Sep 2021 14:54:28 +0900
Labels:       pod-template-hash=6998749496
              run=nginx-test
Annotations:  cni.projectcalico.org/containerID: 7bd697e03b07a78073354fd49eb0387d99e8fd845ba10028e93c956f2a20c1ec
              cni.projectcalico.org/podIP: 192.168.235.137/32
              cni.projectcalico.org/podIPs: 192.168.235.137/32
Status:       Running
IP:           192.168.235.137
IPs:
  IP:           192.168.235.137
Controlled By:  ReplicaSet/nginx-test-6998749496
Containers:
  nginx:
    Container ID:   docker://c50c077525f3c5c0f97d74fc21dd2d56fc53436a6e4971b4a7dda631ce03efd0
    Image:          nginx:1.14.2
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 17 Sep 2021 14:55:04 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/nginx/conf.d from nginx-conf (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tkngj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  nginx-conf:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  nginx-pvc
    ReadOnly:   false
  kube-api-access-tkngj:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  92s   default-scheduler  Successfully assigned default/nginx-test-6998749496-dfqjc to worker1
  Normal  Pulling    90s   kubelet            Pulling image "nginx:1.14.2"
  Normal  Pulled     57s   kubelet            Successfully pulled image "nginx:1.14.2" in 33.521338323s
  Normal  Created    57s   kubelet            Created container nginx
  Normal  Started    56s   kubelet            Started container nginx

Name:         nginx-test-6998749496-w8g7l
Namespace:    default
Priority:     0
Node:         worker2/192.168.122.219
Start Time:   Fri, 17 Sep 2021 14:54:28 +0900
Labels:       pod-template-hash=6998749496
              run=nginx-test
Annotations:  cni.projectcalico.org/containerID: 8263f5e79d09c1455ac89aa442a5daf23428bf2ebed28e20164801a2ae8d07f1
              cni.projectcalico.org/podIP: 192.168.189.70/32
              cni.projectcalico.org/podIPs: 192.168.189.70/32
Status:       Running
IP:           192.168.189.70
IPs:
  IP:           192.168.189.70
Controlled By:  ReplicaSet/nginx-test-6998749496
Containers:
  nginx:
    Container ID:   docker://f328b95290aee85a1ea845f3ac12f2c67d15d56770cb06716260983970b4adf0
    Image:          nginx:1.14.2
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 17 Sep 2021 14:55:03 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/nginx/conf.d from nginx-conf (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7f2qt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  nginx-conf:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  nginx-pvc
    ReadOnly:   false
  kube-api-access-7f2qt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  92s   default-scheduler  Successfully assigned default/nginx-test-6998749496-w8g7l to worker2
  Normal  Pulling    91s   kubelet            Pulling image "nginx:1.14.2"
  Normal  Pulled     58s   kubelet            Successfully pulled image "nginx:1.14.2" in 32.922793976s
  Normal  Created    57s   kubelet            Created container nginx
  Normal  Started    57s   kubelet            Started container nginx

$ kubectl describe pod nginx-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219


$ kubectl describe nodes| grep -e Hostname -e InternalIP
  InternalIP:  192.168.122.183
  Hostname:    master
  InternalIP:  192.168.122.18
  Hostname:    worker1
  InternalIP:  192.168.122.219
  Hostname:    worker2

$ kubectl describe services nginx-test | grep Endpoint
Endpoints:                192.168.189.70:80,192.168.235.137:80


$ kubectl get services
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
employee-test   ClusterIP      10.97.233.212   <none>        5001/TCP,5000/TCP   104m
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP             4d19h
mongo-test      ClusterIP      10.98.25.12     <none>        27017/TCP           130m
nginx-test      LoadBalancer   10.96.221.84    <pending>     8080:31241/TCP      89m



```
