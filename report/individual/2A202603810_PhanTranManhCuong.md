# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

- **MSSV:** 2A202603810
- **Họ và tên:** Phan Trần Mạnh Cường
- **SEED:** 3381

## 1. Bằng chứng

| Tệp | Trạng thái |
|---|---|
| [`day01_lab.ipynb`](day01_lab.ipynb) | Đã chép notebook bản sao vào thư mục nộp. Output cũ còn nguyên; cell thử thách và bonus mới chưa chạy. |
| [`lab1_report.json`](lab1_report.json) | Đã chép báo cáo kết quả trước khi thêm các case bonus. |
| [`hand_calc.jpg`](hand_calc.jpg) | Chưa có; tôi sẽ bổ sung ảnh viết tay sau. |

## 2. Kết quả chính

| Mô hình | Precision | Recall | F1 test | F1 SUB | p95 (ms/tin) | USD/1k tin |
|---|---:|---:|---:|---:|---:|---:|
| BoW + LogReg | 1.000 | 0.886 | 0.940 | 0.919 | 0.063 | 0 |
| BoW + MLP | 1.000 | 0.932 | 0.965 | 0.947 | 0.232 | 0 |
| Encoder + kNN | 0.895 | 0.966 | 0.929 | 0.974 | 18.953 | 0 |
| Encoder + LogReg | 0.977 | 0.955 | 0.966 | 1.000 | 7.133 | 0 |
| Encoder + MLP head | 0.976 | 0.943 | 0.960 | 1.000 | 6.970 | 0 |
| LLM zero-shot (Gemini) | 1.000 | 1.000 | 1.000 | 1.000 | 861.373 | 0.0154 |
| LLM few-shot (Gemini) | 1.000 | 1.000 | 1.000 | 1.000 | 1086.632 | 0.0718 |

*Số liệu chép từ `lab1_report.json`; kết quả LLM trong file được đo trên SUB. Các cell đã sửa cho thử thách và bonus chưa được chạy lại.*

## 3. Dự đoán và thực tế

- **F1 Logistic Regression:** tôi đoán 0.80; kết quả test là 0.940 (F1 SUB 0.919). Mô hình học được tín hiệu từ từ vựng khá rõ nên tốt hơn dự đoán.
- **Few-shot:** tôi đoán F1 sẽ tăng; zero-shot và few-shot đều đạt F1 SUB 1.000 trên kết quả đã lưu. Few-shot tốn khoảng 4.66 lần chi phí mỗi 1.000 tin, nên không cải thiện điểm trong lần đo này.
- **Case khó:** tôi đoán nhóm né từ khóa sẽ khó nhất; với BoW + LogReg, nhóm spam giọng tự nhiên đúng 0% còn nhóm né từ khóa đúng 17%. Dữ liệu stress test không có kết quả LLM nên chưa kết luận nhóm khó nhất của LLM.

## 4. Phân tích 50 case khó

BoW + LogReg tụt nhiều nhất: F1 SUB 0.919 xuống F1 case khó 0.286. Trên bảng theo nhóm, mô hình đúng 0% nhóm spam giọng tự nhiên, 12% nhóm thao túng/lừa đảo và 17% nhóm né từ khóa. Embedding + kNN giữ F1 case khó cao nhất trong các mô hình đã chạy: 0.780.

Hai ví dụ mà cả năm mô hình đều sai: case #12 giả giọng người mới quen để dẫn sang lời mời hẹn hò; case #13 giả làm thông báo ảnh sự kiện sẵn sàng để xem/đặt mua. Cả hai dùng lời văn tự nhiên và thiếu từ khóa spam quen thuộc, khiến mô hình dựa vào từ hoặc mẫu ngữ nghĩa học được nhận nhầm là tin bình thường.

## 5. Thử thách cá nhân

Với SEED 3381, đề được chọn là **Embedding: so sánh kNN với k = 1, 5, 25**. Tôi đã thêm code tính precision, recall, F1 trên tập test và in k có F1 cao nhất. Tôi chưa chạy cell theo yêu cầu không chạy lại notebook, nên chưa ghi số F1 cụ thể. k nhỏ dễ nhạy với hàng xóm nhiễu; k lớn ổn định hơn nhưng có thể làm loãng tín hiệu của lớp ít hơn. Khi chọn k cho sản phẩm, nên chọn trên validation để giữ test độc lập.

## 6. Phương án cho sản phẩm thật

Chọn **Encoder + LogReg** làm phương án chính: F1 test 0.966, F1 SUB 1.000, p95 7.133 ms/tin và throughput đo được khoảng 1.350 tin/s, đáp ứng ngưỡng p95 < 50 ms và tốc độ trung bình khoảng 116 tin/s. Chi phí API trong benchmark là 0 USD/1k tin; cần đo tải thực tế và chi phí hạ tầng trước khi triển khai.

Nếu dùng LLM zero-shot cho toàn bộ 10 triệu tin/ngày, mức giá trong báo cáo tương ứng khoảng **$154/ngày, $4,624 cho 30 ngày**; p95 861 ms vượt SLA. Few-shot khoảng **$21,544/30 ngày** và p95 còn cao hơn. Phương án lai là dùng Encoder + LogReg trước, đưa các tin có xác suất sát ngưỡng sang bước rà soát riêng; chỉ gọi LLM cho phần nhỏ nếu quota, độ trễ và ngân sách cho phép.

## 7. AI log và điều đã học

| Phần | Hỏi AI | AI hỗ trợ | Kiểm chứng |
|---|---|---|---|
| TODO 1–3 | Cách hoàn thiện vectorize, sigmoid, logistic regression, metrics và MLP. | Đưa hướng dẫn/code và giải thích các bước. | `lab1_report.json` ghi 10/10 kiểm tra code đạt. |
| Gemini API | Cách chọn Gemini và xử lý lỗi quota 429. | Giải thích cấu hình provider và cách giảm số tin gọi API. | Notebook dùng Gemini; các cell zero/few-shot hiện lấy 5 tin; không đưa API key vào báo cáo. |
| Thử thách #4 và bonus | So sánh kNN ở ba giá trị k, thêm case khó, in case bị phân loại sai. | Thêm code và 5 case tổng hợp vào notebook. | Chưa chạy cell mới; kết quả F1 và lỗi bonus còn chờ chạy. |

Tôi hiểu BoW dựa nhiều vào từ xuất hiện, còn embedding có thể bắt được ngữ nghĩa nhưng vẫn sai khi văn bản tự nhiên hoặc ngữ cảnh lạ. Điểm test thường không thay thế stress test; khi chọn mô hình cần cân bằng F1, độ trễ, chi phí và lỗi trên các nhóm khó.
