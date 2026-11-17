
## 1. Ngưỡng theo lớp

Quét ngưỡng bằng `scripts/threshold_sweep.py` trên 4 ảnh gold (`submission/labels/gold_manual.json`), mục tiêu recall 90%, chỉ xét ngưỡng ≤ 0.5. Chuỗi dùng ở Part 4:

`--thr "three_wheeler=0.15"`

Các lớp không có trong chuỗi dùng ngưỡng mặc định `--conf 0.3`.

| Lớp | Ngưỡng | Số vật gold | Lý do |
|---|---|---|---|
| three_wheeler | 0.15 | 7 | Recall 100% (>= 90%), precision 78%. Lớp duy nhất đạt mục tiêu. |
| person | 0.3 (mặc định) | 28 | Không ngưỡng nào <= 0.5 đạt recall 90%, tối đa 79%. |
| motorbike | 0.3 (mặc định) | 20 | Tối đa recall 80%, không đạt mục tiêu. |
| car | 0.3 (mặc định) | 9 | Tối đa recall 89%, sát nhưng chưa đạt mục tiêu. |
| bicycle | 0.3 (mặc định) | 8 | Tối đa recall 75%, không đạt mục tiêu. |
| bus | 0.3 (mặc định) | 0 | Dưới 5 vật, mẫu quá ít nên không chọn ngưỡng riêng. |
| truck | 0.3 (mặc định) | 0 | Dưới 5 vật, mẫu quá ít nên không chọn ngưỡng riêng. |

Với person, motorbike, car, bicycle, hạ ngưỡng xuống <= 0.5 vẫn không đạt recall 90%, nên giữ mặc định thay vì ép chọn. Pre-label các lớp này vẫn sẽ sót vật, người gán phải bổ sung bằng tay.

**Giới hạn:**
- Chỉ có 4 ảnh gold nên các ngưỡng chỉ là ước lượng thô; lớp ít vật (three_wheeler 7, bicycle 8, car 9) cho kết luận yếu.
- Ngưỡng được chọn và chấm trên cùng bộ gold, nên precision/recall của máy trên gold ở ngưỡng đã chọn là con số lạc quan, không phản ánh chất lượng trên dữ liệu mới.
- Ngưỡng 0.15 của three_wheeler chỉ dựa trên 7 vật, có thể không giữ được trên tập lớn hơn.
