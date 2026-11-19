---
title: "Bản đề xuất"
weight: 2
chapter: false
pre: "<b>2. </b>"
---



## Thiết kế và triển khai hệ thống Web Server bảo mật trên AWS sử dụng VPC, EC2, S3 và CloudWatch.

**Bài toán ứng dụng:** Website đăng nhập để lưu trữ và tải xuống tài liệu của nhóm học tập.

**Nhóm thực hiện:** 4 thành viên.



## 2.1. Tóm tắt

Dự án xây dựng một Web Server trên AWS, tập trung vào kiểm soát truy cập mạng, bảo mật dữ liệu và giám sát hoạt động. Hệ thống sử dụng bốn dịch vụ chính: Amazon VPC, Amazon EC2, Amazon S3 và Amazon CloudWatch; IAM cung cấp quyền truy cập cho các thành phần.

Bài toán thực tế là một nhóm học tập cần một website nhỏ để chia sẻ tài liệu. Người dùng đăng nhập bằng tài khoản thử nghiệm được tạo sẵn, xem danh sách tài liệu, tải tài liệu lên và tải tài liệu xuống. Người chưa đăng nhập không được thực hiện các thao tác này. Tài liệu được lưu trong S3 riêng tư và truy cập thông qua ứng dụng chạy trên EC2.

Amazon VPC cung cấp môi trường mạng; Security Group và Network ACL kiểm soát lưu lượng. Amazon EC2 chạy Nginx và ứng dụng web. Amazon S3 lưu trữ tài liệu với mã hóa và Versioning. Amazon CloudWatch thu thập chỉ số, nhật ký và hỗ trợ cảnh báo.

Nhóm gồm 4 thành viên, mỗi người phụ trách một mảng dịch vụ và xây dựng workshop tương ứng. Bốn phần được tích hợp thành một hệ thống dùng chung. Phạm vi triển khai là môi trường học tập có thời hạn, ưu tiên cấu hình đơn giản và kiểm soát chi phí.

## 2.2. Vấn đề cần giải quyết

### Nhu cầu lưu trữ và chia sẻ tài liệu

Việc gửi tài liệu qua nhiều kênh khiến nhóm khó theo dõi danh sách và tìm lại phiên bản trước. Nhóm cần một nơi lưu trữ tập trung với giao diện dễ sử dụng.

**Yêu cầu:** Có chức năng đăng nhập, đăng xuất, xem danh sách, tải lên và tải xuống tài liệu. Trong phạm vi cơ bản, các tài khoản thử nghiệm dùng chung một kho tài liệu; chưa triển khai kho riêng cho từng người.

### Kiểm soát truy cập mạng và ứng dụng

Web Server tiếp nhận kết nối từ Internet có thể bị truy cập vào các cổng không cần thiết hoặc thử đăng nhập trái phép. Chỉ ẩn chức năng trên giao diện không đủ để ngăn yêu cầu trực tiếp đến ứng dụng.

**Yêu cầu:** Chỉ mở các cổng cần thiết, giới hạn SSH theo IP quản trị, sử dụng HTTPS và kiểm tra phiên đăng nhập tại mọi chức năng xử lý tài liệu. Security Group và Network ACL phụ trách lọc lưu lượng mạng; việc xác thực người dùng do ứng dụng xử lý.

### Bảo vệ dữ liệu và quyền truy cập S3

Bucket công khai, quyền IAM quá rộng hoặc Access Key được ghi trong mã nguồn có thể làm lộ tài liệu. Việc ghi đè đối tượng cũng cần có phương án phục hồi.

**Yêu cầu:** Giữ S3 riêng tư, sử dụng IAM Role gắn với EC2, giới hạn quyền theo bucket và prefix, mã hóa dữ liệu và kiểm thử phục hồi phiên bản.

### Theo dõi hoạt động và phát hiện lỗi

Nhóm cần xác định được lỗi đến từ tài nguyên máy chủ, ứng dụng hay quyền truy cập S3. Nhật ký mạng không thể thay thế nhật ký đăng nhập và xử lý tài liệu.

