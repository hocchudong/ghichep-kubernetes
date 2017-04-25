# Kubernetes

Kubernetes là dự án mà nguồn mở để quản lý các container: automating deployment, scaling, and management các ứng dụng trên container. (Tạo, sửa, xoá, xếp lịch(schedule), mở rộng (scale)...) trên nhiều máy).

Kubernetes viết tắt là k8s.

Kubernetes hỗ trợ các công nghệ container là docker và rkt.

Các tiện ích mà k8s mang lại cho chúng ta:
  - Triển khai ứng dụng một cách nhanh chóng.
  - Scale ứng dụng dễ dàng.
  - Liên tục đưa ra các tính năng mới.
  - Tối ưu hóa việc sử dụng tài nguyên.


# 1. Các khái niệm trong K8S.
## 1.1 Pod
- Pod là 1 nhóm (1 trở lên) các container chứa ứng dụng cùng chia sẽ các tài nguyên lưu trữ, địa chỉ ip...
- Pod có thể chạy theo 2 cách sau:
  - **Pods that run a single container.**: 1 container tương ứng với 1 pod.
  - **Pods that run multiple containers that need to work together.**: Một Pod có thể là một ứng dụng bao gồm nhiều container
  được kết nối chặt chẽ và cần phải chia sẻ tài nguyên với nhau giữa các container.

- Pods cung cấp hai loại tài nguyên chia sẻ cho các containers: networking và storage.
- **Networking**: Mỗi pod sẽ được cấp 1 địa chỉ ip. Các container trong cùng 1 Pod cùng chia sẽ network namespace (địa chỉ ip và port).
Các container trong cùng pod có thể giao tiếp với nhau và có thể giao tiếp với các container ở pod khác (use the shared network resources).

- **Storage**: Pod có thể chỉ định một `shared storage volumes`. Các container trong pod có thể truy cập vào volume này.

## 1.2 Replication Controllers
- Replication controller đảm bảo rằng số lượng các pod replicas đã định nghĩa luôn luôn chạy đủ số lượng tại bất kì thời điểm nào. 
- Thông qua Replication controller, Kubernetes sẽ quản lý vòng đời của các pod, bao gồm scaling up and down, rolling deployments, and monitoring.

## 1.3 Services
- Vì pod có tuổi thọ ngắn nên không đảm bảo về địa chỉ IP mà chúng được cung cấp. 
- Service là khái niệm được thực hiện bởi : domain name, và port. Service sẽ tự động "tìm" các pod được đánh label phù hợp (trùng với label của service), rồi chuyển các connection tới đó.
- Nếu tìm được 5 pods thoả mã label, service sẽ thực hiện load-balancing: chia connection tới từng pod theo chiến lược được chọn (VD: round-robin: lần lượt vòng tròn).
- Mỗi service sẽ được gán 1 domain do người dùng lựa chọn, khi ứng dụng cần kết nối đến service, ta chỉ cần dùng domain là xong. Domain được quản lý bởi hệ thống name server SkyDNS nội bộ của k8s - một thành phần sẽ được cài khi ta cài k8s.
- Đây là nơi bạn có thể định cấu hình cân bằng tải cho nhiều pod và `expose` các pod đó.


## 1.4 Volumes
- Volumes thể hiện vị trí nơi mà các container có thể truy cập và lưu trữ thông tin.
- Volumes có thể là local filesystem,local storage, Ceph, Gluster, Elastic Block Storage,..
- Persistent volume (PV)  là khái niệm để đưa ra một dung lượng lưu trữ THỰC TẾ 1GB, 10GB ...
- Persistent volume claim (PVC) là khái niệm ảo, đưa ra một dung lượng CẦN THIẾT, mà ứng dụng yêu cầu.

Khi 1 PV thoả mãn yêu cầu của 1 PVC thì chúng "match" nhau, rồi "bound" (buộc / kết nối) lại với nhạu. 

## 1.5 Namespaces
- Namespace hoạt động như một cơ chế nhóm bên trong Kubernetes.
- Các Services, pods, replication controllers, và volumes có thể dễ dàng cộng tác trong cùng một namespace.
- Namespace cung cấp một mức độ cô lập với các phần khác của cluster.

