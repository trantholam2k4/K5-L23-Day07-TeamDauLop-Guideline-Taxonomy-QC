# Guideline gán nhãn — v0 (bản khởi đầu)

- Phiên bản: v0
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con, xe bán tải |
| `truck` | Xe tải, xe van |
| `van` | Xe thùng kín cỡ nhỏ. |
| `bus` | Xe buýt, xe khách |
| `motorcycle` | Xe máy, mô tô, xe 3 bánh. |
| `bicycle` | Xe đạp, xe xích lô |
| `person` | Người. |
| `ignore_region` | Vùng không cần gán từng vật. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2` | Mức bị che: 0 là thấy rõ, 1 là bị che một phần, 2 là bị che hoàn toàn - không nhìn rõ |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện trong ảnh. Nếu vật bị che thì ước lượng kích thước vật thể, kết hợp với vẽ box đối với phần vật thể có thể nhìn thấy còn lại
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.
4. Không gán lớp nếu box có thể nằm hoàn toàn trong khu vực của box khác.
5. Không gán lớp cho box có kích cỡ thấp hơn 20 x 20 px.
6. Hạn chế gán lớp cho box chứa hình ảnh của vật quá nhoè.

---
5. Hạn chế cho box chứa chi tiết thừa gần vật thể cần gán lớp
6. Tránh gán cho vật thể giả: mô hình đồ chơi, tượng / ma-nơ-canh, ảnh quảng cáo / poster / ... có chi tiết vật liên quan đến lớp. 

## 4. Ca khó

(chưa có)

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