**Yêu cầu:** Thu thập log Nginx, log ứng dụng và các chỉ số thiết yếu; xây dựng Dashboard và kiểm thử cảnh báo. Không ghi mật khẩu, cookie hoặc token phiên vào log.

### Giới hạn chi phí và khả năng tái triển khai

Tài nguyên chạy liên tục, log quá nhiều và tài nguyên chưa được dọn dẹp có thể tạo chi phí ngoài kế hoạch.

**Yêu cầu:** Dùng một EC2 cho môi trường lab, giới hạn thời gian vận hành và lưu log, ghi nhận chi phí thực tế, đồng thời xây dựng workshop Markdown đủ rõ để triển khai lại.

## 2.3. Giải pháp đề xuất

### Phạm vi và cấu hình dự kiến

| Hạng mục | Phương án đề xuất |
|---|---|
| Khu vực | Một Region, dự kiến Singapore (`ap-southeast-1`) |
| Mạng | Một VPC, một Public Subnet, Internet Gateway và Route Table |
| Máy chủ | Một EC2 `t4g.micro`, Amazon Linux 2023 ARM64; kiểm tra tương thích thư viện trước triển khai |
| Ổ đĩa | EBS gp3 8 GiB có mã hóa; điều chỉnh nếu thực tế không đủ |
| Web Server | Nginx xử lý HTTPS và chuyển tiếp yêu cầu đến ứng dụng |
| Ứng dụng | Python Flask, chạy bằng Gunicorn và quản lý bằng systemd; không dùng máy chủ phát triển để phục vụ Internet |
| Người dùng | Tài khoản thử nghiệm tạo sẵn; mật khẩu được băm bằng thư viện phù hợp |
| Kho tài liệu | Một S3 bucket riêng tư; prefix thống nhất là `portal/` |
| Kết nối S3 | AWS SDK sử dụng IAM Role của EC2; lưu lượng đến S3 đi qua Gateway VPC Endpoint |
| Giám sát | CloudWatch Metrics, Logs, một Dashboard và tối đa hai alarm hoạt động đồng thời |
| Thông báo | SNS hỗ trợ gửi email khi kiểm thử cảnh báo |

Phạm vi cơ bản chưa có đăng ký tài khoản, quên mật khẩu, phân quyền nhiều nhóm, tìm kiếm toàn văn hoặc chỉnh sửa tài liệu trực tuyến. Chỉ sử dụng tệp thử nghiệm không nhạy cảm. Không bổ sung cơ sở dữ liệu được quản lý cho bài toán này; tài khoản thử nghiệm và cấu hình xác thực được quản lý trên EC2, bên ngoài thư mục phục vụ web.

### Kiến trúc tổng quan

Người dùng truy cập website qua HTTPS. Nginx trên EC2 chuyển yêu cầu đến ứng dụng. Sau khi xác thực phiên đăng nhập, ứng dụng thực hiện thao tác S3 bằng IAM Role và trả kết quả về trình duyệt. S3 và CloudWatch là dịch vụ bên ngoài subnet; S3 Gateway Endpoint không đặt bucket bên trong VPC.

![Sơ đồ kiến trúc](/FCAJ_Worklog/images/5/so_do_kien_truc_aws.drawio.png/?featherlight=false&width=90pc)

**Các thành phần cần có trong sơ đồ:**

- Người dùng Internet và quản trị viên có IP được phép.
- AWS Cloud, VPC, Public Subnet, Internet Gateway và Route Table.
- Security Group và Network ACL bảo vệ EC2.
- EC2 chạy Nginx, Gunicorn và ứng dụng Flask.
- IAM Role của EC2, S3 Gateway Endpoint và S3 bucket riêng tư.
- CloudWatch nhận metrics, log Nginx, log ứng dụng và VPC Flow Logs; SNS hỗ trợ thông báo.

### Luồng đăng nhập và xử lý tài liệu

