# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603319 - Phạm Vũ Tùng Dương

> Bản bổ sung cho `guideline_v1.0_label_contract.md`; các quy tắc v1.0 khác vẫn giữ nguyên.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Gán riêng từng xe khi nhận ra được từng cá thể. Chỉ dùng `ignore_region` cho cụm không thể tách; không bỏ qua chỉ vì xe đứng sát nhau. | img_005 | CAL-02 |

Các câu hỏi mở còn lại (xe điện, hàng nhô ra, người trong xe có mui, kính xe, box chồng lấn, mức conf) chưa được chốt vì chưa có ví dụ đủ phù hợp để quyết định nhất quán.

## 2. Ca khó mới (từ lỗi máy và bẫy lọt)

| Ảnh | Tình huống | Máy làm gì | Quyết định | Quy tắc mới |
|---|---|---|---|---|
| img_002, img_032 | Tuk-tuk/xích lô bị nhầm lớp | YOLO26 có thể gán thành `car` hoặc `truck`; review batch xoá 19 box `three_wheeler` | Xác định theo cấu trúc xe ba bánh; không giữ nhãn máy nếu mâu thuẫn với ảnh. YOLOE chỉ gợi ý; người duyệt quyết định. | Mục 2, 4.3: kiểm tra số bánh/khung xe; nếu không đủ bằng chứng thì tick `can_review`. (CAL-01) |
| img_026, img_035 | Bẫy lọt ở lô review | Lọt một box loose và một vật bị bỏ sót; tổng bắt bẫy 10/12 | Trước khi chấp nhận pre-label, quét toàn ảnh độc lập với nhãn máy; sau đó kiểm từng box và rà các mép ảnh. | Checklist review: quét nền, tiền cảnh và mép ảnh để tìm vật sót; kiểm lại độ ôm box ở vùng khó. (CAL-03) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ cách gán cụm xe máy gần camera; thêm quy tắc kiểm nhãn xe ba bánh và bước quét toàn ảnh trước khi chấp nhận pre-label. | CAL-01, CAL-02, CAL-03 |
