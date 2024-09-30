#Mã nguồn của Web-app

#Following scripts are to be run on the Cloud9 terminal
#Individual scripts are also mentioned in the Instructors Guide stepwise

Script-1:
# Use following script to create secret in secret manager for accessing your database via the web application
# Replace the values for below placeholders with the actual values you have used to configure the service
#<RDS Endpoint>
#<password>
#<username>
#<dbname>

aws secretsmanager create-secret \
    --name Mydbsecret7Student-Website9 \
    --description "Database secret for web app" \
    --secret-string "{\"user\":\"phamminhphuong\",\"password\":\"04082004\",\"host\":\"baystudent-website.cbcwmmea6dn5.us-east-1.rds.amazonaws.com\",\"db\":\"7Student-Website9\"}"

Script-2:
## Load testing

#Following command installs #loadtest package to perform load testing on the application
#Prerequisites are mentioned in the resources section

npm install -g loadtest

#Following command performs load testing on the given URL. replace the URL with Loadbalancer (or Public IP of EC2 instance)
# Press ctrl +C to stop the script

loadtest --rps 1000  -c 500 -k http://54.242.60.103


Script -3:
## Migration
# Following command exports the data from existing server.
# Prerequisites - This script expects mysql-client, mysqlpdump to be present
# AWS Cloud9 environment already has all the prerequisites installed for running these scripts

# Replace the <EC2instancePrivateip> with internal IP address of the EC2 instance (CapstonePOC) created in Phase-2 earlier.
# Provide the password when prompted

mysqldump -h 10.0.2.173 -u phamminhphuong -p --databases 7Student-Website9 > data.sql

#Following command imports the data into RDS database. Replace <RDSEndpoint> with the RDS Database endpoint you noted after RDS Database created in earlier steps.
#when prompted, enter password you provided during the time of database creation
mysql -h <RDSEndpoint> -u nodeapp -p  STUDENTS < data.sql

#Hướng dẫn deploy Web-app

Bước 1: Tạo VPC và các Subnets

Vào Amazon VPC, tạo một VPC với CIDR block 10.0.0.0/16.
Tạo 2 Public Subnets (cho các dịch vụ như Load Balancer và NAT Gateway) và 2 Private Subnets (cho EC2 Instances và RDS Database).
Thiết lập Internet Gateway cho VPC, kết nối với các Public Subnets.
Tạo NAT Gateway trong Public Subnet để kết nối Private Subnet với Internet mà không lộ địa chỉ IP của các Instance.

Bước 2: Cài đặt EC2 Instances

Vào Amazon EC2, tạo các EC2 Instances trong các Private Subnets.
Chọn loại EC2 instance t2.micro để tiết kiệm chi phí.
Sử dụng Amazon Linux 2 làm hệ điều hành.
Đảm bảo rằng các EC2 Instances được cấu hình với Auto Scaling để tự động điều chỉnh số lượng Instance dựa trên lưu lượng truy cập.

Bước 3: Thiết lập RDS Database

Tạo RDS Database (MySQL, Multi-AZ) để đảm bảo tính sẵn sàng cao và sao lưu tự động.
Chọn cấu hình Multi-AZ để bảo vệ dữ liệu trong trường hợp mất kết nối tại một Availability Zone.
Đảm bảo rằng RDS Database nằm trong Private Subnet để bảo mật.

Bước 4: Cấu hình Application Load Balancer (ALB)

Tạo Application Load Balancer (ALB) để cân bằng tải lưu lượng truy cập đến các EC2 Instances.
Cấu hình ALB để hỗ trợ HTTP và HTTPS, đồng thời tích hợp chứng chỉ SSL để bảo mật.
Liên kết ALB với các Security Groups để đảm bảo chỉ nhận được lưu lượng truy cập hợp lệ.

Bước 5: Cấu hình Security Groups và Network ACLs

Tạo các Security Groups cho EC2 Instances, RDS Database và ALB.
Mở cổng 80 và 443 cho ALB để xử lý lưu lượng HTTP/HTTPS.
Chỉ mở các cổng cần thiết (3306 cho MySQL, cổng SSH) giữa EC2 Instances và RDS Database.
Thiết lập Network ACLs để thêm lớp bảo vệ truy cập mạng vào các subnet.

Bước 6: Auto Scaling

Cấu hình Auto Scaling Group cho EC2 Instances, đặt số lượng tối thiểu và tối đa các Instances để đảm bảo ứng dụng luôn sẵn sàng khi có lượng truy cập tăng/giảm.
Cài đặt điều kiện Auto Scaling dựa trên CPU hoặc Network Traffic để tự động tăng/giảm số lượng EC2 Instances.

Bước 7: Monitoring

Sử dụng Amazon CloudWatch để giám sát hiệu suất của EC2 Instances, ALB và RDS Database.
Thiết lập cảnh báo (CloudWatch Alarms) khi hệ thống gặp sự cố hoặc vượt ngưỡng tài nguyên.

Bước 8: CI/CD Automation

Sử dụng AWS CodePipeline để tự động hóa quy trình triển khai mã.
Tích hợp CodeDeploy để triển khai mã tự động đến EC2 Instances, giúp giảm thiểu lỗi và tăng tốc độ phát hành ứng dụng.

Bước 9: Kiểm tra và Tối ưu hóa

Kiểm tra ứng dụng trên môi trường sản xuất để đảm bảo tính ổn định.
Sử dụng Amazon CloudFront để tăng tốc độ tải trang cho người dùng toàn cầu.
Tinh chỉnh cấu hình Auto Scaling, Security Groups và cấu trúc mạng để tối ưu hóa hiệu suất và chi phí.
