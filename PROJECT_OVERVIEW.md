# Tổng quan Dự án: MALLORN Astronomical Classification Challenge

## 1. Mục tiêu Dự án
Mục tiêu chính của dự án là phát triển các thuật toán học máy để **phân loại các sự kiện Tidal Disruption Events (TDEs)** từ dữ liệu đường cong ánh sáng (lightcurves) mô phỏng cho kính thiên văn LSST (Legacy Survey of Space and Time).

*   **Nhiệm vụ:** Phân loại nhị phân (Binary Classification).
    *   `1`: Là sự kiện TDE (Sao bị hố đen xé toạc).
    *   `0`: Không phải TDE (Các loại biến quang khác như Supernovae, AGN, v.v.).
*   **Bối cảnh:** LSST sẽ phát hiện lượng lớn các sự kiện thiên văn biến đổi (transients). Việc phân loại tự động dựa trên đường cong ánh sáng là cần thiết để chọn lọc đối tượng cho các quan sát theo dõi chuyên sâu, vì nguồn lực kính thiên văn quang phổ có hạn.

## 2. Đánh giá (Evaluation)
*   **Metric:** **F1 Score**.
*   **Lý do:** Dữ liệu bị mất cân bằng nghiêm trọng (TDE hiếm hơn nhiều so với các loại khác), nên F1 Score được chọn để cân bằng giữa Precision (độ chính xác) và Recall (độ nhạy).

## 3. Cấu trúc Dữ liệu

### 3.1. Tổ chức Thư mục
Dữ liệu được chia thành 20 phần (splits) để dễ quản lý:
*   `split_01/` đến `split_20/`: Mỗi thư mục chứa các file CSV dữ liệu đường cong ánh sáng (`full_lightcurves.csv`) cho tập train và test tương ứng.

### 3.2. Các File Chính

#### A. File Log (`train_log.csv` và `test_log.csv`)
Chứa thông tin meta (metadata) về các đối tượng thiên văn.
*   `object_id`: Định danh duy nhất (tên tiếng Elvish).
*   `Z`: Redshift (Độ dịch chuyển đỏ).
*   `Z_err`: Sai số của Redshift (chỉ có trong tập test).
*   `EBV`: Hệ số tắt dần (Extinction coefficient) - dùng để hiệu chỉnh độ sáng bị bụi che khuất.
*   `SpecType`: Loại quang phổ thực tế (chỉ có trong tập train).
*   `target`: Nhãn mục tiêu (chỉ có trong tập train). `1` = TDE, `0` = Khác.
*   `split`: Tên thư mục chứa dữ liệu lightcurve chi tiết của đối tượng này.

#### B. File Lightcurves (`[train/test]_full_lightcurves.csv`)
Nằm trong các thư mục `split_XX`, chứa dữ liệu chuỗi thời gian quan sát.
*   `object_id`: Định danh đối tượng.
*   `Time (MJD)`: Thời gian quan sát (Modified Julian Date).
*   `Flux`: Độ sáng đo được (đơn vị microjansky - μJy). Cần được hiệu chỉnh bằng `EBV`.
*   `Flux_err`: Độ không đảm bảo đo của Flux.
*   `Filter`: Kính lọc màu sử dụng (u, g, r, i, z, y). Mỗi filter là một dải bước sóng khác nhau.

#### C. File Nộp bài (`sample_submission.csv`)
Định dạng file kết quả cần nộp.
*   `object_id`: ID của đối tượng trong tập test.
*   `prediction`: Dự đoán nhãn (`0` hoặc `1`).

## 4. Các loại đối tượng trong dữ liệu
Dữ liệu bao gồm các loại biến quang ngoài thiên hà (extragalactic transients):
*   **Mục tiêu quan tâm:** TDEs.
*   **Các lớp khác (Nhiễu):** SN Ia, SN Ib/c, SN II, SLSN, AGN, v.v.

## 5. Lưu ý Xử lý Dữ liệu
*   **Hiệu chỉnh Flux:** Giá trị `Flux` cần được "de-extincted" (khử tắt dần) sử dụng giá trị `EBV` từ file log.
*   **Dữ liệu thưa thớt:** Do đặc điểm quan sát của LSST, các điểm dữ liệu có thể cách xa nhau về thời gian (gaps).
*   **Flux âm:** Có thể xảy ra do phép trừ nền (background subtraction), đặc biệt với AGN.
