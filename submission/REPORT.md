# Báo cáo Day 08 — 2A202603319 - Phạm Vũ Tùng Dương

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Đề xuất sweep: person=0.1, bicycle=0.2; motorbike/car/bus/truck dùng mặc định | r01/r02: YOLO26m bắt nhiều vật nhưng còn box dày/chồng lấn và nhầm xe ba bánh; conf 0.5 giảm box nhưng bỏ vật nhỏ/xa. |
| three_wheeler | YOLOE-26s, prompt three_wheeler | Đề xuất sweep: 0.15; thực tế không đặt lớp này trong chuỗi `--thr` nên dùng mặc định | r03: tìm được một số tuk-tuk/cyclo nhưng kết quả thiếu ở 5/8 ảnh pilot; chỉ dùng để gợi ý và phải rà tay. |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py --pre work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn:** SAM 2.1 B. SAM 3 bị chặn quyền truy cập (403); SAM 2.1 B đã tải và chạy được. Ba mask nháp được lưu trong `submission/masks/`.
- Lỗi không sửa được chỉ bằng đổi ngưỡng: YOLO26 có thể gọi tuk-tuk/xích lô là `car`/`truck`; dùng YOLOE làm gợi ý và sửa lớp theo hình ảnh/guideline.
- Box điểm cao sai hoặc điểm thấp đúng: không ghi lại được ví dụ gắn với confidence và ID cụ thể; không kết luận từ số liệu hiện có.
- Giới hạn: ngưỡng được chọn trên chính 4 ảnh gold; precision/recall trên gold vì vậy lạc quan và mẫu nhỏ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 385 | 87.9% | 438 box máy |
| edited | 12 | 2.7% | 438 box máy |
| deleted | 41 | 9.4% | 438 box máy |
| added | 20 | 4.8% | 417 đối tượng sau duyệt |

- Lớp máy yếu nhất: `three_wheeler` có 19 box bị xoá; YOLO26 có thể nhầm xe ba bánh thành `car`/`truck`. Ví dụ pilot được ghi trong `model_log.csv` là img_002 và img_032.
- So với gold: precision / recall của **máy** 65.2% / 66.2% (mẫu số: 65 đối tượng gold; 43 TP, 23 FP, 22 FN).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3%; lọt `loose` ở img_026 và `missing` ở img_035 (`review_stats.md`). Bước checklist bị bỏ sót cụ thể không được ghi lại nên không suy đoán nguyên nhân cá nhân.
- Accepted 385/438 = 87.9% cùng bắt bẫy 10/12 = 83.3% cho thấy phần lớn box máy giữ được nhưng vẫn cần kiểm tra độc lập; accepted cao không đảm bảo bắt hết lỗi.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay 30/4 = 7.5 phút/ảnh; duyệt batch 90/24 = 3.75 phút/ảnh, gồm lượt đầu và lượt rà lại. Hai nhóm ảnh khác độ khó nên đây không phải so sánh thời gian thuần tương đương. Gold được gán trước khi chạy pre-label theo trình tự đã ghi trong lịch sử; vẫn có thể quen cảnh.
- Kết luận: dùng máy làm bản nháp cho người duyệt, không dùng tự động. Căn cứ: 385/438 box máy được giữ, nhưng precision 43/66 = 65.2%, recall 43/65 = 66.2%, và có 20/417 đối tượng sau duyệt được thêm.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét toàn ảnh trước khi nhìn pre-label; ghi lại vật sót tìm được.
  2. Rà riêng từng box `can_review` và mọi dự đoán `three_wheeler`; không chấp nhận chỉ dựa trên confidence.
  3. Kiểm tra nền, tiền cảnh và mép ảnh sau lượt sửa; đối chiếu số bẫy bắt được trên 12 bẫy.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004 | SAM 2.1 B | 3 (2 dương, 1 âm) | Không thấy người đi bộ trong mask xe; không đánh giá được mọi chi tiết nhỏ | Mask xe bám phần lớn tuk-tuk, còn cần rà biên thủ công | `img_004_tuktuk_sam21.jpg` |
