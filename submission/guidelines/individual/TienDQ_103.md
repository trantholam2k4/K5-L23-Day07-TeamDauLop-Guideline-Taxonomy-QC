# Guideline gán nhãn — v-cá nhân — 2A202603028 - Đinh Quốc Tiến

- Phiên bản: cá nhân (Pha 1)
- Ngày: &lt;điền ngày&gt;
- Người viết / duyệt:

## 1. Mục tiêu

Tạo bounding box cho người và phương tiện nhìn thấy trong ảnh đường phố để huấn luyện mô hình phát hiện đối tượng. Bỏ sót vật nhìn rõ, gán nhầm lớp hoặc box không nhất quán sẽ khiến mô hình học sai. Chỉ gán vật thật thuộc taxonomy; vật không chắc chắn cần đánh dấu `can_review` và ghi chú để xem lại.

## 2. Lớp và thuộc tính

### Lớp


| Lớp             | Định nghĩa                                                                                          | Ví dụ                                                       | Phản ví dụ / dễ nhầm                                    |
| --------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------- |
| `car`           | Ô tô chở người, không thuộc nhóm van, truck hoặc pickup truck.                                      | Ô tô con, SUV.                                              | Xe tải, bán tải, van, xe ba bánh.                       |
| `truck`         | Xe tải chở hàng có khoang hàng chuyên dụng, thường tách biệt hoặc phân biệt rõ với cabin.           | Xe tải thùng, xe tải ben.                                   | Pickup có thùng hở liền thân; van thân kín.             |
| `pickup_truck`  | Xe có cabin và thùng hàng hở phía sau liền cùng thân xe.                                            | Xe bán tải.                                                 | Truck thùng; ô tô/SUV không có thùng hở.                |
| `van`           | Xe có khoang thân kín dùng chở người hoặc hàng, không có thùng hàng hở riêng.                       | Van chở người/giao hàng.                                    | Pickup truck; xe tải có thùng chuyên dụng.              |
| `motorcycle`    | Xe máy/mô tô hai bánh không gắn khoang sidecar.                                                     | Xe máy hai bánh có/không có người lái.                      | Xe ba bánh; tổ hợp có sidecar.                          |
| `person`        | Người thật hiện diện trong cảnh, dù đứng, đi bộ hoặc ngồi trên phương tiện.                         | Người đi bộ, người lái xe nhìn thấy được.                   | Tượng, ma-nơ-canh, hình người in.                       |
| `fake_person`   | Hình thể mô phỏng người nhưng không phải người sống trong cảnh.                                     | Tượng người, ma-nơ-canh, hình người in trên biển/quảng cáo. | Người thật; phản chiếu người (không gán riêng).         |
| `ignore_region` | Vùng chứa các đối tượng mục tiêu không thể gán riêng vì quá nhỏ, dày đặc hoặc không phân biệt được. | Đám đông không thể tách từng người.                         | Đối tượng đơn lẻ nhìn rõ; không dùng để che lỗi bỏ sót. |
| `three_wheels`  | Phương tiện có ba bánh, thiết kế chở người/hàng, có động cơ hoặc dùng sức người.                    | Tuk-tuk, xe lam, xích lô.                                   | Xe bò/ngựa kéo; xe hai bánh.                            |
| `other_vehicle` | Phương tiện thật dùng di chuyển/chở người/hàng nhưng không thuộc lớp phương tiện nào khác.          | Xe bò/ngựa kéo; phương tiện chuyên dụng tự chế.             | Tượng xe, đồ chơi, xe trưng bày không hoạt động.        |
| `sidecar`       | Tổ hợp xe máy gắn khoang phụ bên cạnh; gán toàn bộ tổ hợp như một đối tượng.                        | Xe máy có khoang phụ chở người/hàng bên cạnh.               | Xe máy hai bánh thường; tuk-tuk ba bánh.                |


Các lớp mới là đề xuất cá nhân, chưa phải taxonomy đã chốt. Vật thật không thuộc lớp nào: không tự tạo lớp mới hoặc ép vào lớp gần giống; ghi chú và bật `can_review`. Chỉ dùng `ignore_region` cho vùng không thể gán từng vật theo quy tắc nhóm.

### Thuộc tính


