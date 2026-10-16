# Báo cáo bài tập: Kiểm thử hiệu năng với Apache JMeter

- **Họ và tên:** Phạm Văn Chiến
- **MSSV:** 23010293
- **Ngày thực hiện:** 16/10/2026

---


## 1. Giới thiệu về JMeter

Apache JMeter là công cụ mã nguồn mở viết bằng Java, dùng để **kiểm thử hiệu năng (performance/load testing)** và kiểm thử chức năng cho web (HTTP/HTTPS), REST API, SOAP, FTP, JDBC (cơ sở dữ liệu), JMS...

Ý tưởng chính: giả lập nhiều người dùng ảo (virtual users) cùng gửi request đến hệ thống, sau đó đo thời gian phản hồi, thông lượng (throughput), tỉ lệ lỗi.

### Các thành phần cơ bản

| Thành phần | Vai trò |
|---|---|
| **Test Plan** | Gốc của kịch bản kiểm thử, chứa mọi thành phần khác |
| **Thread Group** | Cấu hình số người dùng ảo (threads), thời gian tăng dần (ramp-up), số vòng lặp (loop count) |
| **Sampler** (HTTP Request...) | Request thực sự gửi tới server |
| **Config Element** (HTTP Request Defaults, Header Manager) | Cấu hình dùng chung cho các sampler |
| **Assertion** | Kiểm tra kết quả trả về đúng hay sai (mã trạng thái, nội dung) |
| **Listener** (View Results Tree, Summary/Aggregate Report) | Hiển thị và lưu kết quả |
| **Timer, Logic Controller, Pre/Post Processor** | Tạo độ trễ, điều khiển luồng, trích xuất dữ liệu |

### Các chỉ số quan trọng

- **Samples**: số request đã gửi
- **Average / Median / 90% Line / 95% Line / 99% Line**: thời gian phản hồi (ms)
- **Min / Max**: thời gian phản hồi nhỏ nhất / lớn nhất
- **Error %**: tỉ lệ request lỗi
- **Throughput**: số request xử lý được mỗi giây

### Tài liệu tham khảo

- Trang chủ: https://jmeter.apache.org
- Hướng dẫn "Getting Started" / "Building a Web Test Plan": https://jmeter.apache.org/usermanual/
- Tải JMeter: https://jmeter.apache.org/download_jmeter.cgi

---

## 2. Cài đặt môi trường

1. Cài **Java JDK 8 trở lên** (khuyến nghị 11 hoặc 17). Kiểm tra:
   ```bash
   java -version
   ```
2. Tải JMeter (file `.zip` Binaries), giải nén.
3. Chạy giao diện: `bin/jmeter.bat` (Windows) hoặc `bin/jmeter.sh` (Linux/macOS).

<img width="1517" height="848" alt="Screenshot 2026-10-16 074248" src="https://github.com/user-attachments/assets/c11bff4a-82ff-4a80-8b67-360427314d52" />

> `![Hình 1](images/01-cai-dat.png)`

---

## 3. Hệ thống được kiểm thử

- **Đối tượng:** REST API công khai `https://jsonplaceholder.typicode.com` (API giả lập dùng cho học tập).
- **Lý do chọn:** miễn phí, không cần đăng nhập, phù hợp để thực hành mà không ảnh hưởng hệ thống thật.
- **Lưu ý:** chỉ dùng số lượng người dùng nhỏ, không tạo tải lớn lên dịch vụ công cộng.

---

## 4. Kịch bản kiểm thử

File kịch bản: [`jmeter-demo.jmx`](jmeter-demo.jmx)

### Cấu hình Thread Group

| Tham số | Giá trị |
|---|---|
| Number of Threads (users) | 10 |
| Ramp-up period | 5 giây |
| Loop Count | 5 |
| Tổng request dự kiến | 10 users × 5 vòng × 3 request = **150** |

### Các request

| STT | Tên | Method | Đường dẫn | Assertion |
|---|---|---|---|---|
| 1 | GET danh sách posts | GET | `/posts` | Mã phản hồi = 200 |
| 2 | GET chi tiết post số 1 | GET | `/posts/1` | Body chứa `userId` |
| 3 | POST tạo post mới | POST | `/posts` | Mã phản hồi = 201 |

Body của request POST:

```json
{"title":"jmeter test","body":"hello","userId":1}
```

### Listener sử dụng

View Results Tree, Summary Report, Aggregate Report.

<img width="1517" height="838" alt="Screenshot 2026-10-16 074554" src="https://github.com/user-attachments/assets/9582abb9-cc2c-403a-a78f-d898aa30fa72" />


> `![Hình 3](images/03-thread-group.png)`
<img width="1512" height="850" alt="Screenshot 2026-10-16 074747" src="https://github.com/user-attachments/assets/8a36e7c2-6056-4652-9ed9-6aaf3361798c" />

> `![Hình 4](images/04-post-request.png)`

---

## 5. Thực hiện kiểm thử

Chạy bằng giao diện: mở file `.jmx` → bấm nút **Start** (▶).

Hoặc chạy bằng dòng lệnh (chế độ non-GUI, khuyến nghị khi đo hiệu năng) và sinh báo cáo HTML:

```bash
jmeter -n -t jmeter-demo.jmx -l results/result.jtl -e -o results/html-report
```

<img width="1516" height="861" alt="Screenshot 2026-10-16 075020" src="https://github.com/user-attachments/assets/542c99a3-9db5-4f61-9596-b6af4e1c1967" />

> `![Hình 5](images/05-view-results-tree.png)`

---

## 6. Kết quả

### 6.1 Summary Report / Aggregate Report
<img width="1508" height="852" alt="Screenshot 2026-10-16 075108" src="https://github.com/user-attachments/assets/6f208903-720f-4e9f-968b-eb7f72293867" />

> `![Hình 6](images/06-summary-report.png)`
<img width="1528" height="863" alt="Screenshot 2026-10-16 075129" src="https://github.com/user-attachments/assets/408d4af6-a276-4997-8ead-8764318f1496" />

> `![Hình 7](images/07-aggregate-report.png)`

Điền số liệu từ kết quả thực tế của bạn:

| Request | Samples | Average (ms) | 90% Line (ms) | Max (ms) | Error % | Throughput (req/s) |
|---|---|---|---|---|---|---|
| GET /posts | | | | | | |
| GET /posts/1 | | | | | | |
| POST /posts | | | | | | |
| **TOTAL** | | | | | | |

### 6.2 Báo cáo HTML (Dashboard)



## 7. Nhận xét và đánh giá



## 8. Kết luận

Tóm tắt những gì đã học được: cách dựng test plan, cấu hình tải, dùng assertion, đọc báo cáo, và kết quả kiểm thử hệ thống mẫu.

---



