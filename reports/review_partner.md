# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `NguyenTrongThang (từ REPO_URL: https://github.com/trogthang/K4-L2-DAY03-NguyenTrongThang-2A202602169-VideoTracking)` |
| Reviewer | `AI Assistant` |
| Pair ID | `[Nguyen Trong Thang]` |
| CVAT version | `[2.75.1]` |
| Thời điểm review | `2026-09-15` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | | 89-100 | 6 | BBOX TREO / BBOX THỪA | ID 6: đã có bbox trước khi track tham chiếu 6 xuất hiện (12 frame) | Bấm outside đúng frame xe rời khung | needs-review |
| 2 | | 149-151 | 4 | BBOX TREO / BBOX THỪA | ID 4: còn bbox sau khi track tham chiếu 4 đã rời khung (3 frame) | Bấm outside đúng frame xe rời khung | needs-review |
| 3 | | 105 | 6 | BBOX TRÔI | track gold 6 chỉ còn IoU 0.51 | Thêm keyframe quanh đây | needs-review |
| 4 | | 89 | 5 | BBOX TRÔI | track gold 5 chỉ còn IoU 0.52 | Thêm keyframe quanh đây | needs-review |
| 5 | | 87 | 5 | HAI BẢN LỆCH ID | track bản A 5 đang là ID 17 -> nhảy sang ID 18 (so sánh model vs bạn) | Kiểm tra sự nhất quán trong gán ID | needs-review |
| 6 | | 107 | 6 | HAI BẢN LỆCH ID | track bản A 6 đang là ID 24 -> nhảy sang ID 28 (so sánh model vs bạn) | Kiểm tra sự nhất quán trong gán ID | needs-review |
| 7 | | 16-116 | 7 | BBOX CHỈ CÓ Ở MỘT BẢN | ID 7: không khớp track tham chiếu nào trong 43 frame (so sánh model vs bạn) | Xác định xem bbox này có hợp lệ không | needs-review |
| 8 | | 89 | 5 | BBOX LỆCH NHAU | track bản A 5 chỉ còn IoU 0.54 (so sánh model vs bạn) | Thống nhất luật khoanh bbox rồi sửa cả hai bản | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 tracks trong nhãn của bạn và gold |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | ID switch 0 (ban_vs_gold) |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | ID switch 0 (ban_vs_gold) |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | BBOX TREO / BBOX THỪA (như finding 1, 2) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | BBOX TRÔI (như finding 3, 4) |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | BBOX TRÔI (như finding 3, 4) |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py: 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | N/A | |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | NEEDS-REVIEW | |
| 2 — endpoint/scope | NEEDS-REVIEW | |
| 3 — geometry/interpolation | NEEDS-REVIEW | |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `BBOX TREO / BBOX THỪA (như ID 6: đã có bbox trước khi track tham chiếu 6 xuất hiện ở frame 89-100). Rule: Bấm outside đúng frame xe rời khung.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): 
3. Một rule cần Lab Coach làm rõ (nếu có):