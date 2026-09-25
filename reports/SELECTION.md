# Vì sao chọn lô này?

## 1. Năm frame ưu tiên nếu chỉ có ngân sách rà năm ảnh

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, tôi ưu tiên các
frame có điểm bất định cao, khả năng chứa lỗi bỏ sót hoặc sai box, đồng
thời cân nhắc chi phí rà nhãn.

  -----------------------------------------------------------------------
  Thứ tự                  Frame                   Lý do ưu tiên
  ----------------------- ----------------------- -----------------------
  1                       frame_0099.jpg          Model bỏ sót 5 xe so
                                                  với pre-label ban đầu,
                                                  sau sửa tăng từ 13 lên
                                                  18 box. Đây là ảnh có
                                                  nhiều lỗi bỏ sót cần bổ
                                                  sung nhãn.

  2                       frame_0312.jpg          Có lỗi bỏ sót xe tải
                                                  lớn. Sau rà nhãn có
                                                  thêm box và chỉnh sửa
                                                  box AI.

  3                       frame_0107.jpg          Sau rà nhãn thêm 6 box,
                                                  cho thấy model chưa bao
                                                  phủ đầy đủ đối tượng
                                                  trong ảnh.

  4                       frame_0182.jpg          Sau rà nhãn thêm 6 box,
                                                  là trường hợp có giá
                                                  trị để cải thiện khả
                                                  năng phát hiện.

  5                       frame_0331.jpg          Có nhiều box cần xử lý:
                                                  xóa 5 box sai và thêm 4
                                                  box mới, thể hiện lỗi
                                                  cả false positive và
                                                  false negative.
  -----------------------------------------------------------------------

Ba frame thuộc lô 12 ảnh model chọn:

-   `frame_0099.jpg`: pre-label 13 box, sau sửa 18 box, thêm 5 box.
-   `frame_0312.jpg`: pre-label 13 box, sau sửa 14 box, có chỉnh sửa,
    xóa và thêm box.
-   `frame_0331.jpg`: pre-label 20 box, sau sửa 19 box, xóa 5 box sai và
    thêm 4 box.

Một frame có điểm cao nhưng không chọn có thể là ảnh gần trùng với các
ảnh đã chọn. Việc chọn thêm ảnh tương tự có thể làm tăng chi phí rà nhãn
nhưng không tăng nhiều thông tin mới cho mô hình.

## 2. Điều phép chọn này chưa chứng minh

Active Learning chỉ giúp ưu tiên các ảnh có khả năng chứa thông tin hữu
ích dựa trên điểm chọn mẫu. Điểm cao không đảm bảo chắc chắn ảnh đó sẽ
giúp mô hình cải thiện nhiều sau fine-tune.

Hiệu quả cuối cùng cần được đánh giá bằng việc huấn luyện lại mô hình và
đo trên cùng tập kiểm thử.
