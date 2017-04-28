# Install webserver on K8S
- Tạo webserver chạy trên docker với k8s.
- Thử nghiệm các tính năng:
  - scale, load balacing.
  - Update images cho deployments.
  - roll back.

# 1. Deploy apache2.
```sh
root@master:/opt/demo# kubectl create -f apache2.yaml
```

- Check deployments, pods, services
```sh
root@master:/opt/demo# kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
apache2   1         1         1            1           9m


root@master:/opt/demo# kubectl describe deployments apache2
Name:                   apache2
Namespace:              default
CreationTimestamp:      Thu, 27 Apr 2017 16:23:03 +0700
Labels:                 name=apache2
Annotations:            deployment.kubernetes.io/revision=1
Selector:               name=apache2
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:       name=apache2
  Containers:
   apache2:
    Image:              cosy294/swarm:1.0
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
OldReplicaSets: <none>
NewReplicaSet:  apache2-4181921630 (1/1 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath   Type        Reason                   Message
  ---------     --------        -----   ----                    -------------   --------    ------                   -------
  9m            9m              1       deployment-controller                   Normal      ScalingReplicaSet        Scaled up replica set apache2-4181921630 to 1
```


```sh
root@master:/opt/demo# kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
apache2-4181921630-w25sw   1/1       Running   0          10m




root@master:/opt/demo# kubectl describe pods apache2-4181921630-w25sw 
Name:           apache2-4181921630-w25sw
Namespace:      default
Node:           minion2/172.16.69.216
Start Time:     Thu, 27 Apr 2017 16:23:03 +0700
Labels:         name=apache2
                pod-template-hash=4181921630
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"apache2-4181921630","uid":"1d32f6e3-2b2b-11e7-b9b0-52540006d9c8"...
Status:         Running
IP:             10.39.0.2
Controllers:    ReplicaSet/apache2-4181921630
Containers:
  apache2:
    Container ID:       docker://2f0fb258f5f2e4339b50340370984a06d84886e0112ff8392f98ae26f05f8a12
    Image:              cosy294/swarm:1.0
    Image ID:           docker-pullable://cosy294/swarm@sha256:4b5c51401edc0af2ea93989aee304baf6d712b15f75e4fec465648c3e8111ff9
    Port:               80/TCP
    State:              Running
      Started:          Thu, 27 Apr 2017 16:23:05 +0700
    Ready:              True
    Restart Count:      0
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dhj1p (ro)
Conditions:
  Type          Status
  Initialized   True 
  Ready         True 
  PodScheduled  True 
Volumes:
  default-token-dhj1p:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-dhj1p
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: <none>
Tolerations:    node.alpha.kubernetes.io/notReady=:Exists:NoExecute for 300s
                node.alpha.kubernetes.io/unreachable=:Exists:NoExecute for 300s
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath               Type             Reason          Message
  ---------     --------        -----   ----                    -------------               -------- ------          -------
  10m           10m             1       default-scheduler                                   Normal           Scheduled       Successfully assigned apache2-4181921630-w25sw to minion2
  10m           10m             1       kubelet, minion2        spec.containers{apache2}    Normal           Pulled          Container image "cosy294/swarm:1.0" already present on machine
  10m           10m             1       kubelet, minion2        spec.containers{apache2}    Normal           Created         Created container with id 2f0fb258f5f2e4339b50340370984a06d84886e0112ff8392f98ae26f05f8a12
  10m           10m             1       kubelet, minion2        spec.containers{apache2}    Normal           Started         Started container with id 2f0fb258f5f2e4339b50340370984a06d84886e0112ff8392f98ae26f05f8a12
root@master:/opt/demo# 

```


```sh
root@master:/opt/demo# kubectl get svc
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
apache2      10.97.115.115   <nodes>       5555:30815/TCP   11m
kubernetes   10.96.0.1       <none>        443/TCP          8d



root@master:/opt/demo# kubectl describe svc apache2
Name:                   apache2
Namespace:              default
Labels:                 <none>
Annotations:            <none>
Selector:               name=apache2
Type:                   NodePort
IP:                     10.97.115.115
Port:                   <unset> 5555/TCP
NodePort:               <unset> 30815/TCP
Endpoints:              10.39.0.2:80
Session Affinity:       None
Events:                 <none>
root@master:/opt/demo# 
```


- Các thông tin quan trọng cần lưu ý:
   - 10.39.0.2:80: Địa chỉ ip endpoint, tức là ip container sẽ chạy apache2 và cổng của service (apache2) chạy trên container này.
   - 10.97.115.115: Địa chỉ ip của cụm cluster apache2.
   - Port: 5555: Cổng truy cập vào services của cụm cluster.
   - NodePort: Cổng truy cập vào service trên bất cứ node nào của cụm cluster.


- Access webserver: 
Khi truy cập vào trang `hostname.php` thì sẽ in ra tên của container đang chạy webserver.
```sh
root@master:/opt/demo# curl http://10.97.115.115:5555/hostname.php
apache2-4181921630-w25sw
```


# 2. Thử nghiệm tính năng scale, load balancing.
- Scale
```sh
root@master:/opt/demo# kubectl scale --replicas=5 deployments/apache2
deployment "apache2" scaled
```

