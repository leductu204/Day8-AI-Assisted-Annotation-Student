# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Lê Đức Tú

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Pool chưa gán nhãn và tập kiểm thử được chia theo trục thời gian, có
vùng đệm ở giữa nhằm hạn chế việc các ảnh gần nhau xuất hiện ở cả tập
huấn luyện và kiểm thử.

Nếu chia ngẫu nhiên, các ảnh tương tự về bối cảnh có thể xuất hiện ở cả
hai tập, làm kết quả đánh giá có nguy cơ cao hơn so với khả năng tổng
quát thực tế.

## 2. Mô hình khởi đầu lạnh (cold start)

Vòng 0 sử dụng `yolov8n cold start (COCO car+bus+truck)` với
`yolov8n.pt`, không có ảnh huấn luyện bổ sung. Kết quả trên 20 ảnh test:

-   AP50: 0.7714
-   Precision: 0.9249
-   Recall: 0.4888
-   F1: 0.6396

Model có precision cao nhưng recall thấp, cho thấy xu hướng bỏ sót nhiều
xe.

Theo kích thước đối tượng: - Small recall: 0.1818 (66 box tham chiếu) -
Medium recall: 0.5473 (296 box tham chiếu) - Large recall: 0.561 (41 box
tham chiếu)

Điều này cho thấy nhóm xe nhỏ là nhóm khó phát hiện nhất.

## 3. Chiến lược chọn mẫu

Vòng chọn mẫu sử dụng chiến lược `uncertainty`.

Điểm chọn mẫu kết hợp: - độ bất định của model; - mức độ khác biệt giữa
các ảnh; - chi phí rà nhãn.

Điểm bất định giúp tìm các ảnh model chưa chắc chắn, nhưng không chứng
minh chắc chắn rằng ảnh đó sẽ cải thiện mô hình sau huấn luyện.

Các frame tiêu biểu: - `frame_0099.jpg`: phát hiện nhiều xe bị bỏ sót,
thêm 5 box. - `frame_0312.jpg`: có lỗi bỏ sót xe và cần chỉnh sửa. -
`frame_0331.jpg`: có cả false positive và false negative.

## 4. Các vòng học chủ động

### Vòng 1

Fine-tune: - Số ảnh train: 12 - Số box train: 205 - Epochs: 50 -
Strategy: uncertainty

Kết quả sửa nhãn: - Model đề xuất: 169 box - Accepted: 157 - Edited: 3 -
Deleted: 9 - Added: 45 - Final boxes: 205

So sánh với cold start:

  Chỉ số        Round 0   Round 1
  ----------- --------- ---------
  AP50           0.7714    0.4708
  Precision      0.9249       1.0
  Recall         0.4888     0.139
  F1             0.6396     0.244

Sau fine-tune, precision tăng nhưng recall giảm mạnh. Điều này cho thấy
mô hình vòng 1 trở nên thận trọng hơn và bỏ sót nhiều đối tượng hơn trên
tập test.

Theo kích thước: - Small recall: từ 0.1818 xuống 0.0 - Medium recall: từ
0.5473 xuống 0.125 - Large recall: từ 0.561 xuống 0.4634

Một nguyên nhân cần kiểm tra là tập fine-tune chỉ có 12 ảnh với phân bố
dữ liệu nhỏ, chưa đủ đại diện cho toàn bộ test set.

## 5. Kết luận và giới hạn

Vòng học chủ động đã hoàn thành quy trình: cold start → chọn mẫu → sửa
nhãn → fine-tune → đánh giá.

Tuy nhiên, kết quả vòng 1 giảm AP50 so với cold start. Cần kiểm tra: -
sự phù hợp giữa batch 12 ảnh và tập test; - chất lượng nhãn sau sửa; -
khả năng mô hình bị overfit do số lượng ảnh huấn luyện nhỏ.

Các nhóm nên tiếp tục rà thêm: 1. Xe nhỏ ở xa do recall nhóm small thấp.
2. Các cảnh nhiều xe chồng lấn hoặc điều kiện ánh sáng khó.

Giới hạn: - Tập test chỉ có 20 ảnh. - Có luật bỏ qua xe quá nhỏ. - Nhãn
tham chiếu do mô hình tạo chưa được rà thủ công.

Vì vậy kết quả cần được hiểu trong phạm vi thí nghiệm này.
