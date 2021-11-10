# Làm quen với K8S

- Các lệnh ban đầu để làm quen với K8S.
- Sau khi cài đặt xong K8S, hãy thực hành các lệnh sau để làm quen với cách sử dụng lệnh trong K8S. 
- K8S sử dụng lệnh `kubectl` để làm việc 

## Liệt kê node và pod

- Liệt kê pod trong namespace `default`

  ```
  kubectl get pods
  ```

Kết quả: 

- Lệnh trên sẽ trả về dòng `No resources found in default namespace.` nếu chưa có pod nào. Nếu có thì danh sách pod sẽ được liệt kê ra.

  ```
  root@master01:~# kubectl get pods
  No resources found in default namespace.
  ```

- Liệt kê `pod` trong tất cả các namespace

  ```
  kubectl get pods --all-namespace
  ```
  
Kết quả: 

- Lệnh trên sẽ trả về danh sách của toàn bộ các `pods` có trong hệ thống.

  ```
  root@master01:~# kubectl get pods --all-namespaces
  NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
  kube-system   calico-kube-controllers-7659fb8886-tflvx   1/1     Running   0          23h
  kube-system   calico-node-k72wv                          1/1     Running   0          23h
  kube-system   calico-node-klg9d                          1/1     Running   0          23h
  kube-system   calico-node-lbbjh                          1/1     Running   0          23h
  kube-system   coredns-78fcd69978-4vnkr                   1/1     Running   0          23h
  kube-system   coredns-78fcd69978-gprzq                   1/1     Running   0          23h
  kube-system   etcd-master01                              1/1     Running   0          23h
  kube-system   kube-apiserver-master01                    1/1     Running   0          23h
  kube-system   kube-controller-manager-master01           1/1     Running   0          23h
  kube-system   kube-proxy-4c89b                           1/1     Running   0          23h
  kube-system   kube-proxy-bw8gg                           1/1     Running   0          23h
  kube-system   kube-proxy-d7nvf                           1/1     Running   0          23h
  kube-system   kube-scheduler-master01                    1/1     Running   0          23h
  ```