# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 438; đối tượng sau duyệt: 417; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 385 | 385/438 = 87.9% | 438 box máy |
| edited (sửa) | 12 | 12/438 = 2.7% | 438 box máy |
| deleted (xoá) | 41 | 41/438 = 9.4% | 438 box máy |
| added (vẽ thêm) | 20 | 20/417 = 4.8% | 417 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 14.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 241 | 1 | 8 | 5 |
| bicycle | 4 | 1 | 4 | 1 |
| motorbike | 84 | 4 | 0 | 1 |
| car | 43 | 1 | 7 | 2 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 3 | 3 |
| three_wheeler | 7 | 5 | 19 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 65 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 23 | 22 | 65.2% (KTC 95%: 53–76%) | 66.2% (KTC 95%: 54–76%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - missing: 2/3
  - loose: 2/3
  - extra: 3/3
  - lọt: img_026.jpg (loose), img_035.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
