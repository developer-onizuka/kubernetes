# 0. Create one master-node and two worker-nodes in the cluster from scratch or using vagrant below
https://github.com/developer-onizuka/kubernetes_vagrant

|  | CPU | Memory | GPU | GPU Driver |
| --- | --- | --- | --- | --- |
| Master | 2 | 8,192 MB | no | N/A |
| Worker1 | 2 | 8,192 MB | no | N/A |
| Worker2 | 2 | 8,192 MB | no | N/A |

# 0-1. Workloads
|  | Replicas | ClusterIP | ExternalIP | Storage | How to access from outside |
| --- | --- | --- | --- | --- | --- |
| HAProxy | N/A | N/A | N/A | N/A | Blowse Host's IP address | 
| Nginx | 2 | resolved DNS | See #10 | Ephemeral | N/A |
| Employee Web App | 4 | resolved DNS | no | Ephemeral | N/A |
| mongoDB | 1 | resolved DNS | no | Persistent | N/A |

```
Type=NodePort
       <HAproxy>                      <Service>                  <Pod>
                                      
                                      +----master----+           +----maste---------+
                      192.168.122.183:30001          |           |                  | 
iPhone,etc                    +------>| nginx-srv    +-----+     |                  |
  |                           |       |              |     |     |                  |
  |    +---------+            |       +--------------+     |     +------------------+
  |    |         +------------+       +----worker2---+     |     +----worker1-------+
  |    |         |     192.168.122.18:30001          |     |     |                  |
  +--->| HAProxy +------------------->| nginx-srv    +-----+     | employee-test x3 |
       |         |                    |              |     |     | mongo-test       |
       |         +------------+       +--------------+     |     +------------------+
       +---------+            |       +----worker2---+     |     +----worker2-------+
       192.168.11.27          |       |              |     |     |                  |
                              +------>| nginx-srv    +-----+---->| nginx-test x2    | 192.168.189.127:80, 192.168.189.65:80
                      192.168.122.219:30001          |           | employee-test x1 |
                                      +--------------+           +------------------+

               <=======================>           <=============>

  [haproxy-nodeport.cfg]                           [nginx-nodeport.yaml]
  server proxy-server1 192.168.122.183:30001         kind: Service
  server proxy-server2 192.168.122.18:30001          metadata:
  server proxy-server3 192.168.122.219:30001           name: nginx-srv
                                                       labels:
                                                         run: nginx-srv
                                                     spec:
                                                       type: NodePort
                                                        ports:
                                                        - port: 8080
                                                          targetPort: 80
                                                          protocol: TCP
                                                          name: http
                                                          nodePort: 30001
                                                        selector:
                                                          run: nginx-test
```

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

$ $ kubectl describe pods mongo-test
Name:         mongo-test-67f5dd84b7-bpn4w
Namespace:    default
Priority:     0
Node:         worker1/192.168.122.18
Start Time:   Tue, 21 Sep 2021 23:49:03 +0900
Labels:       pod-template-hash=67f5dd84b7
              run=mongo-test
Annotations:  cni.projectcalico.org/containerID: 5083c61ec3035aad1c03f87344309416ab6802502453fb0cad15b3a49a3392a1
              cni.projectcalico.org/podIP: 192.168.235.183/32
              cni.projectcalico.org/podIPs: 192.168.235.183/32
Status:       Running
IP:           192.168.235.183
IPs:
  IP:           192.168.235.183
Controlled By:  ReplicaSet/mongo-test-67f5dd84b7
Containers:
  mongodb:
    Container ID:   docker://71fdfd1de68a1835d8cfe2c1b518fcf9daa821c4aa6c399be0371bb436594118
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:ae1ae34d5e470af7b1a5ca26565e913eec5f9f645701036e6d6860dfcacfa116
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 22 Sep 2021 08:07:49 +0900
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Sep 2021 23:49:08 +0900
      Finished:     Wed, 22 Sep 2021 00:06:46 +0900
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /data/db from mongo-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dgcrd (ro)
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
  kube-api-access-dgcrd:
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
  Type    Reason          Age                From               Message
  ----    ------          ----               ----               -------
  Normal  Scheduled       8h                 default-scheduler  Successfully assigned default/mongo-test-67f5dd84b7-bpn4w to worker1
  Normal  Pulling         8h                 kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled          8h                 kubelet            Successfully pulled image "docker.io/mongo" in 2.694415017s
  Normal  Created         8h                 kubelet            Created container mongodb
  Normal  Started         8h                 kubelet            Started container mongodb
  Normal  SandboxChanged  11m (x2 over 12m)  kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal  Pulling         11m                kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled          11m                kubelet            Successfully pulled image "docker.io/mongo" in 3.865717695s
  Normal  Created         11m                kubelet            Created container mongodb
  Normal  Started         11m                kubelet            Started container mongodb
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

