# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Lương Quang Huy |
| MSSV | 2A202602698 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/huyluong1910/K4-L3-DAY21-LuongQuangHuy-2A202602698-CI-CD-for-AI-Systems |
| Ngày nộp | 12/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 4 | 0.7182 | 0.8760 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=4`.

**Lý do:** Bộ này đạt `f1_score` cao nhất (0.7182) trên tập holdout. Đáng chú ý, lần 1 có accuracy cao nhất (0.8780) nhưng F1 chỉ đạt 0.7109. Vì dữ liệu mất cân bằng (chỉ 24.8% thuộc lớp >50K), mô hình lần 1 thiên về đoán lớp đa số nên accuracy cao nhưng bỏ sót nhiều mẫu dương tính. Cấu hình lần 3 với `max_depth=4` và `n_estimators=200` giúp học tốt các tương tác phi tuyến giữa học vấn, nghề nghiệp và giờ làm việc, cải thiện Recall và cân bằng tối ưu F1. Giảm learning rate xuống 0.05 và giới hạn 50 cây ở lần 2 khiến mô hình bị underfitting rõ rệt.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu mất cân bằng nghiêm trọng khi chỉ 24.8% số mẫu có thu nhập >50K USD/năm. Một mô hình ngây thơ luôn dự đoán "thu nhập thấp" vẫn đạt accuracy 75.2% dù hoàn toàn vô dụng trong thực tế. Do đó, accuracy tạo ra ảo tưởng về hiệu năng. Chỉ số `f1_score` trên lớp dương (>50K) là thước đo chuẩn xác vì nó tính trung bình điều hòa giữa Precision và Recall, phạt nặng cả việc dự đoán nhầm lẫn bỏ sót người thu nhập cao. Lab không dùng `average="macro"` hay `weighted` vì các cách tính này sẽ bị điểm số áp đảo của lớp đa số làm lu mờ, làm mất độ nhạy cảm cần thiết với lớp thiểu số mục tiêu.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Tạo tài nguyên AWS bị lỗi UnauthorizedOperation | AWS Organization SCP hạn chế vùng hoạt động | Cấu hình tường minh region ap-southeast-2 (Sydney) cho AWS CLI, S3, EC2 và DVC |
| Release job thất bại khi SSH vào EC2 | Secret SERVER_USER dính ký tự xuống dòng thừa | Thêm bước tiền xử lý lọc ký tự trống bằng tr -d '[:space:]' trước khi kết nối SSH |
| Script train báo lỗi UntrustedTypesFoundException | MLflow mặc định dùng skops từ chối serialize Decision Tree | Truyền tham số serialization_format=SERIALIZATION_FORMAT_CLOUDPICKLE vào mlflow.sklearn.log_model |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7182 | 0.8760 |
| Bước 3 (thêm `train_batch2`) | 0.7240 | 0.8780 |

**Nhận xét:** Khi tăng gấp đôi dữ liệu lên 44.722 mẫu, `f1_score` chỉ tăng nhẹ 0.0058 và accuracy tăng 0.0020 vì hai tập dữ liệu được chia ngẫu nhiên từ cùng một nguồn nên có chung phân phối; mô hình đã học gần hết đặc trưng từ batch đầu. Ý nghĩa cốt lõi của Bước 3 là chứng minh chu trình Continuous Training tự động hoạt động hoàn hảo: commit dữ liệu mới tự kích hoạt pipeline, kéo dữ liệu qua DVC, huấn luyện, qua quality gate và deploy lên VM mà không cần can thiệp thủ công.