| Thuộc tính       | Mức   | Nghĩa / tiêu chí đề xuất                                                                                       |
| ---------------- | ----- | -------------------------------------------------------------------------------------------------------------- |
| `occluded_level` | `0`   | Không bị vật khác che đáng kể.                                                                                 |
|                  | `1`   | Ước tính bị che không quá 25% diện tích toàn vật.                                                              |
|                  | `2`   | Bị che trên 25% đến 50%.                                                                                       |
|                  | `3`   | Bị che trên 50%, hoặc chỉ còn phần nhỏ khiến nhận dạng khó.                                                    |
| `truncated`      | `no`  | Vật nằm trọn trong ảnh, không bị khung ảnh cắt.                                                                |
|                  | `yes` | Vật chạm/cắt mép ảnh và có phần vật thể tiếp tục ra ngoài khung.                                               |
| `can_review`     | `yes` | Chưa chắc lớp, box, mức che khuất hoặc cách áp dụng quy tắc; ghi lý do để nhóm xem lại. Chọn `yes` trong CVAT. |
|                  | `no`  | Đã áp dụng quy tắc và không cần xem lại. Đây là giá trị mặc định trong CVAT.                                   |


Ngưỡng phần trăm che khuất là ước lượng theo diện tích toàn vật và chỉ là đề xuất cho bản cá nhân; cần thống nhất cách ước lượng ở Pha 2. `truncated` chỉ mô tả vật bị mép ảnh cắt, không phải vật bị đối tượng khác che.

## 3. Quy tắc vẽ box

1. Vẽ hình chữ nhật sát đường bao phần vật **nhìn thấy**; không đoán phần bị che khuất.
2. Vật bị che vẫn được gán nếu phần nhìn thấy đủ để nhận diện. Chọn `occluded_level` theo bảng thuộc tính.
3. Vật bị cắt bởi cạnh ảnh: box dừng tại cạnh ảnh và đặt `truncated = yes`.
4. Người thật gán `person`; tượng, ma-nơ-canh và hình người in gán `fake_person`. Không gán phản chiếu thành đối tượng riêng. Đồ chơi và hình phương tiện in không gán như phương tiện thật.
5. Nếu người và xe đều nhìn thấy, vẽ box riêng cho người và xe; không gộp hai vật vào một box.
6. Chỉ vẽ box riêng khi chiều cao vật trong ảnh từ 20 px trở lên. Vật mục tiêu thấp hơn 20 px không vẽ box riêng; cách dùng `ignore_region` cho các vật nhỏ/dày đặc cần được nhóm chốt.
7. Tuk-tuk/xe lam/xích lô gán `three_wheels`; xe máy gắn khoang phụ bên cạnh gán một box `sidecar` cho toàn tổ hợp, không gán thêm `motorcycle`; xe bán tải gán `pickup_truck`. Xe chở phương tiện khác: nếu vật được chở nhìn thấy và tách biệt, gán box riêng. Ca chưa rõ thì bật `can_review` và ghi chú.
8. Ảnh không có vật nào thuộc taxonomy thì để ảnh không có annotation; không tạo box cho nền hoặc vùng đường phố thông thường.

## 4. Ca khó

Điền các hàng sau khi tự xem ảnh. Không ghi ví dụ suy đoán; ghi mã ảnh, điều bạn thực sự quan sát, quyết định ban đầu và lý do.


| Ảnh         | Tình huống quan sát được | Quyết định ban đầu | Lý do / điểm chưa chắc                          |
| ----------- | ------------------------ | ------------------ | ----------------------------------------------- |
| `<img_011>` | `Xe mô hình`             | `Can_review`       | `<quy tắc đã áp dụng và câu hỏi cần nhóm chốt>` |


Câu hỏi chưa chốt được ghi rõ trong bảng và đánh dấu `can_review` khi gán. Không tự đoán để làm cho ca đó có vẻ đã thống nhất.

## 5. QC và phiên bản

- Box phải nằm trong khung ảnh; chiều cao box tối thiểu là 20 px.
- Khi so sánh box, ngưỡng IoU là 0.85 với vật cao từ 50 px; 0.7 với vật thấp hơn 50 px.
- Ngưỡng IAA của lô: IoU trung bình ít nhất 0.85 và đồng thuận lớp ít nhất 95%.
- Tỉ lệ ảnh có lỗi để cân nhắc Release không vượt 2%; luôn ghi tỉ lệ kèm số ảnh lỗi và cỡ mẫu.
- Script kiểm được định dạng, lớp/thuộc tính hợp lệ, box trong ảnh và độ lệch giữa người gán. Script không xác nhận được box có ôm sát vật thật, lớp có đúng thực tế, hay cả hai người cùng bỏ sót một vật.
- Ca chưa rõ được nhóm thảo luận ở Pha 2 hoặc calibration; quyết định và người chốt phải được ghi lại.


| Phiên bản       | Thay đổi                                                         | Lý do / ca nào                                                                                  |
| --------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Cá nhân (Pha 1) | Bản nháp đầu; thêm tiêu chí phân loại, box, che khuất và vật nhỏ | Các điểm v0 chưa quy định rõ; sẽ cập nhật nội dung cá nhân sau khi tự xem ảnh và ghi ca thực tế |




&nbsp;