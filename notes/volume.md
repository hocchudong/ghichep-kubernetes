#1. Volume

## 1.1 Các loại volume trong kubernetes:
- emptyDir
- hostPath
- gcePersistentDisk
- awsElasticBlockStore
- nfs
- iscsi
- flocker
- glusterfs
- rbd
- cephfs
- gitRepo
- secret
- persistentVolumeClaim
- downwardAPI
- azureFileVolume
- azureDisk
- vsphereVolume
- Quobyte
- PortworxVolume
- ScaleIO

### 1.1.1 emptyDir
- Là dạng volume được tạo ra khi 1 Pod được gán vào 1 node, tồn tại trong suốt quá trình Pod chạy trên node.
- Volume trống.
- Container trong Pod có thể đọc và viết vào volume này.
- Khi Pod bị xóa khỏi node, dữ liệu trong emptyDir sẽ bị xóa.
- Container bị crashing, dữ liệu không bị mất.
- emptyDir lưu trữ trên thư mục `/var/lib/kubelet` của host.
- Ví dụ:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: gcr.io/google_containers/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### 1.2.1 hostPath
- Là dạng volume sẽ mount file or thư mục trên máy host vào pod. Tương tự docker.

```sh
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: gcr.io/google_containers/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
```

## 1.2 PersistentVolume
- PersistentVolume (PV) là nguồn tài nguyên lưu trữ qua mạng (networked storage) được cung cấp bởi admin trong 1 cluster.
- PersistentVolumeClaim (PVC) là yêu cầu của user về volume để sử dụng trong một pod.

# 2. Network
- Các yêu cầu cơ bản trong network của kubernetes:
  - Các containers nói chuyện với nhau mà không cần NAT.
  - Tất cả các nodes có thể nói chuyện với containers (và ngược lại) mà không cần NAT.
  - IP mà containers nhìn thấy chính là địa chỉ ip mà các thành phần khác nhìn thấy. (the IP that a container sees itself as is the same IP that others see it as)

- Trên thực tế, k8s áp dụng địa chỉ ip cho Pod. Các containers trong pod chia sẽ chung dải địa chỉ ip.
- Các giải pháp network trong k8s là:
  - Contiv
  - Contrail
  - Flannel
  - GCE
  - L2 networks and linux bridging
  - Nuage Networks VCS
  - OpenVSwitch
  - OVN
  - Project Calico
  - Romana
  - Weave Net from Weaveworks

# Reference
- https://kubernetes.io/docs/concepts/storage/volumes/
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/cluster-administration/networking/