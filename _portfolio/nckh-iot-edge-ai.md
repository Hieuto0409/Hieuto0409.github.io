---
title: "Hệ thống IoT & Edge AI Sàng lọc Tim mạch và Hô hấp (NCKH)"
excerpt: "Hệ thống IoT hỗ trợ sàng lọc không xâm lấn, cảnh báo nguy cơ các bệnh liên quan đến tim mạch và hô hấp dựa trên phân tích tín hiệu PPG, ECG và công nghệ Edge AI trên ESP32-S3."
collection: portfolio
header:
  teaser: None
demo_url: /images/NCKH.mp4
link: https://drive.google.com/drive/folders/1_mGJFauYfuQHPGsLxzTOmc2D74vZTejX?hl=vi
stack:
  - ESP32-S3
  - Edge AI / TinyML
  - Edge Impulse
  - Scikit-learn
  - DSP / SQI
  - PPG & ECG
  - ThingsBoard IoT
---

## Mô tả

Đề tài Nghiên cứu Khoa học Sinh viên: _"Thiết kế và thi công hệ thống IoT hỗ trợ sàng lọc không xâm lấn cảnh báo nguy cơ các bệnh liên quan đến tim và hô hấp dựa trên phân tích tín hiệu PPG và công nghệ Edge AI"_.

Hệ thống tích hợp cảm biến quang học **MAX30102** (thu tín hiệu ánh sáng đỏ và hồng ngoại RED/IR ở đầu ngón tay ở tần số 200 Hz) cùng cảm biến điện tâm đồ một kênh **AD8232** (thu tín hiệu ECG ở tần số 500 Hz), xử lý thời gian thực ngay tại biên trên vi điều khiển **ESP32-S3**, hiển thị trực tiếp qua màn hình cảm ứng **TJC** và đồng bộ dữ liệu phiên đo (60 giây) lên nền tảng đám mây **ThingsBoard** qua giao thức HTTPS.

Điểm nổi bật của dự án là việc kết hợp chuỗi xử lý tín hiệu số – kiểm tra chất lượng tín hiệu (**DSP/SQI**) với các mô hình học máy tại biên (**Edge AI**) trên các đặc trưng biến thiên khoảng nhịp sinh lý (**HRV/PRV**) để tính toán nhịp tim (HR), nhịp mạch (PR), nồng độ bão hòa oxy trong máu SpO2 và hỗ trợ sàng lọc, cảnh báo 4 trạng thái:

- **Căng thẳng tâm lý cấp tính (Stress)** từ biến thiên khoảng mạch PPG.
- **Rối loạn nhịp / Nghi ngờ rung nhĩ (AF / non-AF)** từ biến thiên khoảng RR trên tín hiệu ECG.
- **Cảnh báo nồng độ oxy trong máu thấp (SPO2 thấp)** dựa trên tỷ số quang học AC/DC và cổng kiểm tra chất lượng tín hiệu.
- **Bình thường (Không phát hiện cảnh báo)** khi tất cả các nhánh đo đều đạt chuẩn chất lượng tín hiệu và âm tính, hoặc **Yêu cầu đo lại** khi phát hiện nhiễu chuyển động, mất tiếp xúc hay tín hiệu không đủ độ tin cậy.

## Công nghệ

- **Phần cứng & Cảm biến:** ESP32-S3 (DevKitC-1 N16R8), MAX30102 (PPG/SpO2), AD8232 (ECG 1 kênh), màn hình cảm ứng TJC UART, RTC DS3231.
- **AI & Khoa học dữ liệu:** Python, Scikit-learn (Logistic Regression, StandardScaler), Edge Impulse (Multilayer Perceptron – MLP, INT8 Quantization), TinyML, C/C++ Embedded Inference.
- **Xử lý tín hiệu (DSP & SQI):** Bộ lọc IIR khử trôi đường nền & làm mượt PPG, bộ lọc Notch 50 Hz & Bandpass QRS (5–20 Hz), thuật toán bắt đỉnh R (biến thể Pan–Tompkins kết hợp cơ chế refractory & search-back), cổng đánh giá chất lượng tín hiệu (SQI/Contact Gate).
- **Bộ dữ liệu nghiên cứu:** WESAD (Wearable Stress and Affect Detection), MIT-BIH Atrial Fibrillation Database (AFDB), PhysioNet Pulse Transit Time PPG Dataset.
- **Nền tảng IoT:** ThingsBoard Cloud, HTTPS Telemetry Queue.

