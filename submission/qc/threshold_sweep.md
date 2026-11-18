# Quét ngưỡng theo lớp

Mục tiêu recall: 90%. Ảnh dùng để quét: 4.

| Lớp | Ngưỡng gợi ý | Lý do |
|---|---|---|
| person | 0.1 | recall 93% (>= 90%), precision 49% (27 vật gold) |
| bicycle | 0.2 | recall 100% (>= 90%), precision 71% (5 vật gold) |
| motorbike | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 17 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| car | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 8 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| bus | (mặc định) | chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| truck | (mặc định) | chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| three_wheeler | 0.15 | recall 100% (>= 90%), precision 78% (7 vật gold) |

Dán vào lệnh merge_prelabels (lớp không có trong chuỗi dùng `--conf` mặc định 0.3): `--thr "person=0.1,bicycle=0.2,three_wheeler=0.15"`

Lưu ý: bạn chọn ngưỡng trên chính nhãn gold, nên precision / recall của máy trên gold ở ngưỡng đã chọn là con số **lạc quan** (chọn và chấm trên cùng dữ liệu); chỉ có vài ảnh -> ngưỡng là ước lượng thô; lớp có ít vật (xem n_gold) thì kết luận yếu.
