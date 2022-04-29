# training_plan 

<img src="plan.png">

## Two tier application deployment 

<img src="appdep.png">

## Database mysql preparation 

<img src="st1.png">

### using secret to create and store db root account password 

```
 kubectl create  secret  generic  ashudb_cred  --from-literal  db_pass="Oracle098#?"    
   --dry-run=client -o yaml 
apiVersion: v1
data:
  db_pass: T3JhY2xlMDk4Iz8=
kind: Secret
metadata:
  creationTimestamp: null
  name: ashudb_cred
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl create  secret  generic  ashudb_cred  --from-literal  db_pass="Oracle098#?"       --dry-run=client -o yaml  >dbcred.yaml 
fire@ashutoshhs-MacBook-Air multi_tier_app % pwd
/Users/fire/Desktop/k8s_app_deploy/multi_tier_app
fire@ashutoshhs-MacBook-Air multi_tier_app % ls
dbcred.yaml

```

### creating pv and secret 

```
kubectl  apply -f  . 
secret/ashudb-cred created
persistentvolume/ashu-pv unchanged
fire@ashutoshhs-MacBook-Air multi_tier_app % ls
dbcred.yaml     nfspv.yaml
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl  get secret 
NAME                  TYPE                                  DATA   AGE
ashudb-cred           Opaque                                1      12s
default-token-zlcdf   kubernetes.io/service-account-token   3      2d17h
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl  get  pv    
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                     STORAGECLASS   REASON   AGE
ashu-pv           2Gi        RWO            Retain           Available                             manual                  44s
muskan-pv         2Gi        RWO            Retain           Available                             manual                  10s
mysql-pv          5Gi        RWO            Retain           Available                             manual                  24s
mysql-pv-volume   20Gi       RWO            Retain           Bound       muthu-ns/mysql-pv-claim   manual                  22h
santosh-pv        2Gi        RWO            Retain           Available   
```

### creating pvc 

```
 kubectl apply -f .
secret/ashudb-cred configured
persistentvolume/ashu-pv unchanged
persistentvolumeclaim/ashu-pvc created
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl  get  pvc
NAME       STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
ashu-pvc   Bound    subbunfs   3Gi        RWO            manual         7s
fire@ashutoshhs-MacBook-Air multi_tier_app % 

```

### time to deploy DB pod using deployment 

```
 kubectl  apply -f . 
deployment.apps/ashudb created
secret/ashudb-cred configured
persistentvolume/ashu-pv unchanged
persistentvolumeclaim/ashu-pvc unchanged
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl  get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
ashudb   1/1     1            1           12s
fire@ashutoshhs-MacBook-Air multi_tier_app % kubectl  get  po   
NAME                     READY   STATUS    RESTARTS   AGE
ashudb-98b8f59fc-gwgkf   1/1     Running   0          16s
fire@ashutoshhs-MacBook-Air multi_tier_app % 


```


