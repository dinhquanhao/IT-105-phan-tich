# BT2 – Thiết kế ERD quản lý căn hộ cho thuê RikkeiRealty

## Bước 1 – Entity, Attribute, Khóa chính

| Entity | Attribute cần lưu | Khóa chính (PK) đề xuất |
|---|---|---|
| CAN_HO | MaCH, DienTich, SoTang | MaCH |
| HOP_DONG_THUE | MaHD, NgayBatDau, NgayKetThuc | MaHD |
| **KHU_VUC** | **MaKhuVuc, TenKhuVuc** | **MaKhuVuc** |

**Giải thích:** KHU_VUC là thực thể riêng vì tên khu vực là thông tin dùng chung cho nhiều căn hộ, lưu ở một nơi để không lặp dữ liệu. MaKhuVuc làm khóa chính vì mỗi khu vực cần một định danh duy nhất.

## Bước 2 – Quan hệ và khóa ngoại

File sơ đồ: `ERD_RikkeiRealty.drawio` (mở bằng draw.io / app.diagrams.net).

```
KHU_VUC ||──────o{ CAN_HO ||──────o| HOP_DONG_THUE
(MaKhuVuc PK)     (MaCH PK,          (MaHD PK,
                   MaKhuVuc FK)       MaCH FK UNIQUE)
```

**(a) KHU_VUC 1 – N CAN_HO**
- Một khu vực có 0 hoặc nhiều căn hộ; mỗi căn hộ thuộc đúng 1 khu vực.
- Khóa ngoại đặt ở phía "Nhiều": `CAN_HO.MaKhuVuc → KHU_VUC.MaKhuVuc`, **NOT NULL** (căn hộ luôn thuộc đúng 1 khu vực).

**(b) CAN_HO 1 – 1 (tùy chọn) HOP_DONG_THUE**
- Một căn hộ có 0 hoặc 1 hợp đồng hiệu lực; mỗi hợp đồng thuộc đúng 1 căn hộ.
- Khóa ngoại đặt ở phía **bắt buộc** là HOP_DONG_THUE: `HOP_DONG_THUE.MaCH → CAN_HO.MaCH`, **NOT NULL + UNIQUE**.
  - NOT NULL: hợp đồng phải thuộc một căn hộ.
  - UNIQUE: mỗi căn hộ chỉ có tối đa 1 hợp đồng, tạo nên quan hệ 1-1.
  - Căn hộ chưa cho thuê đơn giản là không có dòng nào trong HOP_DONG_THUE trỏ tới nó (phía tùy chọn, ký hiệu "0..1"). Nếu đặt FK ở CAN_HO thì cột đó phải cho NULL, kém tự nhiên hơn.

## Bước 3 – Chuẩn hóa dữ liệu (3NF)

| MaCH | DienTich | MaKhuVuc | TenKhuVuc |
|---|---|---|---|
| CH01 | 45 | KV01 | Quận 1 |
| CH02 | 60 | KV01 | Quận 1 |
| CH03 | 50 | KV02 | Quận 7 |

Khóa chính là `MaCH` (khóa đơn nên không thể vi phạm 2NF, các ô đều là giá trị đơn nên đạt 1NF).

### Cột vi phạm 3NF: `TenKhuVuc`

Phụ thuộc hàm:
- `MaCH → MaKhuVuc` và `MaCH → DienTich`: thuộc tính không khóa phụ thuộc trực tiếp vào khóa → đạt.
- `MaKhuVuc → TenKhuVuc`: `TenKhuVuc` do `MaKhuVuc` (một thuộc tính **không khóa**) quyết định.
- Suy ra `MaCH → MaKhuVuc → TenKhuVuc`: `TenKhuVuc` phụ thuộc vào khóa chính **một cách gián tiếp (bắc cầu)** → vi phạm 3NF.

**Hậu quả (dị thường):**
- *Dư thừa:* "Quận 1" lặp ở CH01 và CH02.
- *Dị thường cập nhật:* đổi tên KV01 phải sửa nhiều dòng, sót là mâu thuẫn.
- *Dị thường thêm:* khu vực mới chưa có căn hộ thì không lưu được.
- *Dị thường xóa:* xóa căn hộ CH03 (duy nhất của KV02) thì mất luôn khu vực "Quận 7".

### Cách tách bảng đạt 3NF

Đưa thuộc tính bắc cầu cùng thuộc tính quyết định nó (`MaKhuVuc`) ra bảng riêng, giữ `MaKhuVuc` ở bảng gốc làm khóa ngoại:

**KHU_VUC** (PK: MaKhuVuc)

| MaKhuVuc | TenKhuVuc |
|---|---|
| KV01 | Quận 1 |
| KV02 | Quận 7 |

**CAN_HO** (PK: MaCH; FK: MaKhuVuc → KHU_VUC)

| MaCH | DienTich | MaKhuVuc |
|---|---|---|
| CH01 | 45 | KV01 |
| CH02 | 60 | KV01 |
| CH03 | 50 | KV02 |

(`SoTang` không có trong bảng nháp nên chỉ thêm vào CAN_HO khi có dữ liệu, không ảnh hưởng chuẩn hóa.)

**Kiểm tra lại:** ở mỗi bảng, mọi thuộc tính không khóa chỉ phụ thuộc trực tiếp vào khóa chính, không còn phụ thuộc bắc cầu → đạt 3NF, và khớp với ERD ở Bước 2.

## Phụ lục – SQL tham khảo

```sql
CREATE TABLE KHU_VUC (
  MaKhuVuc   VARCHAR(10) PRIMARY KEY,
  TenKhuVuc  VARCHAR(100) NOT NULL
);
CREATE TABLE CAN_HO (
  MaCH       VARCHAR(10) PRIMARY KEY,
  DienTich   DECIMAL(6,2),
  SoTang     INT,
  MaKhuVuc   VARCHAR(10) NOT NULL,
  FOREIGN KEY (MaKhuVuc) REFERENCES KHU_VUC(MaKhuVuc)
);
CREATE TABLE HOP_DONG_THUE (
  MaHD          VARCHAR(10) PRIMARY KEY,
  NgayBatDau    DATE NOT NULL,
  NgayKetThuc   DATE,
  MaCH          VARCHAR(10) NOT NULL UNIQUE,
  FOREIGN KEY (MaCH) REFERENCES CAN_HO(MaCH)
);
```