1. Người dùng truy cập website bằng HTTPS và gửi thông tin đăng nhập.
2. Ứng dụng kiểm tra mật khẩu với giá trị băm, tạo phiên đăng nhập và trả cookie với các thuộc tính bảo vệ phù hợp như `Secure`, `HttpOnly`, `SameSite`.
3. Khi người dùng mở danh sách tài liệu, ứng dụng kiểm tra phiên và liệt kê đối tượng trong prefix `portal/`.
4. Khi tải lên, ứng dụng xác thực phiên, kiểm tra CSRF, loại và kích thước tệp, chuẩn hóa tên, rồi ghi đối tượng vào S3.
5. Khi tải xuống, ứng dụng kiểm tra phiên và object key thuộc phạm vi cho phép, đọc đối tượng từ S3 rồi truyền về trình duyệt dưới dạng tệp đính kèm.
6. Khi đăng xuất, phiên được vô hiệu hóa; các yêu cầu tiếp theo đến chức năng tài liệu phải đăng nhập lại.
7. Ứng dụng ghi kết quả thao tác và lỗi để gửi đến CloudWatch.

Trong phương án này, trình duyệt không truy cập S3 trực tiếp và không nhận Access Key. Không sử dụng đường dẫn tải xuống trực tiếp từ S3 trong luồng cơ bản. Tài liệu không được đồng bộ vào thư mục web công khai của Nginx.

### Firewall và bảo mật ứng dụng

| Kiểm soát | Cấu hình dự kiến |
|---|---|
| SSH — TCP 22 | Chỉ cho phép IP quản trị `/32`; xác thực bằng khóa |
| HTTPS — TCP 443 | Phục vụ truy cập website |
| HTTP — TCP 80 | Chỉ chuyển hướng sang HTTPS và phục vụ xác minh chứng chỉ khi cần |
| Cổng ứng dụng | Chỉ lắng nghe trên loopback; không mở trực tiếp ra Internet |
| Network ACL | Cấu hình cả hai chiều và cổng phản hồi; kiểm thử DENY với IP thử nghiệm rồi khôi phục |
| Đăng nhập | Kiểm tra tại máy chủ, thông báo lỗi không tiết lộ tài khoản, giới hạn tốc độ thử đăng nhập |
| Phiên | Có thời hạn; không lưu mật khẩu hoặc thông tin bí mật trong cookie |
| Tải tệp | Dự kiến chỉ cho phép PDF/TXT, tối đa 10 MiB mỗi tệp; kiểm tra cả phía ứng dụng và Nginx |
| Tên tệp | Loại bỏ thành phần đường dẫn; giới hạn object key trong prefix cho phép; xử lý tên trùng có chủ đích |
| Bí mật | Không đưa mật khẩu, khóa SSH, khóa phiên hoặc thông tin xác thực vào Git/workshop/log |

Sử dụng tên miền do nhóm quản lý và chứng chỉ hợp lệ nếu có. Nếu chỉ có chứng chỉ tự ký, phải ghi rõ đây là kiểm thử mã hóa trong lab, chưa đạt tiêu chí chứng chỉ được trình duyệt tin cậy. Không gửi thông tin đăng nhập qua HTTP.

Security Group và Network ACL không thay thế kiểm tra đầu vào hoặc bảo vệ ứng dụng trước các lỗi như XSS. Phạm vi này không triển khai dịch vụ AWS Network Firewall hoặc AWS WAF. Kiểm tra định dạng tệp cũng không tương đương quét mã độc; chỉ sử dụng tài liệu thử nghiệm đáng tin cậy.

### Bảo mật dữ liệu trên S3

- Bật toàn bộ Block Public Access; không bật Static Website Hosting.
- Sử dụng SSE-S3 và kiểm tra thuộc tính mã hóa của đối tượng sau khi tải lên.
- Bật Versioning để kiểm thử khôi phục khi ghi đè tài liệu.
- Cấp `s3:ListBucket` trên bucket với điều kiện prefix phù hợp; cấp `s3:GetObject` và `s3:PutObject` cho `portal/*`.
- Ứng dụng không có chức năng xóa nên không cấp quyền xóa đối tượng cho role ứng dụng. Quyền khôi phục phiên bản và dọn dẹp thuộc vai trò quản trị phù hợp.
- Dùng Bucket Policy từ chối truyền dữ liệu không qua TLS; đồng bộ IAM Policy và Endpoint Policy với cùng bucket/prefix.
- Nếu giới hạn truy cập theo `aws:SourceVpce`, thiết kế ngoại lệ quản trị có chủ đích và kiểm thử trước khi áp dụng; thao tác AWS Console không tự đi qua Gateway Endpoint.
- Versioning phục vụ phục hồi trong lab, không thay thế bản sao lưu độc lập khi tài khoản hoặc toàn bộ phiên bản bị mất.

