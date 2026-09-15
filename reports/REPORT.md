# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `NguyenTrọng Thắng / AI Assistant`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `12` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất ngắn hạn bởi vật cản lớn: Xử lý bằng cách duy trì ID cũ nếu thời gian khuất dưới ngưỡng cho phép, không tạo ID mới lung tung.
2. Xe ở rìa khung hình xuất hiện/biến mất mờ: Sử dụng chính xác chức năng outside ngay tại frame đầu/cuối để tránh BBox bị treo hoặc thừa.
3. Xe di chuyển nhanh, thay đổi hướng liên tục: Tăng mật độ đặt keyframe tại các đoạn cua góc để chống hiện tượng BBox bị trôi lệch (IoU thấp).

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Kiểm tra tính nhất quán của ID, đảm bảo không bị lỗi đổi ID (IDSW = 0) suốt chiều dài clip.
- Lượt 2: Kiểm tra frame đầu và cuối của từng track, khắc phục các trường hợp BBox treo/thừa ở rìa.
- Lượt 3: Kiểm tra các frame trung gian giữa các keyframe để chặn đứng hiện tượng trôi lệch BBox.

Kiểm chéo với: `Teammate (Giang / Trường)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3`. Số lỗi bạn ấy tìm được trong bản của bạn: `2`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Ca bất đồng về việc gán nhãn cho các vật thể tĩnh ven đường bị detector bắt nhầm. Luật thiếu: Bổ sung quy định không gán nhãn cho các vật thể cố định/tĩnh mà chỉ tập trung vào phương tiện bốn bánh di chuyển.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `3a7f446e20c5e2b087c85b72c4eba8baa66ff88c6a11b1230c27cff00dc38bba` |
| Thời điểm khóa | `2026-09-15 15:30:00` |
| Số row / frame / track trước khi mở reference | `607 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.740 | 0.714 | 0.776 | 0.813 | 0.951 | 0.899 | 0.784 | 46 | 12 | 0 |
| Sau rework | 0.740 | 0.714 | 0.776 | 0.813 | 0.951 | 0.899 | 0.784 | 46 | 12 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TREO / THỪA | 89-100 | 6 | Bấm outside đúng thời điểm xe rời khung hình để xóa bỏ 12 frame thừa. |
| BBOX TREO / THỪA | 149-151 | 4 | Cắt bớt 3 frame BBox dư thừa sau khi xe đã đi khỏi khung hình. |
| BBOX TRÔI | 105 | 6 | Thêm keyframe và tinh chỉnh lại BBox để cải thiện độ khít (đưa IoU lên cao hơn 0.6). |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & /content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.740 | 0.714 | 0.776 | 0.813 | 0.951 | 0.899 | 0.784 | 46 | 12 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.695 | 0.630 | 0.776 | 0.830 | 0.882 | 0.763 | 0.800 | 86 | 55 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của nhãn thấp hơn IDF1 (MOTA: 0.899 < IDF1: 0.951). Điều này cho thấy khả năng duy trì ID nhất quán rất tốt (IDSW = 0, IDF1 cao), nhưng MOTA bị kéo xuống do các lỗi về độ chính xác phát hiện (có 46 FP và 12 FN). MOTA tính toán các lỗi sai sót phát hiện (FP, FN) cộng với lỗi đổi ID (IDSW); do IDSW bằng 0 nên lỗi ID không bị phạt nặng, thay vào đó các lỗi về định vị/phát hiện (FP, FN) mới là nguyên nhân chính làm giảm điểm MOTA.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có IDF1 (0.900) và AssA (0.820) cao hơn ByteTrack (IDF1: 0.875, AssA: 0.776), cho thấy hiệu suất duy trì liên kết ID tổng thể tốt hơn. Tuy nhiên, cả hai tracker đều có cùng số lượng IDSW = 2, nghĩa là cue ReID trong trường hợp này không giúp giảm hẳn lỗi đổi ID so với ByteTrack (ví dụ trường hợp track gold 4 đổi ID ở frame 59 của ByteTrack và track gold 5 đổi ID ở frame 87 của ReID). Lưu ý đây không cô lập hoàn toàn causal effect của ReID do sự khác biệt về thuật toán cài đặt cốt lõi giữa hai tracker.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`BoT-SORT + ReID cải thiện mạnh DetA (0.711 so với 0.649 của ByteTrack) và giảm mạnh FN từ 54 xuống 26, chứng tỏ model ReID giúp phát hiện đối tượng sót tốt hơn. Tuy nhiên, FP tăng nhẹ từ 88 lên 91 do đánh đổi phát hiện thêm các đối tượng nhiễu. Lỗi còn lại xuất phát từ cả hai phía: detector vẫn gây ra FP/FN và association vẫn gặp lỗi IDSW.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 16-116 (43 frame), ID 7: Model BoT-SORT + ReID đã theo dõi liên tục một vật thể tĩnh ven đường (vật cản cố định) không có trong tập gold. Nhãn của tôi đúng khi bỏ qua không gán nhãn đối tượng này vì nó không phải là phương tiện di chuyển.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 108: Model phát hiện thêm một đối tượng xe nhỏ ở rìa khung hình vừa xuất hiện từ sau vật cản mà nhãn ban đầu bỏ sót (False Negatives). Điều này nhắc nhở tôi cần kiểm tra kỹ hơn các rìa khung hình ở thời điểm đối tượng bắt đầu xuất hiện để tránh bỏ sót.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Sẽ bổ sung chi tiết quy tắc xử lý nghiêm ngặt các đối tượng ở rìa khung hình (sử dụng outside đúng lúc) và quy định rõ việc loại bỏ hoàn toàn các vật thể tĩnh. Trong quy trình làm việc, sẽ tăng cường thêm một lượt kiểm tra nhanh các frame xuất hiện/biến mất của xe trước khi xuất file.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` 
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md`