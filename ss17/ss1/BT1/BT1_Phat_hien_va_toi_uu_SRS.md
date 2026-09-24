# BT1 – PHÁT HIỆN VÀ TỐI ƯU HÓA ĐẶC TẢ SRS BỊ LỖI
**Dự án:** RikkeiLogistics – Phân hệ Tiếp nhận Yêu cầu Vận chuyển Bưu kiện
**Chuẩn tham chiếu:** IEEE 830-1998 (Unambiguous, Verifiable, Complete)

---

## PHẦN 1. BẢNG ĐỐI SOÁT 4 LỖI VI PHẠM IEEE 830

| # | Vị trí trong bản thảo | Nội dung lỗi | Loại vi phạm | Giải thích vi phạm | Hướng khắc phục |
|---|---|---|---|---|---|
| 1 | Mục 2.4 – các dòng "Định nghĩa: COD…", "Định nghĩa: Bưu kiện hỏa tốc…" | Bảng thuật ngữ đặt trong mục Ràng buộc | **Sai vị trí chương mục** | IEEE 830 quy định thuật ngữ, từ viết tắt phải nằm ở **1.3 Definitions, acronyms, abbreviations**. Mục 2.4 chỉ chứa ràng buộc (ngôn ngữ lập trình, chuẩn giao thức, giới hạn phần cứng…). Ngoài ra, định nghĩa "hỏa tốc = giao nhanh nhất có thể" không có ngưỡng thời gian nên không dùng làm căn cứ nghiệp vụ được. | Chuyển sang 1.3; định nghĩa "hỏa tốc" bằng thời gian cam kết cụ thể. |
| 2 | REQ-01 | "cực kỳ đẹp mắt, hài hòa, dễ dùng" | **Vi phạm Unambiguous** | Các tính từ cảm tính, mỗi người đọc hiểu một kiểu; dev, QA và khách hàng không thể cùng đưa ra một cách hiểu duy nhất. | Thay bằng tiêu chí định lượng: thời gian hoàn thành, số trường, tỷ lệ thành công, độ tương phản (WCAG). |
| 3 | REQ-02 | "phản hồi siêu nhanh, làm khách hài lòng" | **Vi phạm Verifiable** | Không có ngưỡng thời gian, tải, dữ liệu nền; "hài lòng" là cảm xúc. Không thể viết test case có kết quả Đạt/Không đạt. | Gắn chỉ số: p95 (giây), số request đồng thời, kích thước dữ liệu, môi trường đo. |
| 4 | REQ-03 và REQ-04 | REQ-03: chỉ nêu luồng đúng, "chuyển ngay đến kho đích". REQ-04: "nếu thấy cần thiết" | **Vi phạm Complete** | REQ-03 thiếu: kiểm tra dữ liệu đầu vào (cân nặng/kích thước = 0, âm, vượt tải trọng 50kg xe máy / 500kg xe tải), thông báo lỗi, xử lý đường truyền chập chờn (trùng mã vận đơn), và "ngay" là mơ hồ. REQ-04 thiếu: ghi những sự kiện nào, những trường nào, lưu bao lâu, điều kiện ghi là tùy ý. | Bổ sung điều kiện biên, mã lỗi, Idempotency Key; xác định rõ phạm vi, dữ liệu, thời gian lưu của nhật ký. |

**Nhận xét bổ sung:**
- REQ-03 còn gộp nhiều hành vi (tạo đơn, kiểm tra, điều phối) vào một câu, vi phạm tính nguyên tử, khó truy vết. Bản mới tách thành 3 yêu cầu.
- Từ "ngay", "nhanh nhất có thể", "nếu thấy cần thiết" cũng là từ mơ hồ cần loại bỏ.

---

## PHẦN 2. ĐẶC TẢ VIẾT LẠI

### 1.3 Definitions, acronyms, abbreviations (chuyển từ mục 2.4)