### Giám sát và cảnh báo

CloudWatch thu thập CPU, network, status check của EC2; CloudWatch Agent bổ sung các chỉ số bộ nhớ, đĩa cần thiết và gửi log. Log ứng dụng ghi thời gian, loại thao tác, mã yêu cầu và kết quả đăng nhập/tải lên/tải xuống/lỗi S3, tránh nội dung tài liệu và dữ liệu bí mật.

Dashboard thể hiện sức khỏe máy chủ và lỗi ứng dụng. Nhóm chọn tối đa hai alarm để kiểm thử, chẳng hạn CPU cao và nhiều lần đăng nhập thất bại. Metric filter phải khớp định dạng log thực tế; ngưỡng được điều chỉnh sau khi quan sát hoạt động thử nghiệm. SNS hỗ trợ kiểm tra nhận email thông báo.

VPC Flow Logs chỉ bật trong các buổi kiểm thử mạng để đối chiếu ACCEPT/REJECT. Flow Logs không cho biết tài khoản nào đã tải tài liệu nào; thông tin đó phải lấy từ log ứng dụng.

### Phân công 4 thành viên

| Thành viên | Phụ trách | Công việc và đầu ra chính |
|---|---|---|
| Thành viên 1 | VPC và Firewall | Tạo mạng, route, Security Group, Network ACL, S3 Gateway Endpoint; kiểm thử cho phép/từ chối và bàn giao thông tin mạng |
| Thành viên 2 | EC2 và ứng dụng | Triển khai Nginx/HTTPS, đăng nhập/đăng xuất, danh sách, tải lên/tải xuống, kết nối S3, bảo vệ phiên và ghi log |
| Thành viên 3 | S3 và bảo mật dữ liệu | Tạo bucket/prefix, mã hóa, Versioning, quyền IAM và Bucket Policy; phối hợp Endpoint Policy, kiểm thử truy cập và phục hồi |
| Thành viên 4 | CloudWatch | Cấu hình Agent, log group, Dashboard, metric filter, alarm, thông báo; đối chiếu log và hỗ trợ đánh giá chi phí |

Cả nhóm thống nhất Region, tên bucket, prefix, role EC2, định dạng log và tài khoản thử nghiệm trước khi triển khai. Các quyền S3 và CloudWatch cần được tích hợp vào role gắn với EC2. Mỗi người có phần giới thiệu, chuẩn bị, triển khai, kiểm thử và dọn dẹp trong workshop riêng; cùng sử dụng một bài toán và kiến trúc thống nhất.

Tài nguyên được tạo chủ yếu qua AWS Console. Việc cấu hình hệ điều hành, Nginx và triển khai mã nguồn vẫn cần thực hiện trên EC2. Ảnh minh chứng và liên kết video chỉ được bổ sung sau khi thực hành, không dùng kết quả giả định làm bằng chứng.

### Kế hoạch triển khai và tối ưu chi phí

Thời gian dự kiến là 6 tuần: tuần 1 thống nhất thiết kế và quyền; tuần 2 triển khai mạng/EC2; tuần 3 tích hợp ứng dụng/S3; tuần 4 triển khai giám sát; tuần 5 kiểm thử; tuần 6 hoàn thiện tài liệu, video và dọn dẹp.

