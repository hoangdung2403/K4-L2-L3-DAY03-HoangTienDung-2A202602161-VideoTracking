# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: **Hoàng Tiến Dũng / cá nhân**
MSSV: **2A202602161**
Ngày: **2026-09-15**

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | Không lưu trong artifact hiện có |
| Thời gian gán `clip_01` | Không lưu trong artifact hiện có |
| Số track đã vẽ trong `clip_01` | 8 track |
| Số keyframe trung bình mỗi track | Không thể truy hồi chính xác từ MOT export |

Ba tình huống khó nhất:

1. Giữ identity khi hai xe ở gần/cắt nhau; đặc biệt đoạn frame 113–115 của ID 6/7.
2. Điều chỉnh bbox ở đoạn xe thay đổi kích thước/vị trí nhanh; nổi bật frame 91–95 của ID 5.
3. Xác định frame kết thúc khi xe rời mép ảnh; nổi bật ID 4 ở frame 149–151 và ID 8 ở frame 169–171.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua:

- Lượt 1 — nhìn ID: kiểm tra continuity của các track, đặc biệt đoạn crossing/overlap.
- Lượt 2 — frame đầu/cuối: kiểm tra start/end và outside.
- Lượt 3 — frame giữa: kiểm tra bbox/keyframe ở các đoạn xe đổi hướng hoặc bị che.

Kiểm chéo với: **CHƯA THỰC HIỆN / cần partner điền `reports/review_partner.md`.**

Số lỗi tìm được trong bản của partner: **chưa có**.
Số lỗi partner tìm được trong bản của tôi: **chưa có**.

Ca hai người quyết khác nhau: **chưa có dữ liệu partner**.

## 3. Pre-gold lock và chấm trước/sau rework

Pre-gold snapshot đã được tạo từ `annotations/clip_01/gt.txt` trước khi dùng teaching reference.

| Evidence | Giá trị |
| --- | --- |
| SHA-256 | `f5dc38bd5dac6e1debd1d5e0a30334657a4b0d88a5e6391d3f738a58f2ea6fc3` |
| Thời điểm khóa | Theo git/repo workflow ngày 2026-09-15; timestamp UTC cụ thể nằm trong `manifest.json` |
| Số row / frame / track trước reference | 565 rows / 190 frames / 8 tracks |

### Kết quả pre-gold / hiện tại so với gold

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold / annotation hiện tại | 0.811 | 0.795 | 0.830 | 0.881 | 0.956 | 0.913 | 0.869 | 21 | 29 | 0 |

**Lưu ý:** chưa có một snapshot “sau rework” được tạo vì annotation phải được sửa trong CVAT và export lại. Không bịa số liệu sau rework.

**Qua cổng:** **CÓ** — IDF1 0.956 >= 0.80, MOTA 0.913 >= 0.75, MOTP 0.869 >= 0.70.

### Các điểm cần rework

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | ---: | ---: | --- |
| Bbox treo sau khi rời khung | 149–151 | 4 | Chưa sửa; cần chỉnh `outside` trong CVAT |
| Bbox treo sau khi rời khung | 169–171 | 8 | Chưa sửa; cần chỉnh `outside` trong CVAT |
| Bbox lệch | 91–95 | 5 | Chưa sửa; cần kiểm tra/thêm keyframe trong CVAT |
| Bbox lệch | 113–115 | 6/7 | Chưa sửa; cần kiểm tra crossing và bbox trong CVAT |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Hai lần chạy model đã được thực hiện với đúng cấu hình lab. Config ghi nhận Python 3.13.15, Ultralytics 8.4.145, PyTorch 2.11.0+cu128, LAP 0.5.13, `yolo26n.pt`, `conf=0.25`, `IoU=0.70`, `imgsz=960`, classes `[2,5,7]`, device `0`, `persist=true`.

| Phương pháp | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW | Gate |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| Annotation của tôi vs gold | 0.8114 | 0.7947 | 0.8298 | 0.8806 | 0.9561 | 0.9127 | 0.8685 | 21 | 29 | 0 | PASS |
| YOLO26n + ByteTrack | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 | FAIL |
| YOLO26n + BoT-SORT + ReID | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 | PASS |