| Thuật ngữ | Định nghĩa |
|---|---|
| **COD** | Cash On Delivery – dịch vụ thu hộ tiền mặt từ người nhận khi giao hàng, số tiền thu hộ được nhập khi tạo đơn. |
| **Bưu kiện hỏa tốc** | Bưu kiện đăng ký dịch vụ hỏa tốc, có thời gian giao cam kết **không quá 4 giờ** kể từ lúc trạng thái đơn là "Đã tiếp nhận", trong cùng tỉnh/thành *(ngưỡng 4 giờ là giả định, cần Product Owner xác nhận)*. |
| **Idempotency Key** | Chuỗi định danh duy nhất (UUID v4) do client sinh cho mỗi lần thao tác "Tạo đơn"; cùng key gửi nhiều lần chỉ tạo tối đa một đơn. |
| **Mã vận đơn** | Mã duy nhất do hệ thống sinh để định danh một bưu kiện. |
| **p95** | Ngưỡng thời gian mà 95% số yêu cầu đáp ứng nhanh hơn hoặc bằng. |

### 2.4 Constraints (ví dụ nội dung đúng chương mục)
- Hệ thống giao tiếp với các dịch vụ khác qua REST/HTTPS (TLS 1.2 trở lên).
- Tải trọng phương tiện: xe máy ≤ 50 kg, xe tải ≤ 500 kg.

### 3.2 Yêu cầu chức năng / phi chức năng (bản mới)

#### USA-01 (thay REQ-01) – Khả năng sử dụng của giao diện tiếp nhận đơn
> Màn hình tiếp nhận đơn phải: (a) hiển thị toàn bộ trường bắt buộc trên **một màn hình**, tối đa **10 trường bắt buộc**, đánh dấu bằng ký hiệu "*"; (b) hiển thị thông báo lỗi ngay cạnh trường sai trong **≤ 200 ms** sau khi rời trường; (c) đạt độ tương phản chữ/nền **≥ 4.5:1** (WCAG 2.1 mức AA).

**Tiêu chí kiểm chứng:** Kiểm thử người dùng với 10 nhân viên tiếp nhận mới được hướng dẫn 30 phút: **≥ 9/10** người tạo đơn hợp lệ trong **≤ 90 giây**; công cụ kiểm tra độ tương phản báo 0 lỗi mức AA trên màn hình tiếp nhận.

#### PER-01 (thay REQ-02) – Hiệu năng tra cứu trạng thái bưu kiện
> Với cơ sở dữ liệu **5 triệu** bưu kiện, hệ thống phải trả kết quả tra cứu theo mã vận đơn trong **≤ 2 giây ở p95** và **≤ 3 giây ở p99** khi có **500 request đồng thời**, tỷ lệ lỗi **< 0,1%**.

**Tiêu chí kiểm chứng:** Chạy test tải (JMeter/k6) 10 phút trên môi trường staging có cấu hình tương đương production, với 500 luồng đồng thời; báo cáo p95 ≤ 2 s, p99 ≤ 3 s, lỗi < 0,1%.

#### REQ-INTAKE-01 (thay REQ-03, phần luồng chính) – Tạo đơn và chuyển yêu cầu về kho đích
> Khi nhân viên nhập đủ thông tin hợp lệ và nhấn "Tạo đơn", hệ thống phải, trong **≤ 3 giây**: (a) sinh mã vận đơn duy nhất; (b) lưu đơn với trạng thái "Đã tiếp nhận"; (c) gửi yêu cầu điều phối đến kho đích để kho đích thấy đơn trong danh sách "Chờ nhận" (không bao gồm việc vận chuyển vật lý).

**Tiêu chí kiểm chứng:** Tạo 100 đơn hợp lệ: 100% có mã vận đơn không trùng, trạng thái "Đã tiếp nhận", xuất hiện ở danh sách "Chờ nhận" của kho đích; thời gian xử lý ≤ 3 s ở ≥ 95 đơn.

#### REQ-INTAKE-02 (thay REQ-03, phần kiểm tra dữ liệu) – Kiểm tra cân nặng/kích thước và tải trọng
> Hệ thống phải từ chối tạo đơn và hiển thị thông báo lỗi riêng cho từng trường hợp, không dùng thông báo chung chung:

| Mã lỗi | Điều kiện | Thông báo hiển thị |
|---|---|---|
| ERR-W01 | Cân nặng = 0 hoặc để trống | "Cân nặng phải lớn hơn 0 kg." |
| ERR-W02 | Cân nặng < 0 | "Cân nặng không được là số âm." |
| ERR-W03 | Cân nặng > 50 kg với phương tiện xe máy | "Cân nặng vượt tải trọng xe máy (tối đa 50 kg). Hãy chọn xe tải." |
| ERR-W04 | Cân nặng > 500 kg với phương tiện xe tải | "Cân nặng vượt tải trọng xe tải (tối đa 500 kg)." |
| ERR-D01 | Bất kỳ kích thước (dài/rộng/cao) = 0 hoặc để trống | "Kích thước <tên trường> phải lớn hơn 0 cm." |
| ERR-D02 | Bất kỳ kích thước < 0 | "Kích thước <tên trường> không được là số âm." |

> Khi có lỗi, đơn không được lưu, dữ liệu đã nhập được giữ nguyên trên form.

**Tiêu chí kiểm chứng:** Bộ test giá trị biên: cân nặng {0, -1, 50, 50.01, 500, 500.01} và mỗi kích thước {0, -1, 1}; mỗi ca báo đúng mã lỗi/thông báo ở bảng trên; giá trị 50 kg (xe máy) và 500 kg (xe tải) được chấp nhận.

#### REQ-INTAKE-03 (thay REQ-03, phần chống trùng) – Chống trùng mã vận đơn khi mạng chập chờn (Idempotency Key)
> Mỗi lần nhấn "Tạo đơn", client phải gửi kèm một Idempotency Key (UUID v4), giữ nguyên key khi tự động gửi lại. Server phải lưu key trong **24 giờ**; nếu nhận lại cùng key, server **không tạo đơn mới** mà trả về kết quả (mã vận đơn) của lần đầu với cùng mã phản hồi. Nếu cùng key nhưng nội dung khác, trả lỗi HTTP 409.

**Tiêu chí kiểm chứng:** Gửi cùng một yêu cầu 5 lần (kể cả 5 luồng song song, hoặc ngắt mạng giữa chừng rồi gửi lại): CSDL chỉ có **đúng 1 đơn**, cả 5 phản hồi có cùng mã vận đơn; gửi cùng key khác nội dung → HTTP 409.

#### AUD-01 (thay REQ-04) – Nhật ký kiểm toán
> Hệ thống **luôn** ghi nhật ký kiểm toán cho mọi thao tác tạo, sửa, hủy đơn và tra cứu đơn có COD. Mỗi bản ghi gồm: mã người dùng, thời điểm (UTC, chính xác đến mili-giây), loại thao tác, mã vận đơn, giá trị trước/sau (với sửa), địa chỉ IP. Bản ghi chỉ được thêm mới, không được sửa/xóa, được lưu **≥ 12 tháng** *(giả định, cần xác nhận với bộ phận kiểm toán)* và ghi trong **≤ 1 giây** sau thao tác.

**Tiêu chí kiểm chứng:** Thực hiện 20 thao tác thuộc từng loại: có đủ 20 bản ghi với đầy đủ trường; thử sửa/xóa bản ghi bằng tài khoản quản trị ứng dụng bị từ chối; kiểm tra chính sách lưu trữ ≥ 12 tháng.

---

## BẢNG TRUY VẾT: BẢN CŨ → BẢN MỚI

| Bản cũ | Lỗi | Bản mới |
|---|---|---|
| Mục 2.4 (định nghĩa) | Sai chương mục | Mục 1.3 |
| REQ-01 | Unambiguous | USA-01 |
| REQ-02 | Verifiable | PER-01 |
| REQ-03 | Complete | REQ-INTAKE-01, -02, -03 |
| REQ-04 | Complete | AUD-01 |

*Các con số (ngưỡng 4 giờ, 24 giờ, 12 tháng, 500 request…) là giả định hợp lý cho bài tập, cần thống nhất với các bên liên quan trước khi chốt SRS.*