root@dnsutils:/# nslookup mongo-test
Server:		10.96.0.10
Address:	10.96.0.10#53

** server can't find mongo-test: NXDOMAIN

root@dnsutils:/# curl mongo-test:27017
curl: (6) Could not resolve host: mongo-test

root@dnsutils:/# nslookup mongo-srv 
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	mongo-srv.default.svc.cluster.local
Address: 10.110.87.184

root@dnsutils:/# curl mongo-srv:27017
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

$ kubectl describe pod employee-test |grep ^Node:
Node:         worker1/192.168.122.18
Node:         worker2/192.168.122.219
Node:         worker1/192.168.122.18
Node:         worker1/192.168.122.18
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
$ cat <<EOF > default.conf
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
EOF

$ kubectl create configmap nginx-config --from-file=default.conf
configmap/nginx-config created
```
# 8. Create depolyment of Nginx with 2 repricas
```
$ kubectl apply -f nginx-nodeport.yaml 

$ kubectl describe pod nginx-test |grep ^Node:
Node:         worker2/192.168.122.219
Node:         worker2/192.168.122.219

$ $ kubectl describe services nginx-srv
Name:                     nginx-srv
Namespace:                default
Labels:                   run=nginx-srv
Annotations:              <none>
Selector:                 run=nginx-test
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.103.147.112
IPs:                      10.103.147.112
Port:                     http  8080/TCP
TargetPort:               80/TCP
NodePort:                 http  30001/TCP
Endpoints:                192.168.189.127:80,192.168.189.65:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
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
# 9-2. Cluster IP
```
$ kubectl exec -it dnsutils -- /bin/bash
root@dnsutils:/# nslookup nginx-srv
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	nginx-srv.default.svc.cluster.local
Address: 10.103.147.112

root@dnsutils:/# curl nginx-srv:80
curl: (7) Failed to connect to nginx-srv port 80: Connection timed out

root@dnsutils:/# curl nginx-srv:8080  
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title> - Employee</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>

... snip ...

</body>
</html>
```

# 9-3. Endpoint IP address of Nginx's service ( = Pod's IP address)
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-22%2008-37-18.png

https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-22%2008-37-26.png
```
$ kubectl describe services nginx-srv | grep Endpoint
Endpoints:                192.168.189.127:80,192.168.189.65:80
```
```
$ kubectl exec -it nginx-test-85c6647877-627k7 -- hostname -i
192.168.189.127
$ kubectl exec -it nginx-test-85c6647877-wwqvz -- hostname -i
192.168.189.65
```

# 9-4. NodePort IP address and ports ( = Node's IP address + 30001)

192.168.122.183:30001 --> 
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-21%2009-51-21.png

192.168.122.18:30001 --> 
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-21%2009-51-29.png

192.168.122.219:30001 --> 
https://github.com/developer-onizuka/kubernetes/blob/main/Screenshot%20from%202021-09-21%2009-51-35.png
```
$ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
employee-srv   ClusterIP   10.102.92.11     <none>        5001/TCP,5000/TCP   8h
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP             8h
mongo-srv      ClusterIP   10.110.87.184    <none>        27017/TCP           8h
nginx-srv      NodePort    10.103.147.112   <none>        8080:30001/TCP      8h
```

# 10. Expose Proxy address for outside world using HAproxy on Host Machine
Docker.io should be installed on Host Machine, prior to this step.

The following picture is taken on my smart phone involved in same network as Host Machine (192.168.11.xx).

https://github.com/developer-onizuka/kubernetes/blob/main/image_123986672.JPG

```
$ cat <<EOF > haproxy-nodeport.cfg 
global
    maxconn 256

defaults
    mode http
    timeout client     120000ms
    timeout server     120000ms
    timeout connect      6000ms

listen http-in
    bind *:80
    server proxy-server1 192.168.122.183:30001 
    server proxy-server2 192.168.122.18:30001 
    server proxy-server3 192.168.122.219:30001
EOF
$ sudo docker run -itd --rm --name haproxy -p 80:80 -v $(pwd)/haproxy-nodeport.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy:1.8
```

# 11. Dash board
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
$ kubectl proxy
```
You need the token to sign in dash board.
```
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep deploy |awk '{print $1}')
```
Web access to followings:
-----
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=default