ByteTrack đạt IDF1 0.8746 nhưng MOTA 0.7487, thấp hơn ngưỡng 0.75 khoảng 0.0013 nên không qua gate. ReID tăng HOTA/DetA/AssA/LocA/IDF1/MOTA/MOTP so với ByteTrack; đặc biệt FN giảm từ 54 xuống 26 và IDF1 tăng từ 0.8746 lên 0.9001. Tuy nhiên FP tăng nhẹ 88 → 91 và IDSW vẫn là 2.

### Chẩn đoán model

**ByteTrack:** có ID switch tại frame 59 của gold track 4 và frame 94 của gold track 5; đồng thời có các đoạn coverage thấp của gold tracks 5, 6 và 8.

**ReID:** có ID switch tại frame 87 của gold track 5 và frame 113 của gold track 6. Có ba gold track bị fragmentation nhẹ (track 5, 6, 7) và một track bị partial coverage (track 6, 79%).

ReID có `n_pred_tracks=16`, bằng ByteTrack, nhưng chất lượng tốt hơn theo IDF1/MOTA/HOTA. ReID vẫn sinh các ghost prediction dài ở các khoảng 16–43 frame, vì vậy appearance matching chưa loại bỏ hoàn toàn false positive/track không khớp reference.

## 5. Phân tích — năm câu hỏi

### 1. MOTA của bạn cao hơn hay thấp hơn IDF1?

MOTA = **0.9127**, thấp hơn IDF1 = **0.9561**. IDF1 cao và IDSW = 0 cho thấy identity consistency của annotation tốt; MOTA còn chịu ảnh hưởng của FP/FN nên thấp hơn là hợp lý.

### 2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào?

ReID cho kết quả tốt hơn ByteTrack trên clip này: HOTA **0.7635 vs 0.7085**, IDF1 **0.9001 vs 0.8746**, MOTA **0.7923 vs 0.7487**, MOTP **0.8595 vs 0.8226**. FN giảm mạnh **54 → 26**, nhưng FP tăng **88 → 91** và IDSW giữ nguyên ở **2**. Vì vậy treatment ReID cải thiện association/coverage tổng thể, nhưng không giải quyết toàn bộ ID switch và ghost tracks.

### 3. DetA, FP và FN đổi thế nào?

So với gold, DetA tăng từ **0.6487 (ByteTrack)** lên **0.7110 (ReID)**; FN giảm **54 → 26**, trong khi FP tăng **88 → 91**. Điều này cho thấy ReID treatment có lợi rõ rệt về coverage nhưng phải trả giá bằng một lượng false positive nhỏ.

### 4. Một chỗ bạn đúng và ReID sai

Một ví dụ rõ là **frame 113, gold track 6**. Evaluator ghi nhận ReID đổi identity từ prediction 24 sang 31 tại frame này, tức có ID switch. Trong annotation của tôi, đoạn crossing 113–115 được giữ liên tục theo cặp ID 6/7 và evaluation annotation-vs-gold ghi nhận **0 IDSW**. Vì vậy đây là trường hợp annotation của tôi giữ identity ổn định hơn ReID.

### 5. Một chỗ ReID làm bạn xem lại annotation

**Frame 87–95, gold track 5** là điểm cần xem lại. ReID có ID switch ở frame 87 và prediction track 18 xuất hiện trước khi track 5 trong annotation của tôi bắt đầu; khi so ReID trực tiếp với annotation của tôi, evaluator cũng đánh dấu prediction 18 ở frame 87–89 là ghost vì xuất hiện trước track 5. Điều này không chứng minh ReID đúng, nhưng là evidence để kiểm tra lại trong CVAT xem track 5 có thực sự xuất hiện sớm hơn frame 90 hay không.

Ngoài ra, các loose boxes ở frame 94–95 của ReID so với annotation của tôi có IoU chỉ khoảng 0.58, càng cho thấy vùng frame 91–95 cần được kiểm tra lại về thời điểm bắt đầu và geometry của track 5.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ bổ sung guideline/quy trình theo ba hướng:

1. Quy định rõ cách kiểm tra crossing bằng trajectory trước và sau điểm giao, thay vì nhìn một frame.
2. Bắt buộc tua chậm quanh frame vào/ra và đặt keyframe dày hơn khi xe đổi kích thước nhanh.
3. Có một lượt kiểm tra riêng cho `outside` ở hai mép ảnh, vì bbox treo chỉ vài frame vẫn làm giảm chất lượng.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`
- [x] `outputs/eval_reid_vs_gold.json`
- [x] `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` — cần partner thực tế điền
- [x] `reports/REPORT.md`
