# NIDS Dựa Trên Phát Hiện Bất Thường cho Mạng Hạn Chế Tài Nguyên sử dụng Học Máy
## Tổng Quan 

Mục tiêu chính nghiên cứu này là khám phá và phát triển một NIDS dựa trên học máy hiệu quả nhưng nhẹ, phù hợp để triển khai trên phần cứng hạn chế tài nguyên. Các giải pháp NIDS truyền thống thường yêu cầu công suất xử lý và bộ nhớ đáng kể, khiến chúng không thực tế đối với các doanh nghiệp nhỏ hoặc thiết bị biên. Dự án này điều tra các thuật toán học máy khác nhau, cuối cùng phát triển một giao diện cân bằng giữa độ phát hiện và hiệu quả hoạt động.

Phương pháp là khả năng ứng dụng thực tế của các model dựa trên instance như KNN . Trong khi các mô hình  dựa trên cây như XGBoost đạt được độ chính xác cao trên tập dữ liệu CICIDS2017 được chọn trong quá trình huấn luyện, mô hình KNN đđhoajt động hiệu quả hơn khi prototype NIDS được xác thực với lưu lượng mạng được phát lại (các file pcap) trong môi trường mô phỏng.

## Tính Năng Chính
- **Tập Trung vào Tài Nguyên Hạn Chế:** Prototype NIDS cuối cùng được thiết kế với các thiết bị công suất thấp trên máy ảo unbuntu 4gb ram
- **Phát Hiện Dựa Trên Bất Thường:** Nhằm mục đích xác định các sai lệch so với hành vi mạng bình thường, cho phép phát hiện cả các mối đe dọa đã biết và mới.
- **Core Anomaly detection -  KNN:** Prototype NIDS hoạt động sử dụng mô hình K-Nearest Neighbors (KNN) làm chính
- **Phân Tích Lưu Lượng Thời Gian Thực:** Sử dụng `scapy` để bắt gói tin trực tiếp và trích xuất đặc trưng từ lưu lượng mạng.
- **Giao Diện GUI dễ sử dụng ** 
- **Xác Thực Thực Tế:** Prototype NIDS được kiểm tra test bằng cách phát lại các  lưu lượng mạng thực tế (các file pcap CICIDS2017) sử dụng `tcpreplay` trong môi trường kiểm nghiệm.
- **Ghi Log Cảnh Báo Có Cấu Trúc:** Các cảnh báo được tạo bởi NIDS được ghi log.

## Các Giai Đoạn Dự Án

Dự án được thực hiện theo các giai đoạn sau:

1.  **Tiền Xử Lý Dữ Liệu và Phân Tích Dữ Liệu Khám Phá (EDA):**

    - Làm sạch, chuyển đổi và phân tích toàn diện tập dữ liệu CICIDS2017.
    - Các bước bao gồm xử lý giá trị bị thiếu, loại bỏ trùng lặp, kỹ thuật đặc trưng, cân bằng lớp (SMOTE và random undersampling), và chuẩn hóa đặc trưng (RobustScaler).

2.  **Huấn Luyện Mô Hình Học Máy và Đánh Giá So Sánh:**

    - Huấn luyện và đánh giá nghiêm ngặt các mô hình học máy có giám sát (Random Forest, XGBoost, KNN) và không giám sát (Isolation Forest, K-Means).
    - Hiệu suất được đánh giá bằng các chỉ số như accuracy, precision, recall, F1-score, và tiêu thụ tài nguyên (thời gian huấn luyện, sử dụng CPU/bộ nhớ).
    - Điều chỉnh siêu tham số được thực hiện bằng `RandomizedSearchCV`.

