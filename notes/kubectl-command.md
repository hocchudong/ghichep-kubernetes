# Câu lệnh về kubectl

#### View cluster info
```sh
root@master:~# kubectl cluster-info
Kubernetes master is running at https://172.16.69.237:6443
Heapster is running at https://172.16.69.237:6443/api/v1/proxy/namespaces/kube-system/services/heapster
KubeDNS is running at https://172.16.69.237:6443/api/v1/proxy/namespaces/kube-system/services/kube-dns

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
root@master:~# 
```

#### List the nodes in cluster:
```sh
root@master:~# kubectl get nodes
NAME      STATUS    AGE       VERSION
master    Ready     7d        v1.6.1
minion1   Ready     7d        v1.6.1
minion2   Ready     7d        v1.6.1
root@master:~# 
```


#### List all Containers in all namespaces
```sh
kubectl get pods --all-namespaces
```

#### List Containers filtering by Pod namespace
```sh
kubectl get pods --namespace kube-system
```

Trong đó, `kube-system` là tên namespace.

#### List cluster events:
```sh
root@master:/opt/heapster/deploy# kubectl get events
LASTSEEN   FIRSTSEEN   COUNT     NAME                              KIND                      SUBOBJECT                         TYPE      REASON                         SOURCE                      MESSAGE
3m         3m          1         php-apache-3580908300-x6tmd       Pod                                                         Normal    Scheduled                      default-scheduler           Successfully assigned php-apache-3580908300-x6tmd to minion2
3m         3m          1         php-apache-3580908300-x6tmd       Pod                       spec.containers{php-apache}       Normal    Pulling                        kubelet, minion2            pulling image "gcr.io/google_containers/hpa-example"
2m         2m          1         php-apache-3580908300-x6tmd       Pod                       spec.containers{php-apache}       Normal    Pulled                         kubelet, minion2            Successfully pulled image "gcr.io/google_containers/hpa-example"
2m         2m          1         php-apache-3580908300-x6tmd       Pod                       spec.containers{php-apache}       Normal    Created                        kubelet, minion2            Created container with id b5ff9a5fe3e47666b78835dd818ea70d989f48121db9806628dc9cb0e0bb0d6c
2m         2m          1         php-apache-3580908300-x6tmd       Pod                       spec.containers{php-apache}       Normal    Started                        kubelet, minion2            Started container with id b5ff9a5fe3e47666b78835dd818ea70d989f48121db9806628dc9cb0e0bb0d6c
3m         3m          1         php-apache-3580908300             ReplicaSet                                                  Normal    SuccessfulCreate               replicaset-controller       Created pod: php-apache-3580908300-x6tmd
```

#### List deployments, pods, svc 
```sh
kubectl get deployments
kubectl get pods
kubectl get svc
```

#### Delete nodes
```sh
root@master:~# kubectl delete nodes minion1
node "minion1" deleted
root@master:~# 
```

#### Delete deployments, pods, svc
```sh
root@master:~# kubectl delete svc monitoring-grafana --namespace=kube-system
service "monitoring-grafana" deleted
root@master:~# 
```

#### Get describe deployments pods, svc,...
```sh
root@master:~# kubectl describe deployments php-apache
Name:                   php-apache
Namespace:              default
CreationTimestamp:      Wed, 26 Apr 2017 00:01:27 +0700
Labels:                 run=php-apache
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=php-apache
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:       run=php-apache
  Containers:
   php-apache:
    Image:      gcr.io/google_containers/hpa-example
    Port:       80/TCP
    Requests:
      cpu:              200m
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
OldReplicaSets: <none>
NewReplicaSet:  php-apache-3580908300 (1/1 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath   Type            Reason                    Message
  ---------     --------        -----   ----                    -------------   --------        ------                    -------
  5m            5m              1       deployment-controller                   Normal          ScalingReplicaSet Scaled up replica set php-apache-3580908300 to 1
root@master:~# 

```

#### Get a shell to the running Container:
```sh
kubectl exec -it shell-demo /bin/bash
```

Trong đó `shell-demo` là tên container.

#### Scale
```sh
# Scale a replicaset named 'foo' to 3.
root@master:~# kubectl scale --replicas=3 rs/foo

# If the deployment named mysql's current size is 2, scale mysql to 3.
root@master:~# kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
```

#### auto scale
```sh
# Auto scale a replication controller "foo", with the number of pods between 1
and 5, target CPU utilization at 80%:
root@master:~# kubectl autoscale rc foo --max=5 --cpu-percent=80
```


#### Port Forwarding: Forward one or more local ports to a pod.
```sh
# Listen on port 8888 locally, forwarding to 5000 in the pod
root@master:~# kubectl port-forward mypod 8888:5000
```

#### Expose: Expose a resource as a new Kubernetes service. 
```sh
# Create a service for an nginx deployment, which serves on port 80 and connects to the containe
rs on port 8000.
root@master:~# kubectl expose deployment nginx --port=80 --target-port=8000
```

#### Rollout
kubectl rollout supports both Deployment and DaemonSet. It has the following subcommands:

- `kubectl rollout undo` works like rollback; it allows the users to rollback to a previous version of deployment.
```sh
# Rollback to the previous deployment
root@master:~# kubectl rollout undo deployment/abc
```

- `kubectl rollout pause` allows the users to pause a deployment. See pause deployments.
- `kubectl rollout resume` allows the users to resume a paused deployment.
- `kubectl rollout status` shows the status of a deployment.
- `kubectl rollout history` shows meaningful version information of all previous deployments. See development version.
- `kubectl rollout retry` retries a failed deployment. See perm-failed deployments.

#### Rolling-update
- Note that kubectl rolling-update only supports Replication Controllers.
- A rolling update works by:
  - 1. Creating a new replication controller with the updated configuration.
  - 2. Increasing/decreasing the replica count on the new and old controllers until the correct number of replicas is reached.
  - 3. Deleting the original replication controller.

```sh
// Update pods of frontend-v1 using new replication controller data in frontend-v2.json.
$ kubectl rolling-update frontend-v1 -f frontend-v2.json

// Update the pods of frontend-v1 to frontend-v2
$ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

// Update the pods of frontend, keeping the replication controller name
$ kubectl rolling-update frontend --image=image:v2
```

# Reference
- https://kubernetes.io/docs/tasks/