- Check deployment, pods, services
```sh
root@master:/opt/demo# kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
apache2   5         5         5            5           19m


root@master:/opt/demo# kubectl describe deployment apache2
Name:                   apache2
Namespace:              default
CreationTimestamp:      Thu, 27 Apr 2017 16:23:03 +0700
Labels:                 name=apache2
Annotations:            deployment.kubernetes.io/revision=1
Selector:               name=apache2
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:       name=apache2
  Containers:
   apache2:
    Image:              cosy294/swarm:1.0
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Progressing   True    NewReplicaSetAvailable
  Available     True    MinimumReplicasAvailable
OldReplicaSets: <none>
NewReplicaSet:  apache2-4181921630 (5/5 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath   Type        Reason                   Message
  ---------     --------        -----   ----                    -------------   --------    ------                   -------
  20m           20m             1       deployment-controller                   Normal      ScalingReplicaSet        Scaled up replica set apache2-4181921630 to 1
  46s           46s             1       deployment-controller                   Normal      ScalingReplicaSet        Scaled up replica set apache2-4181921630 to 5
root@master:/opt/demo# 
```

```sh
root@master:/opt/demo# kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
apache2-4181921630-58f7d   1/1       Running   0          1m
apache2-4181921630-5ssnk   1/1       Running   0          1m
apache2-4181921630-brblz   1/1       Running   0          1m
apache2-4181921630-nhq09   1/1       Running   0          1m
apache2-4181921630-w25sw   1/1       Running   0          21m
root@master:/opt/demo# 
```

```sh
root@master:/opt/demo# kubectl get svc
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
apache2      10.97.115.115   <nodes>       5555:30815/TCP   21m
kubernetes   10.96.0.1       <none>        443/TCP          8d
root@master:/opt/demo# 




root@master:/opt/demo# kubectl describe svc apache2
Name:                   apache2
Namespace:              default
Labels:                 <none>
Annotations:            <none>
Selector:               name=apache2
Type:                   NodePort
IP:                     10.97.115.115
Port:                   <unset> 5555/TCP
NodePort:               <unset> 30815/TCP
Endpoints:              10.36.0.3:80,10.36.0.4:80,10.39.0.2:80 + 2 more...
Session Affinity:       None
Events:                 <none>
root@master:/opt/demo# 
```


Chú ý, vào thông số endpoint, so với ban đầu, ta có thêm các endpoint khác vừa mới được tạo ra (các container vừa được tạo ra).


- Access webserver
```sh
root@master:/opt/demo# while true; do curl 10.97.115.115:5555/hostname.php;echo -e "\n";sleep 10;done
apache2-4181921630-nhq09

apache2-4181921630-nhq09

apache2-4181921630-brblz

apache2-4181921630-w25sw

apache2-4181921630-w25sw

apache2-4181921630-5ssnk

apache2-4181921630-brblz

apache2-4181921630-5ssnk

apache2-4181921630-5ssnk

apache2-4181921630-brblz

apache2-4181921630-w25sw

apache2-4181921630-w25sw

apache2-4181921630-w25sw

apache2-4181921630-brblz
```

=> Các request sẽ được tuần tự chuyển đến các container khác nhau xử lý. (Cân bằng tải).

## 2.1 Thử nghiệm tính năng autoscale.
- Tính năng autoscale sẽ tự động tạo ra các endpoint (container khác) trên cụm cluster khi mà các endpoint hiện tại không đáp ứng được năng lực xử lý. Nhằm giải quyết bài toán đáp ứng yêu cầu của người dùng trong từng thời điểm khác nhau.

```sh
root@master:~# kubectl autoscale deployments apache2 --max=10 --cpu-percent=80
```

# 3. Thử nghiệm tính năng update images, roll back.

## 3.1 Update với deployment
### 3.1.1 Update deployments với file cấu hình:

```sh
root@master:/opt/demo# kubectl apply -f apache2_new.yaml --record
```

Trong đó file `apache2_new.yaml` là file cấu hình mới.

#### 3.1.2 Update deployments với image phiên bản mới hơn.
```sh
root@master:/opt/demo# kubectl set image deployments/apache2 apache2=nginx
```

Trong đó, `apache2` là tên container và `images` mới là nginx

Sau khi update, toàn bộ container sẽ được update lên image mới. Ở đây tôi chuyển sang sử dựng webserver là nginx chứ ko phải là apache2 như ban đầu nữa.

```sh
root@master:~# curl 10.97.115.115:5555
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
root@master:~# 
```

## 3.2 Update với Replication Controller

### 3.2.1 Update Replication Controller với file cấu hình mới:

```sh
$ kubectl rolling-update frontend-v1 -f frontend-v2.yaml
```

Trong đó, `demo-rc-v0.2.yml` là file cấu hình mới.

### 3.2.2 Update Replication Controller với images mới.

```sh
$ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2
```


## 3.3 Rollback deployments apache2 quay về phiên bản trước đấy:

```sh
root@master:/opt/demo# kubectl rollout undo deployments apache2
deployment "apache2" rolled back
root@master:/opt/demo# 
```
