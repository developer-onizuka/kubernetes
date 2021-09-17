
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



```
