# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Tên: `Nguyễn Trọng Thắng`
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

Bổ sung của nhóm (nếu có): `Không gán nhãn cho các vật thể tĩnh (biển báo, cây cối, vật cản cố định) ngay cả khi detector bắt được.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Đảm bảo tính nhất quán của ID, tránh tạo ID mới không cần thiết khi xe bị che khuất ngắn hạn.` |
| Xe bị che lâu hơn ngưỡng trên | `Cấp ID mới khi xe xuất hiện trở lại nếu vượt quá thời gian che khuất cho phép.` | `Tránh lỗi suy luận sai ID do khoảng cách thời gian biến mất quá lâu.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Đảm bảo phân biệt rõ các lượt di chuyển ra vào khung hình khác nhau của phương tiện.` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ nguyên ID của từng xe qua điểm giao cắt dựa trên quỹ đạo di chuyển trước đó.` | `Hạn chế tối đa lỗi đổi ID (IDSW) khi các xe đi sát nhau.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `Sử dụng chức năng outside chính xác ngay tại frame đầu tiên/cuối cùng vật thể nhìn thấy rõ để tránh BBox treo/thừa ở rìa.` |
| Xe đang đỗ, không di chuyển | `Chỉ gán nhãn cho các vật thể chuyển động hoặc có khả năng tham gia giao thông, bỏ qua các vật thể tĩnh bị nhầm lẫn.` |
| Keyframe đặt dày ở đâu | `Đặt dày tại các đoạn xe thay đổi tốc độ đột ngột, cua góc, hoặc bị che khuất một phần để tránh BBox bị trôi (IoU < 0.6).` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 59 / ID 14 nhảy sang ID 15`
- Tình huống: `Xe bị che khuất một phần ngắn hạn rồi chuyển ID do thuật toán tự động gán nhãn ban đầu.`
- Quyết định: `Gộp chung lại thành một ID duy nhất xuyên suốt.`
- Lý do: `Duy trì IDF1 cao và giữ ID nhất quán theo đúng quy định che khuất dưới 25 frame.`

### Ca 2
- Clip / frame / ID: `clip_02 / frame 108 / T# (model ReID phát hiện thêm đối tượng)`
- Tình huống: `Model phát hiện thêm một chiếc xe nhỏ ở rìa khung hình mà nhãn ban đầu bỏ sót.`
- Quyết định: `Kiểm tra kỹ frame 108, bổ sung BBox cho đối tượng hợp lệ mới xuất hiện từ phía sau vật cản.`
- Lý do: `Giảm thiểu số lượng False Negatives (FN) và cải thiện độ chính xác phát hiện (DetA).`

### Ca 3
- Clip / frame / ID: `clip_01 / từ frame 16 đến 116 (43 frame)`
- Tình huống: `Model BoT-SORT + ReID theo dõi một vật thể tĩnh (vật cản ven đường) liên tục không có trong gold.`
- Quyết định: `Bỏ qua không gán nhãn cho vật thể tĩnh này.`
- Lý do: `Tránh tạo các False Positives (FP) do detector nhầm lẫn các vật thể không phải xe bốn bánh.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Luật gán nhãn ra/vào khung hình: Luôn kiểm tra kỹ các track tại thời điểm vật thể bắt đầu xuất hiện hoặc biến mất; sử dụng outside chính xác ở frame đầu/cuối để tránh BBox thừa/treo ở rìa clip và chỉ gán nhãn cho phương tiện di chuyển.`
- `Luật chống BBox trôi: Giữa các keyframe, cần kiểm tra ngẫu nhiên để đảm bảo BBox không bị lệch; nếu IoU với vật thể thực tế xuống thấp (< 0.6), bắt buộc phải thêm keyframe hoặc tinh chỉnh lại ở các frame trung gian.`