3.  **Phát Triển và Xác Thực Prototype NIDS:**
    - Phát triển một prototype NIDS hoạt động trong Python, tích hợp mô hình học máy được chọn (KNN) vì hiệu suất thực tế của nó.
    - Script `prototype/nids_prototype_knn.py` chứa class `NetworkAnomalyDetector` để xử lý gói tin và trích xuất đặc trưng, và class `NetworkAnomalyGUI` cho giao diện người dùng.
    - Prototype được xác thực trên Raspberry Pi 5 bằng cách phát lại các file pcap CICIDS2017 sử dụng `tcpreplay` để mô phỏng các cuộc tấn công mạng (ví dụ: DoS, Port Scan, Botnet). Phương pháp này cho phép so sánh trực tiếp với các mẫu lưu lượng đã biết.

<br>


## Bắt Đầu
### Yêu Cầu 

- Python 3.11+
- pip (trình cài đặt gói Python)
- `libpcap-dev` (hoặc tương đương cho hệ điều hành của bạn, cần thiết cho Scapy để bắt gói tin)
  - Trên Debian/Ubuntu: `sudo apt-get install libpcap-dev`
  - Trên Fedora: `sudo dnf install libpcap-devel`
  - Trên macOS (với Homebrew): `brew install libpcap`

### Cài Đặt

1.  **Clone repository:**

    ```bash
    git clone https://github.com/anacletu/ml-intrusion-detection-cicids2017.git
    cd ml-intrusion-detection-cicids2017
    ```

2.  **Tạo và kích hoạt môi trường ảo :**
    pip install pandas numpy scikit-learn joblib scapy netifaces tk
 # nên chạy trên môi trường unbuntu hay những hdh tương đương, window và macOS cần thiết lập thư viện và sử dụng WSL do cần thư viện 
    ```bash
    python3 -m venv venv
    source venv/bin/activate  # window: venv\Scripts\activate
    ```

3.  ** Cài Python cần thiết:**
    ```bash
    pip install pandas numpy scikit-learn joblib scapy netifaces tk
    ```

### Chạy Prototype NIDS (dựa trên KNN)

Prototype NIDS nên dùng quyền root/administrator để bắt các gói tin mạng.

1.  **Di chuyển đến thư mục prototype:**

    ```bash
    cd prototype
    ```

2.  **Chạy script NIDS:**
    ```bash
    sudo python nids_prototype_knn.py
    ```
 
### Kiểm Thử NIDS với Các File PCAP

Để kiểm thử NIDS với lưu lượng đã ghi sẵn (ví dụ: từ tập dữ liệu CICIDS2017):

1.  **Lấy =PCAP:** Tải  pcap CICIDS2017 gốc từ [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html).đặt trong folder `pcaps/` 

2.  **Khởi động prototype NIDS** như mô tả ở trên

3.  **Sử dụng `tcpreplay` để gửi lưu lượng pcap.** Mở một cli terminal khác riêng.

- Kiểm tra ip máy và gói tin. Nếu không trùng sử dụng wireshark haytcpwire để chỉnh sửa gói hoặc thiết lập lại ip máy
- Để phát lại một file pcap (`Thurssday-WorkingHours.pcap`) lên một giao diện cụ thể ( `enps33`):

```bash
sudo tcpreplay -i eth0 Thursday-WorkingHours.pcap
```

## Kết Quả và Thảo Luận

## Công Việc cần hoàn thiện

- **Triển Khai và Đánh Giá Thực Tế:** Triển khai NIDS trong môi trường mạng thực tế hơn
- **Tổng Quát Hóa Đa Tập Dữ Liệu:** Áp dụng các kỹ thuật kỹ thuật đặc trưng và đánh giá hiệu suất mô hình trên các tập dữ liệu NIDS chuẩn khác (ví dụ: UNSW-NB15, CSE-CIC-IDS2018) để đánh giá khả năng tổng quát hóa và xác định các hạn chế của các mô hình được huấn luyện độc quyền trên CICIDS2017.
- **Cải Thiện Kỹ Thuật Đặc Trưng:** Liên tục tùy chỉnh kỹ thuật đặc trưng bằng cách kết hợp dữ liệu từ các nguồn đa dạng hoặc khám phá các đặc trưng thời gian tinh vi hơn để cải thiện độ chính xác phát hiện và giảm false positives.