| img_013 | SAM 2.1 B | 4 (2 dương, 2 âm) | Click âm loại được một phần người; còn dính vùng sát người lái | 232403 px (33.3% ảnh), một polygon; còn lỗ/nhiễu trong khung xe | `img_013_tuktuk_sam21.jpg` |
| img_003 | SAM 2.1 B | 6 (2 dương, 4 âm) | Mask vẫn dính một phần người ngồi trên xe; hàng hoá gắn trên xe nằm trong vùng mask | Thử thêm click âm nhưng vẫn cần tách người và sửa mask bằng tay | `img_003_cargo_trike_sam21.jpg` |

- So sánh cùng prompt trên `img_013`: MobileSAM tạo mask 246462 px (35.3%) trong 2.83s, SAM 2.1 B tạo 232403 px (33.3%) trong 3.89s. Ở ảnh này MobileSAM cho thân/mui xe liền mạch hơn và phủ ít lên thân người lái hơn; vẫn dính một phần chân, và diện tích lớn hơn 14059 px nên cần kiểm tra biên. Đây là quan sát một ảnh duy nhất, không đủ kết luận MobileSAM tốt hơn nói chung. Overlay: `img_013_tuktuk_mobile_sam.jpg`.
- Đối chiếu guideline mục 4.2: quy tắc mask theo phần nhìn thấy, bỏ bóng đổ và chú ý lỗ khung giúp kiểm tra kết quả; quy tắc chưa đủ là xử lý người/hàng hoá sát xe, vì mask img_003 còn dính người và các vùng khung tạo lỗ/nhiễu.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không có lần thử được ghi nhận; so sánh với offline chưa đánh giá.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất: CAL-01, xác minh lớp `three_wheeler` bằng hình thái thay vì giữ nhãn YOLO26.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: tách từng xe còn nhận dạng được trong cụm gần camera; kiểm tra độc lập lớp xe ba bánh; quét toàn ảnh và mép ảnh để tìm vật sót.
- Nguyên nhân gốc: vừa có nhầm lớp của máy, vừa có điểm cần cụ thể hoá trong guideline và quy trình review.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh ca khó + 5 ngẫu nhiên, seed 42), 157 đối tượng (`qc_sample.csv`); kiểm tra hỗ trợ bằng cách đối chiếu ảnh gốc và overlay. Đây không thay thế xác nhận cuối của học viên.
- Lỗi theo **đối tượng**: 0/157 = 0.0% · theo **ảnh**: 0/8 = 0.0% (ngưỡng Release ≤ 2%; `qc_report_auto.md`).
- Top lỗi: chưa ghi nhận lỗi nào trong mẫu.
- **Giới hạn mẫu nhỏ**: cận trên Wilson 95% là 32%; mẫu 8 ảnh không chứng minh được tỉ lệ lỗi thật ≤ 2% (`qc_report_auto.md`).
- Script đếm lỗi đã ghi trong phiếu; không xác minh được box ôm sát vật thật, lỗi cùng bị máy và người bỏ sót, hoặc việc tuân thủ checklist.

**Quyết định: RELEASE (gợi ý QC)**, ngày 2026-11-18. Lý do: mẫu có 0/8 ảnh ghi lỗi, không phát hiện lỗi hệ thống và bắt bẫy đạt 10/12 = 83.3% (`review_stats.md`, ngưỡng 80%). Kết quả mẫu nhỏ nên không chứng minh được tỉ lệ lỗi thật ≤ 2%; học viên cần xác nhận cuối.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Copilot hỗ trợ theo dõi lệnh, đối chiếu ảnh QC, ghi chú calibration và tạo mask nháp; các kết quả ảnh/mask cần người học xác nhận trực quan.
- Gold có bị nhiễm không: lịch sử hội thoại cho thấy gold được gán và validate trước các lần chạy pre-label; không có bằng chứng về việc xem pre-label trước khi gán, nhưng người học cần tự xác nhận.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: MacBook Pro, macOS, Apple MPS, Python 3.13.18, CVAT Docker 2.74.1. SAM 3 bị từ chối quyền truy cập (403); chuyển sang SAM 2.1 B.
- Dùng cache `data/cache/`: không có việc dùng cache được ghi nhận.
- Lỗi / bug script đã gặp: một số script cần chạy với `PYTHONPATH=scripts` để tìm được dependencies/module trong virtualenv.
- Một điều làm khác lần sau: quét toàn ảnh trước khi nhìn pre-label và kiểm lại mép ảnh để giảm bỏ sót.
