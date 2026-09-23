# BT1 – Thiết kế ERD quản lý khóa học EduSmart

## Bước 1 – Entity, Attribute, Khóa chính

| Entity | Attribute cần lưu | Khóa chính (PK) đề xuất |
|---|---|---|
| HOC_VIEN | MaHV, HoTen, NgaySinh | MaHV |
| KHOA_HOC | MaKH, TenKH, SoTietHoc | MaKH |
| **DANG_KY** (thực thể trung gian) | **MaHV (FK), MaKH (FK), DiemSo** | **(MaHV, MaKH)** – khóa ghép |

**Giải thích:** HOC_VIEN – KHOA_HOC là quan hệ N-N nên phải tách qua thực thể trung gian DANG_KY. `DiemSo` là thuộc tính của *lượt đăng ký* (phải biết học viên nào, khóa nào mới có điểm) nên đặt ở DANG_KY, không thuộc HOC_VIEN hay KHOA_HOC. Khóa ghép (MaHV, MaKH) đảm bảo một học viên không đăng ký trùng một khóa.

## Bước 2 – Quan hệ qua thực thể trung gian

File sơ đồ: `ERD_EduSmart.drawio` (mở bằng draw.io / app.diagrams.net).

```
HOC_VIEN ||──────o{ DANG_KY }o──────|| KHOA_HOC
 (MaHV PK)          (MaHV FK, MaKH FK, DiemSo)     (MaKH PK)
```

- (a) HOC_VIEN **1 – N** DANG_KY: một học viên có nhiều lượt đăng ký; phía "Nhiều" ở DANG_KY. Khóa ngoại: `DANG_KY.MaHV → HOC_VIEN.MaHV`.
- (b) KHOA_HOC **1 – N** DANG_KY: một khóa học có nhiều lượt đăng ký; phía "Nhiều" ở DANG_KY. Khóa ngoại: `DANG_KY.MaKH → KHOA_HOC.MaKH`.

Hai khóa ngoại này đồng thời là hai thành phần của khóa chính ghép (MaHV, MaKH).

## Bước 3 – Chuẩn hóa dữ liệu (2NF)

Bảng nháp, khóa chính ghép (MaHV, MaKH):

| MaHV | MaKH | TenKH | DiemSo |
|---|---|---|---|
| HV01 | KH01 | Nhập môn Lập trình | 8.5 |
| HV01 | KH02 | Thiết kế CSDL | 9.0 |
| HV02 | KH01 | Nhập môn Lập trình | 7.0 |

### Cột vi phạm 2NF: `TenKH`

Xét phụ thuộc hàm:
- `(MaHV, MaKH) → DiemSo`: **phụ thuộc đầy đủ** vào khóa ghép (cần cả học viên lẫn khóa học mới xác định được điểm) → đạt.
- `MaKH → TenKH`: chỉ cần **một phần** khóa (MaKH) là xác định được TenKH, không liên quan MaHV → **phụ thuộc không đầy đủ (phụ thuộc bộ phận)** → vi phạm 2NF.

**Hậu quả (dị thường dữ liệu):**
- *Dư thừa:* "Nhập môn Lập trình" bị lặp ở HV01 và HV02.
- *Dị thường cập nhật:* đổi tên KH01 phải sửa nhiều dòng, sót một dòng là dữ liệu mâu thuẫn.
- *Dị thường thêm:* chưa có học viên đăng ký thì không thể lưu khóa học mới.
- *Dị thường xóa:* xóa hết lượt đăng ký KH02 thì mất luôn thông tin khóa "Thiết kế CSDL".

### Cách tách bảng đạt 2NF

Chuyển thuộc tính phụ thuộc bộ phận (TenKH) sang bảng riêng với khóa MaKH:

**KHOA_HOC** (PK: MaKH)

| MaKH | TenKH | SoTietHoc |
|---|---|---|
| KH01 | Nhập môn Lập trình | (theo dữ liệu thực tế) |
| KH02 | Thiết kế CSDL | (theo dữ liệu thực tế) |

**DANG_KY** (PK: MaHV, MaKH; FK: MaHV, MaKH)

| MaHV | MaKH | DiemSo |
|---|---|---|
| HV01 | KH01 | 8.5 |
| HV01 | KH02 | 9.0 |
| HV02 | KH01 | 7.0 |

**HOC_VIEN** (PK: MaHV): MaHV, HoTen, NgaySinh (giữ nguyên).

**Kiểm tra lại:** KHOA_HOC có khóa đơn nên tự động đạt 2NF; DANG_KY chỉ còn `DiemSo` phụ thuộc đầy đủ vào (MaHV, MaKH) → đạt 2NF. Tên khóa học giờ chỉ lưu một lần, đúng với ERD ở Bước 2.

## Phụ lục – SQL tham khảo

```sql
CREATE TABLE HOC_VIEN (
  MaHV     VARCHAR(10) PRIMARY KEY,
  HoTen    VARCHAR(100) NOT NULL,
  NgaySinh DATE
);
CREATE TABLE KHOA_HOC (
  MaKH       VARCHAR(10) PRIMARY KEY,
  TenKH      VARCHAR(150) NOT NULL,
  SoTietHoc  INT
);
CREATE TABLE DANG_KY (
  MaHV    VARCHAR(10),
  MaKH    VARCHAR(10),
  DiemSo  DECIMAL(4,2),
  PRIMARY KEY (MaHV, MaKH),
  FOREIGN KEY (MaHV) REFERENCES HOC_VIEN(MaHV),
  FOREIGN KEY (MaKH) REFERENCES KHOA_HOC(MaKH)
);
```
