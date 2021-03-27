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
