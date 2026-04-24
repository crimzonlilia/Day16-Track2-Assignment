# Lab 16: Cloud AI Environment Setup - Báo Cáo Thực Hành
Họ tên: Nguyễn Thị Diệu Linh
MSHV: 2A202600209

## Phương Án Triển Khai: CPU Fallback với LightGBM

### 1. Lý Do Sử Dụng CPU Thay GPU

Tài khoản GCP mới không được cấp quota GPU (mặc định = 0). Yêu cầu tăng quota GPU T4 bị từ chối do tài khoản chưa đủ lịch sử thanh toán. Thay vì chờ xét duyệt (có thể 24 giờ trở lên), dự án đã chuyển sang **phương án dự phòng (Phần 7 trong README)**: triển khai ML benchmark trên instance CPU cao cấp (`e2-standard-4`) thay vì GPU.

---

## 2. Kết Quả Benchmark LightGBM

**Dataset:** Credit Card Fraud Detection (284,807 giao dịch, 31 features)  
**Model:** LightGBM (Gradient Boosting) 

### Metrics Chi Tiết:

| Metric | Giá trị |
|---|---|
| **Data Load Time** | 2.73 giây |
| **Training Time** | 2.08 giây |
| **AUC-ROC** | 0.9250 |
| **Accuracy** | 0.9993 |
| **F1-Score** | 0.7935 |
| **Precision** | 0.8488 |
| **Recall** | 0.7449 |
| **Inference Latency (per row)** | 0.05 ms |
| **Throughput** | 20,443 rows/sec |


---

## 3. So Sánh CPU vs GPU (Lý Thuyết)

Mặc dù không có quota GPU để test thực tế, dựa trên ước tính từ README:

| Khía Cạnh | CPU `e2-standard-4` | GPU `n1-standard-4` + T4 |
|---|---|---|
| **Machine Type** | `e2-standard-4` | `n1-standard-4` |
| **vCPU / RAM** | 4 vCPU, 16 GB | 4 vCPU, 15 GB |
| **GPU** | Không | 1x NVIDIA T4 |
| **Chi phí/giờ** | ~$0.134 | ~$0.54 (~$0.35 GPU + $0.19 VM) |
| **Training Time (ước tính)** | 2.08 giây | < 1 giây (GPU accelerated) |
| **Dùng ngay?** |  Có (không cần quota) |  Chờ quota 24h+ |

**Kết luận:** Phương án CPU không chỉ khả thi ngay mà còn rẻ hơn ~74% so với GPU. Cho workload như LightGBM (không yêu cầu matrix operations phức tạp như DNN), hiệu suất CPU đã đủ.
