### Note thu hanh k8s

- Trien khai mot ung dung  trong k8s

kubectl run kubernetes-bootcamp \
      --image=docker.io/jocatalin/kubernetes-bootcamp:v1 \
      --port=8080
			
- Sau khi trien khai xong, can thu hienj expose de ben ngoai co the truy cap su dung ung dung
			
kubectl expose deployment/kubernetes-bootcamp \
      --type="NodePort" \
      --port 8080
			
- Sau khi expose xong, su dung lenh duoi de xem port cua ung dung la gi

kubectl get services

- Ta có kết quả từ lệnh kubectl get services và thấy cặp port http://prntscr.com/jk4rs2, trong đó, Port thứ 2 là port của node để truy cập vào ứng dụng, ta sử dụng địa chỉ http://ip_cua_node:32633. Port của NodePort được sinh mặc định.
- Ta cũng có thể sử dụng lệnh curl để thực hiện lệnh "curl 10.96.198.164:8080", trong đó IP 10.96.198.164 được sinh ra phục vụ các kết nối bên trong của container khi triển khai trên K8S.  Quan sát các kết quả tại: http://prntscr.com/jk4soo


- Sau khi triển khai ứng dụng ở trên, thử scale up số cho ứng dụng vừa tạo ở trên (tăng số lượng container cùng chạy ứng dụng để đáp ứng yêu cầu dự phòng hoặc phục vụ số lượng truy cập lơn), ta thực hiện tăng số bản replicate container lên 03.


kubectl scale deployments/kubernetes-bootcamp --replicas=3

- Ta sẽ nhìn thấy số lượng bản replicas trong lệnh "kubectl get deployments" , tham khảo: http://prntscr.com/jk4tf0

- Khi thử truy cập vào ip http://ip_cua_node:32633, ta sẽ thấy lần lượt các request được gửi đến các container khác nhau. Tham khảo kết quả: http://prntscr.com/jk4trf

- Để thực hiện scale down về 02 ta dùng lệnh

kubectl scale deployments/kubernetes-bootcamp --replicas=2

- Ta quan sát kết quả sau khi thực hiện scale down: https://prnt.sc/jk4uso , sẽ có 1 pod chuyển trạng thái Terminating và biến mất.

- Tiếp theo, ta sẽ thực hiện rolling update cho ứng dụng vừa lấy ví dụ ở trên. Trước khi rolling upgrade, ta thấy khi curl hoặc truy cập vào web của ứng dụng, có dòng thông báo là version1, bây giờ ta sẽ thực hiện rolling upgrade lên version2.


kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2


- Kết quả: trước khi rolling upgrade http://prntscr.com/jk4vk0, sau khi rolling upgrade http://prntscr.com/jk4vnm


- Nếu ta muốn quay trở lại phiên bản trước, ta thực hiện lệnh: 

kubectl rollout undo deployments/kubernetes-bootcamp

- Thực hiện lệnh curl hoặc truy cập vào web để xem version.















