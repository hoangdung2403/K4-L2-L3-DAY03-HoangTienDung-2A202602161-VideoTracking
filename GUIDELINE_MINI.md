# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: Hoàng Tiến Dũng — MSSV 2A202602161
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: không có.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu occlusion dưới **25 frame** (mặc định lab, tương đương 2 giây @ 12.5 fps). | Đây vẫn là cùng một vật thể nếu còn đủ continuity theo chuyển động/hình dáng. |
| Xe bị che lâu hơn 25 frame | Chỉ tạo ID mới khi không còn đủ bằng chứng để nối chắc chắn với track cũ. | Tránh gán nhầm identity sau một khoảng mất dấu dài. |
| Xe rời khung hình rồi quay lại | **Track mới**. | Tuân thủ quy tắc lab và tránh nối nhầm một xe khác đi vào cùng vùng. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo trajectory trước/sau điểm giao; không đổi ID chỉ vì bbox overlap. | Identity phải liên tục qua crossing, không phụ thuộc riêng vào vị trí tức thời. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox ôm phần **nhìn thấy được**. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu từ frame đầu tiên có đủ hình dạng để xác định chắc chắn là xe bốn bánh. |
| Xe đang đỗ, không di chuyển | Vẫn giữ track nếu xe thuộc phạm vi gán nhãn; không dùng chuyển động làm điều kiện bắt buộc. |
| Keyframe đặt dày ở đâu | Đặt dày hơn tại lúc xe đổi hướng, tăng/giảm kích thước nhanh, crossing, occlusion và gần frame vào/ra khung. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01 / frame 90–95 / ID 5`
- Tình huống: bbox của xe ID 5 có khác biệt hình học rõ với teaching reference quanh đoạn này; evaluation đánh dấu các frame 91–95 là các điểm IoU thấp.
- Quyết định: giữ cùng identity, nhưng cần kiểm tra lại bbox/keyframe ở đoạn xe thay đổi vị trí.
- Lý do: lỗi chính là hình học bbox, không phải ID switch; identity vẫn nhất quán.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 113–115 / ID 6 và 7`
- Tình huống: hai xe ở các vùng gần nhau; bản annotation hiện tại dùng ID 6/7 trong khi reference ánh xạ hai trajectory theo thứ tự ngược ở đoạn này.
- Quyết định: xác định identity bằng trajectory trước/sau crossing, không đổi ID chỉ vì hai bbox gần/chồng nhau.
- Lý do: đây là tình huống association/crossing dễ gây đổi ID nếu chỉ nhìn một frame.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 149–151 / ID 4`
- Tình huống: ID 4 của annotation còn bbox ở 3 frame sau khi trajectory tương ứng đã rời khung.
- Quyết định: khi rework phải kiểm tra frame outside chính xác và kết thúc track tại frame xe thực sự rời ảnh.
- Lý do: evaluation xác định đây là bbox treo sau khi xe rời khung.

### Ca 4 (bổ sung)
- Clip / frame / ID: `clip_01 / frame 169–171 / ID 8`
- Tình huống: ID 8 còn một bbox hẹp ở mép phải trong 3 frame sau khi xe tương ứng đã rời khung.
- Quyết định: kiểm tra lại outside ở mép phải và không để bbox treo.
- Lý do: evaluation xác định đây là lỗi boundary/outside, không phải identity switch.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Các điểm cần rework được xác định bằng evaluation:
- kiểm tra bbox/khung thời gian của ID 5 quanh frame 91–95;
- kiểm tra bbox của ID 6/7 quanh frame 113–115;
- kiểm tra `outside` của ID 4 ở frame 149–151;
- kiểm tra `outside` của ID 8 ở frame 169–171;
- kiểm tra các đoạn phủ thiếu của trajectory tương ứng với gold track 5 và 6.

**Lưu ý:** các thay đổi annotation phải được thực hiện trong CVAT rồi export lại MOT; không sửa trực tiếp `gt.txt` để vượt validator.
