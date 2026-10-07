# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Ngo Minh Thu |
| MSSV | 2A202602679 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/ngominhthu09/K4-L3-DAY21-NgoMinhThu-2A202602679-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Cấu hình thứ ba có F1 cao nhất (0.7149) và vượt ngưỡng 0.65. Cấu hình thứ hai có accuracy cao nhất (0.8780) nhưng F1 thấp hơn, chứng tỏ accuracy không đủ để chọn mô hình khi dữ liệu mất cân bằng. Lần đầu dùng ít cây, learning rate thấp và cây nông nên F1 chỉ đạt 0.6051. Khi learning rate nhỏ thường phải tăng số cây; cấu hình được chọn dùng 200 cây, learning rate 0.1 và độ sâu 5.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ 24,8% mẫu thuộc lớp thu nhập trên 50K nên dữ liệu mất cân bằng. Mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy 0.752 nhưng F1 bằng 0 vì không phát hiện trường hợp thu nhập cao nào. F1 của lớp dương kết hợp precision và recall, phản ánh cả độ chính xác lẫn khả năng tìm đủ mẫu dương. Quality gate dùng F1 để chặn mô hình có accuracy cao nhưng bỏ sót lớp cần quan tâm. Lab gọi `f1_score(y_eval, preds)` trực tiếp; không dùng weighted hoặc macro vì phép gộp có thể che khuất hiệu năng kém trên lớp dương, nhất là weighted F1 bị lớp đa số chi phối.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow SQLite báo lỗi import | SQLAlchemy 2.1 đã bỏ API mà MLflow 2.13 sử dụng | Ghim `SQLAlchemy>=2.0,<2.1` trong requirements và cài lại môi trường. |
| Test tạo artifact trong thư mục repo | `train()` ghi đường dẫn tương đối `outputs/` và `models/` | Dùng `monkeypatch.chdir(tmp_path)` để cô lập file của từng test. |
| Accuracy và F1 xếp hạng mô hình khác nhau | Lớp thu nhập cao chỉ chiếm 24,8% | Chọn cấu hình theo F1 của lớp dương và giữ accuracy để tham khảo. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi tăng từ 22.361 lên 44.722 mẫu, F1 tăng 0.0205 và accuracy tăng 0.0080. Mức tăng vừa phải là hợp lý vì batch mới cùng nguồn và phân phối; giá trị chính của Bước 3 là pipeline tự huấn luyện và triển khai lại từ một commit dữ liệu.
