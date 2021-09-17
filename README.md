# 0. Create one master-node and two worker-nodes in the cluster
https://github.com/developer-onizuka/gpu-operator3

|  | CPU | Memory | GPU | GPU Driver |
| --- | --- | --- | --- | --- |
| Master | 2 | 8,192 MB | no | --- |
| Worker1 | 2 | 8,192 MB | no | --- |
| Worker2 | 2 | 8,192 MB | no | --- |

# 0-1. Workloads
|  | Replicas | ClusterIP | ExternalIP | Storage | How to access from outside |
| --- | --- | --- | --- | --- | --- |
| HAProxy | N/A | N/A | N/A | N/A | Blowse master-node's IP address | 
| Nginx | 2 | resolved DNS | 192.168.1.10 | Ephemeral | --- |
| Employee Web App | 4 | resolved DNS | no | Ephemeral | --- |
| mongoDB | 1 | resolved DNS | no | Persistent | --- |

# 1. git clone this project
```
$ git clone https://github.com/developer-onizuka/kubernetes.git
```

# 2. Create a storage class, PV and PVC for mongoDB
```
$ kubectl apply -f sc.yaml 
storageclass.storage.k8s.io/local-storage created

$ kubectl get sc
NAME            PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
local-storage   kubernetes.io/no-provisioner   Retain          Immediate           false                  11s

$ kubectl apply -f mongo-pvc.yaml 
persistentvolume/mongo-pv created
persistentvolumeclaim/mongo-pvc created

$ kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongo-pvc   Bound    mongo-pv   1Gi        RWO            local-storage   9s

$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    REASON   AGE
mongo-pv   1Gi        RWO            Retain           Bound    default/mongo-pvc   local-storage            11s
```

# 3. Create deployment of mongoDB
```
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
```

# 4. Check if mongoDB is running properly
```
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

$ kubectl exec -it dnsutils -- /bin/bash
root@dnsutils:/# 
root@dnsutils:/# nslookup mongo-test
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	mongo-test.default.svc.cluster.local
Address: 10.98.25.12

root@dnsutils:/# curl mongo-test:27017
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

# 5. Create deployment of "Employee Web app" with 4 repricas
```
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

$ kubectl describe pod employee-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219
```

# 6. Check if Employee web app is running properly
```
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

... snip ...

</body>
</html>
```

# 7. Create Nginx's config files and Configmap
```
$ cat default.conf
upstream proxy.com {
        server employee-test:5001;
}

server {
        listen 80;
        server_name localhost;
        location / {
                root /usr/share/nginx/html;
                index index.html index.htm;
                proxy_pass https://proxy.com;
        }
}

$ kubectl create configmap nginx-config --from-file=default.conf
configmap/nginx-config created
```
# 8. Create depolyment of Nginx with 2 repricas
```
$ kubectl apply -f nginx.yaml 
$ kubectl describe pod nginx-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219

$ kubectl describe services nginx-test
Name:                     nginx-test
Namespace:                default
Labels:                   run=nginx-test
Annotations:              <none>
Selector:                 run=nginx-test
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.108.176.185
IPs:                      10.108.176.185
External IPs:             192.168.1.10
Port:                     http  8080/TCP
TargetPort:               80/TCP
NodePort:                 http  30875/TCP
Endpoints:                192.168.189.73:80,192.168.235.138:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason      Age    From                Message
  ----    ------      ----   ----                -------
  Normal  ExternalIP  3m43s  service-controller  Added: 192.168.1.10
```

# 9. Check each IP
# 9-1. Node's IP address
```
$ kubectl describe nodes| grep -e Hostname -e InternalIP
  InternalIP:  192.168.122.183
  Hostname:    master
  InternalIP:  192.168.122.18
  Hostname:    worker1
  InternalIP:  192.168.122.219
  Hostname:    worker2
```
# 9-2. Endpoint IP address of Nginx's service
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-17%2018-19-24.png

https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-17%2018-19-16.png
```
$ kubectl describe services nginx-test | grep Endpoint
Endpoints:                192.168.189.73:80,192.168.235.138:80
```

# 9-3. External IP address 
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-17%2018-19-33.png
```
$ kubectl get services
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)             AGE
employee-test   ClusterIP      10.97.233.212    <none>         5001/TCP,5000/TCP   3h29m
kubernetes      ClusterIP      10.96.0.1        <none>         443/TCP             4d21h
mongo-test      ClusterIP      10.98.25.12      <none>         27017/TCP           3h55m
nginx-test      LoadBalancer   10.108.176.185   192.168.1.10   8080:30875/TCP      4m26s
```

# 10. Expose Proxy address for outside world
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-17%2018-16-40.png
```
$ sudo docker run -itd --rm --name haproxy -p 80:80 -v $(pwd)/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy:1.8

$ cat haproxy.cfg 
global
    maxconn 256

defaults
    mode http
    timeout client     120000ms
    timeout server     120000ms
    timeout connect      6000ms

listen http-in
    bind *:80
    server proxy-server 192.168.1.10:8080 
   
```