## Vai trò

Trong dự án, tôi chịu trách nhiệm chính về **nghiên cứu cơ sở lý thuyết AI – sinh lý học, thiết kế quy trình trích xuất đặc trưng và trực tiếp huấn luyện, tối ưu hóa các mô hình học máy (Edge AI)** cho các module phân loại của hệ thống, cũng như viết các chuyên đề báo cáo và báo cáo tổng kết đề tài:

1. **Nghiên cứu và huấn luyện Module AI nhận diện Căng thẳng (Stress) từ tín hiệu PPG:**
   - Nghiên cứu cơ sở sinh lý của phản ứng căng thẳng cấp tính thông qua hệ thần kinh tự chủ (ANS) và biểu hiện trên khoảng cách đỉnh mạch (PPI) cùng biến thiên nhịp mạch (PRV).
   - Chuẩn bị, làm sạch và phân chia độc lập theo đối tượng (subject-wise split) trên bộ dữ liệu **WESAD** nhằm tránh rò rỉ dữ liệu giữa tập huấn luyện và kiểm thử.
   - Thiết kế và trích xuất **14 đặc trưng PRV miền thời gian** trên cửa sổ 60 giây (`mean_hr_bpm`, `std_hr_bpm`, `min_hr_bpm`, `max_hr_bpm`, `mean_pp_ms`, `median_pp_ms`, `sdnn_ms`, `rmssd_ms`, `sdsd_ms`, `pnn20_pct`, `pnn50_pct`, `cvnn`, `beat_count`, `valid_rr_ratio`).
   - Xây dựng và huấn luyện mô hình **Logistic Regression** kết hợp chuẩn hóa **StandardScaler**, đạt **Accuracy 93,33%**, **F1-score 0,9375**, **Recall lớp Stress 100%** và **ROC AUC 1,0000** trên tập Test độc lập; xuất toàn bộ tham số chuẩn hóa và trọng số sang mã C++ để suy luận trực tiếp trên ESP32-S3.

2. **Nghiên cứu và huấn luyện Module Edge AI sàng lọc Rung nhĩ (AF / non-AF) từ tín hiệu ECG:**
   - Nghiên cứu đặc điểm rối loạn nhịp thất bất quy tắc trên chuỗi khoảng đỉnh R (RR intervals) và xử lý dữ liệu từ bộ cơ sở dữ liệu chuẩn **MIT-BIH AFDB** với các cửa sổ ECG 30 giây.
   - Xây dựng bộ **9 đặc trưng thống kê khoảng nhịp RR** (`mean_rr`, `median_rr`, `sdnn`, `rmssd`, `pnn50`, `cv_rr`, `iqr_rr`, `min_rr`, `max_rr`) kết hợp các ngưỡng cổng kiểm tra độ tin cậy runtime (coverage, tỷ lệ RR ngắn, khoảng trống tối đa, tỷ lệ search-back).
   - Huấn luyện mô hình mạng nơ-ron truyền thẳng nhiều lớp (**MLP**) trên nền tảng **Edge Impulse** và thực hiện **lượng tử hóa INT8 (Quantization)** để tối ưu bộ nhớ và tốc độ xử lý tại biên. Mô hình INT8 đạt **Accuracy 98,1%** (Loss 0,05, AUC 0,98) trên tập Internal Validation và **Accuracy 92,72%** (F1-score 0,95, AUC 0,96) trên 6.912 cửa sổ kiểm thử ngoại tuyến.

# Video demo

<video controls width="100%">
  <source src="/images/NCKH.mp4" type="video/mp4">
</video>

## Tài liệu & Báo cáo

- [Thư mục Báo cáo NCKH & Tài liệu chuyên đề (Google Drive)](https://drive.google.com/drive/folders/1_mGJFauYfuQHPGsLxzTOmc2D74vZTejX?hl=vi)