## 1.6 ConfigMap (cm) - Secret
- ConfigMap là giải pháp để nhét 1 file config / đặt các ENVironment var hay set các argument khi gọi câu lệnh. ConfigMap là một cục config, mà pod nào cần, thì chỉ định là nó cần - giúp dễ dàng chia sẻ file cấu hình.
- secret dùng để lưu trữ các mật khẩu, token, ... hay những gì cần giữ bí mật. Nó nằm bên trong container.

## 1.7 Labels - Annotations
- Labels: Là các cặp  **key-value** được Kubernetes đính kèm vào pods, replication controllers,...
- Annotations: You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata.
- Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. 

# 2. Thành phần

![](https://s3-us-west-2.amazonaws.com/x-team-ghost-images/2016/06/o7leok.png)

## 2.1 Master node
- Chịu trách nhiệm quản lý Kubernetes cluster. 
- Đây là nơi mà sẽ cấu hình các nhiệm vụ sẽ thực hiện.
- Quản lý, điều hành các work node. 

### 2.1.1 API server
- API server là nơi tiếp nhận các lệnh REST được sử dụng để kiểm soát cluster.
- Nó xử lý các yêu cầu, xác nhận chúng, thực hiện các ràng buộc. 
- Trạng thái kết quả phải được duy trì ở một nơi nào đó, và điều đó đưa chúng ta đến thành phần tiếp theo của nút chính.

### 2.1.2 etcd storage
- etcd: một cơ sở dữ liệu key-value có tính khả dụng cao, phân phối và nhất quán sử dụng để tìm kiếm dịch vụ.
- Nó chủ yếu được sử dụng để chia sẻ các cấu hình và khám phá dịch vụ (service discovery).
- Ví dụ về dữ liệu được lưu trữ bởi Kubernetes trong etcd là các công việc được lên kế hoạch (jobs being scheduled), tạo và triển khai pod, services, namespaces, replication information,..

### 2.1.3 Scheduler
- Đảm nhiệm chức năng là triển khai các pods, services lên các nodes.
- Scheduler nắm các thông tin liên quan đến các tài nguyên có sẵn trên các thành viên của cluster, cũng như các yêu cầu cần thiết cho dịch vụ cấu hình để chạy và do đó có thể quyết định nơi triển khai một dịch vụ cụ thể.

## 2.1.4 controller-manager
- Sử dụng api server để có thể xem trạng thái của cluster và từ đó thực hiện các thay đổi chính xác cho trạng thái hiện tại để trở thành một trạng thái mong muốn.
- Ví dụ Replication controller có chức năng đảm bảo rằng số lượng các pod replicas đã định nghĩa luôn luôn chạy đủ số lượng tại bất kì thời điểm nào.

## 2.2 Worker node
- Là nơi mà các pod sẽ chạy.
- Chứa tất cả các dịch vụ cần thiết để quản lý kết nối mạng giữa các container, giao tiếp với master node, và gán các tài nguyên cho các container theo kế hoạch.

### 2.2.1 Docker
- Là môi trường để chạy các container.

### 2.2.2 kubelet
- kubelet lấy cấu hình thông tin pod từ api server và đảm bảo các containers up và running.
- kubelet chịu trách nhiệm liên lạc với master node.
- Nó cũng liên lạc với etcd, để có được thông tin về dịch vụ và viết chi tiết về những cái mới được tạo ra.

### 2.2.3 kube-proxy
- Kube-proxy hoạt động như một proxy mạng và cân bằng tải cho một dịch vụ trên một work node.
- Nó liên quan đến việc định tuyến mạng cho các gói TCP và UDP.

### 2.2.4 kubectl 
- Giao diện dòng lệnh để giao tiếp với API service.
- Gửi lệnh đến master node.

# 3. Tham khảo
- https://kubernetes.io/docs/concepts/
- https://deis.com/blog/2016/kubernetes-illustrated-guide/
- https://x-team.com/blog/introduction-kubernetes-architecture/
- http://www.familug.org/2017/04/kubernetes-end.html
- https://sapham.net/k8s-101-kien-truc-kubernetes/
