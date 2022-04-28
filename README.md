# training_plan 

<img src="plan.png">

### kubectl cheat sheet 

[cheat_sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### clean up namespace data

```
 kubectl  delete   all --all
pod "ashuapp-868fc9dfdf-6hvhm" deleted
pod "ashuapp-868fc9dfdf-qfrmz" deleted
service "s1" deleted
deployment.apps "ashuapp" deleted
horizontalpodautoscaler.autoscaling "ashuapp" deleted
```


### DNS in k8s 

<img src="dns.png">

### checking dns in k8s 

```
kubectl  get  pods -n kube-system   |   grep -i dns
coredns-64897985d-l872g                   1/1     Running   5 (17h ago)   11d
coredns-64897985d-ld4c9                   1/1     Running   5 (17h ago)   11d
```

## testing DNS in single namespace 

### deploy app using deployment 

```
kubectl  apply -f  deployment.yaml 
deployment.apps/ashuapp created
fire@ashutoshhs-MacBook-Air k8s_app_deploy % kubectl  get deploy 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
ashuapp   1/1     1            1           4s
fire@ashutoshhs-MacBook-Air k8s_app_deploy % kubectl  get  po -owide
NAME                       READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
ashuapp-868fc9dfdf-s678j   1/1     Running   0          11s   192.168.50.216   minion3   <none>           <none>
fire@ashutoshhs-MacBook-Air k8s_app_deploy % 

```

### deploy nodeport / Loadbalancer svc 

```
 kubectl  get  deploy 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
ashuapp   1/1     1            1           3m51s
fire@ashutoshhs-MacBook-Air k8s_app_deploy % kubectl  expose deployment  ashuapp  --type NodePort  --port 1234 --target-port 80 --na
me  ashulb1 
service/ashulb1 exposed
fire@ashutoshhs-MacBook-Air k8s_app_deploy % kubectl  get  svc
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
ashulb1   NodePort   10.96.160.47   <none>        1234:30394/TCP   52s
```
### testing access from Internal 

```
 kubectl  run  webclient --image=busybox --command sleep inf 
pod/webclient created
fire@ashutoshhs-MacBook-Air ~ % kubectl  get  po
NAME                       READY   STATUS    RESTARTS   AGE
ashuapp-868fc9dfdf-s678j   1/1     Running   0          7m30s
webclient                  1/1     Running   0          4s
fire@ashutoshhs-MacBook-Air ~ % 

```

### accessing page 

```
kubectl  exec  -it webclient -- sh 
/ # 
/ # wget  http://ashulb1 
Connecting to ashulb1 (10.96.160.47:80)
^C
/ # wget  http://ashulb1:1234 
Connecting to ashulb1:1234 (10.96.160.47:1234)
saving to 'index.html'
index.html           100% |****************************************************************************|  2866  0:00:00 ETA
'index.html' saved

```

### accessing svc from different namespace pod 

```

fire@ashutoshhs-MacBook-Air ~ % kubectl  run  webclient --image=busybox --command sleep inf  -n app-access 
pod/webclient created
fire@ashutoshhs-MacBook-Air ~ % 
fire@ashutoshhs-MacBook-Air ~ % kubectl  -n app-access exec -it  webclient -- sh 
/ # 
/ # wget http://ashulb
wget: bad address 'ashulb'
/ # wget http://ashulb1
wget: bad address 'ashulb1'
/ # 
/ # wget http://ashulb1
wget: bad address 'ashulb1'
/ # wget http://ashulb1.ashu-oci.svc.cluster.local
Connecting to ashulb1.ashu-oci.svc.cluster.local (10.96.116.145:80)
saving to 'index.html'
index.html           100% |****************************************************************************|  2866  0:00:00 ETA
'index.html' saved
/ # 
/ # cat  /etc/resolv.conf 
nameserver 10.96.0.10
search app-access.svc.cluster.local svc.cluster.local cluster.local ec2.internal
options ndots:5

```

### POD dns demo 

```
kubectl  -n app-access exec -it  webclient -- sh 

/ # 
/ # ping  192.168.50.216
PING 192.168.50.216 (192.168.50.216): 56 data bytes
64 bytes from 192.168.50.216: seq=0 ttl=63 time=0.540 ms
64 bytes from 192.168.50.216: seq=1 ttl=63 time=0.060 ms
^C
--- 192.168.50.216 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.060/0.300/0.540 ms
/ # 
/ # 
/ # ping  192-168-50-216.ashu-oci.pod.cluster.local
PING 192-168-50-216.ashu-oci.pod.cluster.local (192.168.50.216): 56 data bytes
64 bytes from 192.168.50.216: seq=0 ttl=63 time=0.061 ms
64 bytes from 192.168.50.216: seq=1 ttl=63 time=0.081 ms
^C
--- 192-168-50-216.ashu-oci.pod.cluster.local ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.061/0.071/0.081 ms

```

### Dashboard deployment in k8s 

<img src="dash.png">

### Deploy 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

### Verify dashboard 

```
kubectl  get deploy -n kubernetes-dashboard 
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
dashboard-metrics-scraper   1/1     1            1           82s
kubernetes-dashboard        1/1     1            1           83s
fire@ashutoshhs-MacBook-Air ~ % kubectl  get  po  -n kubernetes-dashboard 
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-799d786dbf-djhzm   1/1     Running   0          96s
kubernetes-dashboard-546cbc58cd-gv2jj        1/1     Running   0          97s
fire@ashutoshhs-MacBook-Air ~ % 
fire@ashutoshhs-MacBook-Air ~ % kubectl  get  svc  -n kubernetes-dashboard 
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
dashboard-metrics-scraper   ClusterIP   10.109.105.22   <none>        8000/TCP   106s
kubernetes-dashboard        ClusterIP   10.96.166.40    <none>        443/TCP    112s
fire@ashutoshhs-MacBook-Air ~ % kubectl  get  ep  -n kubernetes-dashboard 
NAME                        ENDPOINTS              AGE
dashboard-metrics-scraper   192.168.235.130:8000   113s
kubernetes-dashboard        192.168.235.129:8443   119s
fire@ashutoshhs-MacBook-Air ~ % 

```

### to access dashboard from outside k8s cluster we need to change service to NP / LB 

```
 kubectl edit  svc  kubernetes-dashboard   -n kubernetes-dashboard 
service/kubernetes-dashboard edited
fire@ashutoshhs-MacBook-Air ~ % 
fire@ashutoshhs-MacBook-Air ~ % kubectl  get  svc  -n kubernetes-dashboard                        
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.109.105.22   <none>        8000/TCP        3m50s
kubernetes-dashboard        NodePort    10.96.166.40    <none>        443:32479/TCP   3m56s
fire@ashutoshhs-MacBook-Air ~ % 


```

### get dashboard token using given method 

```
 kubectl  get  secret  -n kubernetes-dashboard 
NAME                               TYPE                                  DATA   AGE
default-token-282zv                kubernetes.io/service-account-token   3      25m
kubernetes-dashboard-certs         Opaque                                0      25m
kubernetes-dashboard-csrf          Opaque                                1      25m
kubernetes-dashboard-key-holder    Opaque                                2      25m
kubernetes-dashboard-token-km56d   kubernetes.io/service-account-token   3      25m
fire@ashutoshhs-MacBook-Air ~ % kubectl describe  secret kubernetes-dashboard-token-km56d   -n kubernetes-dashboard 
Name:         kubernetes-dashboard-token-km56d
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 55692ce7-0fe8-43f9-90b8-2cf13aea4198

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1099 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ikc4Zmp5VERyUzFLRWpVMkdMMXBGM0haRXFNX2VYNGl4X1BzaGhzRFNySWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdW
```

### give access permission to dashbaord 

```
kubectl create clusterrolebinding power --clusterrole=cluster-admin  --serviceaccount=kubernetes-dashboard:kubernetes-dashboard
```

