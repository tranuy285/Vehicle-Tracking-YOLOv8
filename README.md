# Vehicle-Tracking-YOLOv8

## 1. Giới thiệu
Dự án này tập trung xây dựng hệ thống theo dõi phương tiện giao thông tự động bằng cách sử dụng mô hình phát hiện YOLOv8 kết hợp thuật toán liên kết ByteTrack. Mục tiêu chính của hệ thống là phát hiện, thống kê số lượng và tính toán vận tốc của các phương tiện khi di chuyển qua các điểm kiểm soát. Giải pháp được thiết kế nhằm hoạt động ổn định trong điều kiện giao thông thực tế đô thị đông đúc, đặc biệt là khắc phục tốt tình trạng mất dấu đối tượng khi xảy ra hiện tượng che khuất (occlusion)

## 2. Mô tả dữ liệu
Hệ thống được huấn luyện và đánh giá trên bộ dữ liệu Smart Vehicle Dataset được thu thập từ nền tảng Roboflow.  
-  **Số lượng và phân chia:** Tập dữ liệu bao gồm tổng cộng 5892 hình ảnh, trong đó được chia thành 5460 ảnh cho tập huấn luyện (train) và 432 ảnh cho tập kiểm định (validation).  
-  **Lớp đối tượng (Classes):** Bao gồm 4 lớp phương tiện giao thông phổ biến: xe buýt (bus), xe hơi (car), xe máy (motorbike) và xe tải (truck).  
-  **Đặc điểm phân bố:** Dữ liệu có sự mất cân bằng khi lớp car chiếm đa số với 78%, tiếp đến là motorbike (9,1%), truck (8,5%) và bus chiếm tỷ lệ nhỏ nhất (4,4%). 
-  **Tiền xử lý và tăng cường (Data Augmentation):** Hình ảnh đầu vào được chuẩn hóa về kích thước 640x640 pixel. Để tăng tính khái quát, dữ liệu đã được áp dụng các kỹ thuật biến đổi như: lật ngang (Flip), xoay (Rotation ±15°), bóp méo (Shear), thay đổi độ sáng (Brightness), làm mờ (Blur) và thêm nhiễu (Noise)

## 3. Quy trình thực hiện
Quy trình xây dựng và tích hợp hệ thống được chia thành 3 giai đoạn cốt lõi:
-  **Phát hiện đối tượng (Detection):** Thực hiện tinh chỉnh (Fine-tuning) mô hình YOLOv8n trên tập dữ liệu Smart Vehicle Dataset trong 100 epochs thông qua môi trường Google Colab (GPU Tesla T4). YOLOv8 đảm nhiệm việc quét từng khung hình video để xuất ra các hộp giới hạn (bounding boxes) cùng điểm độ tin cậy (confidence scores).  
-  **Theo dõi đối tượng (Tracking):** Thuật toán ByteTrack được tích hợp để nhận dữ liệu đầu ra từ YOLOv8. ByteTrack sẽ tận dụng cả các hộp giới hạn có độ tin cậy thấp và sử dụng thuật toán Hungarian kết hợp chỉ số IoU để liên kết đối tượng giữa các khung hình liên tiếp, giúp cấp và duy trì ID cố định cho từng phương tiện.  
-  **Tính toán vận tốc:** Áp dụng kỹ thuật biến đổi phối cảnh (Perspective Transformation) nhằm ánh xạ tọa độ điểm ảnh sang tọa độ thế giới thực giả lập (Bird's Eye View) để triệt tiêu độ méo phối cảnh. Hệ thống sau đó dựa vào sự thay đổi vị trí của đối tượng (quãng đường) qua một số khung hình (thời gian) để tính toán vận tốc tức thời theo đơn vị km/h.

## 4. Kết quả
Dự án đã đạt được các kết quả như sau:
- **Hiệu suất phát hiện:** Trên tập kiểm định, mô hình YOLOv8n đạt Precision 0.934, Recall 0.795, mAP@50 đạt 0.884 và mAP50–95 đạt 0.718. Mô hình nhận diện cực kỳ chính xác nhóm đối tượng car và motorbike.  
- **Tốc độ xử lý:** Hệ thống YOLOv8 + ByteTrack có thể xử lý video với tốc độ trung bình từ 14 - 25 FPS. Qua thử nghiệm, YOLOv8 (22.4 FPS) cho tốc độ xử lý nhanh hơn khoảng 1.5 lần so với bản YOLOv12 (13 FPS) trên cùng bối cảnh, rất phù hợp cho ứng dụng thời gian thực.  
- **Xử lý che khuất (Occlusion):** Hệ thống có khả năng kiên nhẫn lưu trữ trạng thái bằng Kalman Filter và gán lại đúng ID cũ (ví dụ: khôi phục thành công ID sau 13 frames bị vật cản che lấp). Tuy nhiên, vẫn xuất hiện hiện tượng chuyển đổi sai ID nếu phương tiện bị che khuất quá lâu (trên 16 frames) hoặc bất ngờ thay đổi vận tốc trong vùng tối.  
- **Đo vận tốc:** Hệ thống hiển thị các thông tin gồm bounding box, định danh ID, độ tin cậy và vận tốc di chuyển rát trực quan
