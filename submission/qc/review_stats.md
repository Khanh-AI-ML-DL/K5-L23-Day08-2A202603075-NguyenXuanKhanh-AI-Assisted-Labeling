# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 517; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 507 | 507/519 = 97.7% | 519 box máy |
| edited (sửa) | 0 | 0/519 = 0.0% | 519 box máy |
| deleted (xoá) | 12 | 12/519 = 2.3% | 519 box máy |
| added (vẽ thêm) | 10 | 10/517 = 1.9% | 517 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 5.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 271 | 0 | 1 | 1 |
| bicycle | 7 | 0 | 1 | 1 |
| motorbike | 139 | 0 | 0 | 2 |
| car | 54 | 0 | 0 | 4 |
| bus | 2 | 0 | 1 | 1 |
| truck | 4 | 0 | 2 | 0 |
| three_wheeler | 30 | 0 | 7 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 72 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 49 | 25 | 23 | 66.2% (KTC 95%: 55–76%) | 68.1% (KTC 95%: 57–78%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 1/12 = 8.3% (KTC 95%: 1–35%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 0/3
  - missing: 1/3
  - loose: 0/3
  - wrong_class: 0/3
  - lọt: img_026.jpg (extra), img_010.jpg (missing), img_003.jpg (loose), img_009.jpg (extra), img_014.jpg (missing), img_034.jpg (extra), img_033.jpg (wrong_class), img_022.jpg (wrong_class), img_035.jpg (wrong_class), img_024.jpg (loose), img_017.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