- Duy trì một EC2; dự kiến tổng thời gian chạy lab khoảng 48 giờ, điều chỉnh theo thực tế. Với dòng T, dự kiến dùng CPU credit mode Standard.
- Không triển khai NAT Gateway, ALB hoặc cấu hình nhiều AZ trong phạm vi cơ bản.
- Dừng EC2 sau buổi thực hành; theo dõi các khoản còn phát sinh như EBS và các tài nguyên được giữ lại. Kiểm tra IP public và cập nhật DNS khi khởi động lại.
- Chỉ thu thập metrics cần thiết, một Dashboard, tối đa hai alarm; lưu log 1–3 ngày trong lab và chụp minh chứng trước khi log hết hạn.
- Dùng tệp nhỏ, giới hạn số lượt thử nghiệm; kiểm soát dung lượng cả phiên bản hiện tại và phiên bản cũ của S3.
- Lập dự toán theo Region, thời gian chạy, EBS, IPv4 public, dung lượng/yêu cầu S3, truyền dữ liệu và CloudWatch trước khi triển khai. Không mặc định toàn bộ dự án miễn phí hoặc tài khoản đủ điều kiện Free Tier.
- Ghi nhận chi phí thực tế sau các buổi lab; cập nhật ngưỡng ngân sách do nhóm thống nhất.
- Sau nghiệm thu, xóa các tài nguyên không còn dùng: EC2/EBS liên quan, mọi phiên bản và delete marker trong S3, log group, Dashboard, alarm, metric filter, SNS, Endpoint và tài nguyên mạng theo thứ tự phụ thuộc. Chỉ xóa role/policy do dự án tạo và không dùng chung.

## 2.4. Kết quả dự kiến

### Hệ thống và chức năng

Nhóm triển khai được một website tài liệu hoạt động qua HTTPS, với chức năng đăng nhập, đăng xuất, xem danh sách, tải lên và tải xuống. Tài liệu được lưu tại S3 riêng tư, EC2 sử dụng quyền tạm thời từ IAM Role. Người chưa đăng nhập bị từ chối ở phía máy chủ.

### Tiêu chí kiểm thử và nghiệm thu

| Mã | Kịch bản | Kết quả mong đợi |
|---|---|---|
| T01 | Truy cập HTTPS và thử HTTP | HTTPS hoạt động; HTTP chuyển hướng; ghi rõ tình trạng tin cậy của chứng chỉ |
| T02 | Đăng nhập sai | Bị từ chối và có sự kiện log phù hợp |
| T03 | Đăng nhập đúng | Truy cập được trang danh sách tài liệu |
| T04 | Chưa đăng nhập, gọi trực tiếp chức năng danh sách/tải lên/tải xuống | Bị từ chối hoặc chuyển về đăng nhập; không trả dữ liệu tài liệu |
| T05 | Tải lên tệp hợp lệ | Có đối tượng trong prefix `portal/`; thuộc tính mã hóa đúng dự kiến |
| T06 | Tải xuống sau đăng nhập | Nội dung khớp tệp gốc, có thể đối chiếu checksum |
| T07 | Tải tệp quá giới hạn hoặc loại không được phép | Bị từ chối và không tạo đối tượng ngoài dự kiến |
| T08 | Đăng xuất rồi yêu cầu tài liệu | Phiên cũ không còn được chấp nhận |
| T09 | Truy cập đối tượng S3 ẩn danh | Không đọc được tài liệu |
| T10 | Role ứng dụng truy cập ngoài prefix hoặc thực hiện thao tác không được cấp | Bị từ chối; thao tác hợp lệ trong prefix vẫn thành công |
| T11 | Ghi đè một đối tượng thử nghiệm | Có phiên bản cũ và quản trị viên phục hồi được nội dung |
| T12 | Thử SSH từ nguồn cho phép/không cho phép và thử NACL DENY | Kết quả đúng cấu hình; có bước hoàn nguyên quy tắc thử nghiệm |
| T13 | Thực hiện đăng nhập, tải lên, tải xuống và tạo lỗi có kiểm soát | Tra cứu được sự kiện tương ứng trong CloudWatch |
| T14 | Tạo điều kiện vượt ngưỡng alarm | Alarm đổi trạng thái và nhận được thông báo thử nghiệm |
| T15 | Dọn dẹp sau nghiệm thu | Kiểm tra không còn tài nguyên dự án ngoài danh sách chủ đích giữ lại |

Các kiểm thử phải ghi kết quả thực tế, thời điểm, ảnh minh chứng và cách khắc phục khi chưa đạt. Nhóm cần trình diễn ít nhất một lần phục hồi phiên bản và một cảnh báo có thông báo thành công.

