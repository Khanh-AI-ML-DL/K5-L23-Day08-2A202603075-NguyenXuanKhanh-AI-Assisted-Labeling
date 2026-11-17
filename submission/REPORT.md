# Báo cáo Day 08 — 2A202603075 - Nguyễn Xuân Khánh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.
>
> **Trạng thái:** mục 1–3 đã điền từ số liệu (`model_log.csv`, `threshold_sweep.md`, `review_stats.md`). Chỗ còn `<<...>>` là phần chỉ bạn trả lời được (nhìn ảnh, thời gian, quyết định). Mục 4–8 chưa điền.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`--size m`) | 0.3 (mặc định, không ngưỡng riêng) | Quét ngưỡng trên 4 ảnh gold: không ngưỡng nào <= 0.5 đạt recall 90% (person tối đa 79% / 28 vật; motorbike 80% / 20; car 89% / 9; bicycle 75% / 8), nên giữ mặc định. bus, truck có 0 vật trong gold (< 5) nên không chọn ngưỡng riêng. So hai ngưỡng ở `t_y26` (conf 0.25: 215 box) và `t_y26_c05` (conf 0.5: 139 box): ngưỡng 0.5 mất 76 box (person 131→87, motorbike 55→32). Chạy toàn tập ở conf 0.05 (`y26_all`: 1669 box, 0.14 s/ảnh) rồi lọc bằng `merge_prelabels`. |
| three_wheeler | YOLOE , prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | Lớp duy nhất đạt mục tiêu: recall 100%, precision 78% trên 7 vật gold. YOLO26 không có lớp ba-gác trong COCO; `t_yolo` (conf 0.25) tìm được 8 box trên 8 ảnh pilot, `yoloe_all` (conf 0.05) cho 125 ứng viên trên 36 ảnh. |

- Chuỗi ngưỡng dùng ở Part 4: `--thr "three_wheeler=0.15"`
- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  `yoloe_all` đứng trước nên box three_wheeler thắng box car/truck của YOLO26 khi chồng nhau (IoU >= 0.6).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: đã thử SAM 2.1 (`sam2.1_b.pt`) ở `t_sam`: 16/16 box có mask nhưng chậm, 2.81 s/ảnh so với 0.62 s/ảnh của YOLO26 thường (gấp ≈ 4.5 lần), chỉ chạy 2 ảnh. Phiên bản chốt cho Part 5 và lý do (chất lượng mask, RAM, quyền truy cập): 
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>> (gợi ý để đối chiếu: ở bước duyệt có 5 trường hợp đổi lớp, xem mục 2)
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): 
- Giới hạn: ngưỡng được chọn và chấm trên cùng 4 ảnh gold, nên precision/recall của máy trên gold là con số lạc quan (in-sample). Chỉ 4 ảnh nên ngưỡng là ước lượng thô; lớp ít vật cho kết luận yếu (three_wheeler 7 vật, bicycle 8, car 9). Ngưỡng 0.15 của three_wheeler chỉ dựa trên 7 vật nên có thể không giữ được trên tập lớn hơn.

## 2. AI error profile (từ `qc/review_stats.md`)

Ngưỡng accepted: IoU >= 0.9. Đã loại 27 đối tượng nằm quanh các bẫy khỏi bảng này (lỗi nhân tạo, không phản ánh chất lượng máy).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 507 | 97.7% | 519 box máy |
| edited | 0 | 0.0% | 519 box máy |
| deleted | 12 | 2.3% | 519 box máy |
| added | 10 | 1.9% | 517 đối tượng sau duyệt |

- Trong số deleted + added có **5 trường hợp đổi lớp** (máy gán sai lớp, box đúng chỗ), không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 10 − 5 = 5.
- Lớp nào máy tệ nhất? **three_wheeler**: bị xoá 7 trên 37 box máy (30 accepted + 7 deleted) = 18.9%, cao nhất trong các lớp có đủ mẫu (person 1/272 = 0.4%, motorbike 0/139, car 0/54). truck (2/6 = 33%) và bus (1/3) có tỉ lệ cao hơn nhưng chỉ 6 và 3 box nên không đủ để kết luận. Về bỏ sót, **car** có added nhiều nhất (4). Ảnh ví dụ: <<img_0xx>>. Nguyên nhân (đối chiếu: three_wheeler bị xoá nhiều trong khi car được thêm nhiều có thể là ba-gác bị gọi nhầm `car`, bạn cần nhìn ảnh để xác nhận): <<>>
- So với gold: precision / recall của **máy** = 66.2% (KTC 95%: 55–76%) / 68.1% (KTC 95%: 57–78%). TP 49, FP 25, FN 23. Mẫu số: 72 đối tượng gold (recall), 74 box máy (precision). Lạc quan vì ngưỡng chọn trên chính 4 ảnh này; đọc khoảng tin cậy, không đọc con số đơn.

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

## 3. Thiên kiến tự động hóa và ảnh bẫy

Bắt bẫy: 1/12 = 8.3% (KTC 95%: 1–35%), ngưỡng đạt là 80%. Theo loại: extra 0/3, missing 1/3, loose 0/3, wrong_class 0/3.
Loại nào lọt: cả 4 loại, nhiều nhất là extra, loose và wrong_class (lọt hết 3/3 mỗi loại). Danh sách lọt: img_026, img_009, img_034 (extra); img_010, img_014 (missing); img_003, img_024, img_017 (loose); img_033, img_022, img_035 (wrong_class). Chỉ có 12 bẫy nên khoảng tin cậy rất rộng.
Vì sao (bạn đã bỏ bước nào của checklist)?
Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt? Accepted là 97.7% và edited là 0/519, trong khi bắt bẫy chỉ 8.3% và máy trên gold chỉ đạt precision 66.2% / recall 68.1%. Máy kém hơn nhiều so với tỉ lệ bạn chấp nhận, và không có box nào được sửa (kể cả loại bẫy "loose" 0/3) cho thấy tôi có xu hướng chấp nhận nhãn máy thay vì kiểm từng box. Đây là suy luận từ số liệu, bạn viết lại theo cách bạn thực sự duyệt:
## 4. Máy có đáng dùng không?

Thời gian: duyệt có pre-label mất 14 phút / 24 ảnh = 0,58 phút/ảnh (nguồn: time_log.csv, theo thông tin tổng hợp trong STUDENT.md). Thời gian gán tay chưa được cung cấp nên chưa thể tính mức tiết kiệm hoặc so sánh công bằng.
Thiên lệch: người thực hiện đã gán gold trước khi đánh giá pre-label; vì vậy có thể đã quen với cảnh và đối tượng trong ảnh. Đây là nguy cơ ảnh hưởng kết quả, không có số đo độc lập để định lượng.
Kết luận: chỉ nên dùng AI làm pre-label hỗ trợ, không dùng nhãn máy như nhãn cuối cùng. Cơ sở: precision/recall trên gold lần lượt 66,2% / 68,1% (TP=49, FP=25, FN=23; precision có mẫu số 74 box máy, recall có mẫu số 72 đối tượng gold); bắt bẫy chỉ 1/12 = 8,3%; tỷ lệ accepted cao không đồng nghĩa nhãn đúng.
Ba quy tắc giảm thiên kiến đề xuất cho lần duyệt lại:1. Quét toàn ảnh và tự tìm đối tượng trước khi xem box AI; ghi nhận số đối tượng tự phát hiện.
2. Kiểm tra từng box về lớp, vị trí, kích thước và đối tượng bị bỏ sót; không chấp nhận chỉ vì box có sẵn.
3. Dùng checklist và phiếu QC với error_type đúng các giá trị hợp lệ (khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask); kiểm tra lại các ca lỗi và ảnh bẫy trước khi chốt.
## 5. Mask SAM cho 3 vật (Part 5)

 | Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos + 1 neg), chạy 1 lần | có / không gồm người lái; có / không gồm hàng hoá>> | Mask 317.255 px (45.4% ảnh), 1 polygon. Đã mở ra kiểm tra, mask ôm đúng vật, không tràn ra nền. Diện tích lớn vì xe chụp cận cảnh. <<lỗi khác nếu có, ví dụ thiếu mui / bánh>> | `submission/masks/tuktuk_img013.jpg` |
| img_004 (tuk-tuk) | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos + 1 neg), chạy 1 lần | có / không gồm người lái; có / không gồm hàng hoá>> | Mask 93.190 px (13.3% ảnh), 1 polygon. <<lỗi quan sát sau khi mở mask: tràn nền, thiếu phần nào>> | `submission/masks/tuktuk_img004_sam2.jpg` |
| img_017 (tuk-tuk) | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos + 1 neg), chạy 1 lần | có / không gồm người lái; có / không gồm hàng hoá>> | Mask 84.532 px (12.1% ảnh), 1 polygon. <<lỗi quan sát sau khi mở mask: tràn nền, thiếu phần nào>> | `submission/masks/tuktuk_img017_sam2.jpg` |
 
Cả 3 vật dùng cùng SAM 2.1 và cùng số click (3), mỗi lần chạy 1 lần ra 1 polygon. Lệnh đã dùng:
 
```
python3 scripts/sam2_points.py --image img_013.jpg --pos "400,480" --pos "600,110" --neg "720,300" --sam sam2.1_b.pt --out submission/masks --name tuktuk_img013
python3 scripts/sam2_points.py --image img_004.jpg --pos "620,480" --pos "700,560" --neg "465,450" --sam sam2.1_b.pt --out submission/masks --name tuktuk_img004_sam2
python3 scripts/sam2_points.py --image img_017.jpg --pos "560,420" --pos "680,500" --neg "645,355" --sam sam2.1_b.pt --out submission/masks --name tuktuk_img017_sam2
```

## 6. Calibration (Part 6)

- Số quyết định:  (`calibration_decisions.csv`): 
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: 
- Nguyên nhân gốc: guideline mơ hồ / chưa nắm quy tắc / máy

## 7. QC và quyết định (Part 6)

7.1. Mẫu và kết quả kiểm tra
- Mẫu QC: 5/24 ảnh batch (0 ca khó + 5 ảnh ngẫu nhiên, seed 42), tương ứng 106 đối tượng. Người kiểm: chính học viên; đây là hạn chế vì người tạo/duyệt nhãn cũng là người tự QC.
- Kết quả script tạm tính: lỗi theo đối tượng 0/106 = 0,0% (mẫu số 106 đối tượng trong mẫu); ảnh có lỗi 0/5 = 0,0% (mẫu số 5 ảnh). Tuy nhiên, không được xem đây là kết quả QC hợp lệ: log qc_report_auto.md ghi nhiều dòng có error_type không đúng enum nên bị bỏ qua, đồng thời liệt kê cả 5 ảnh img_003.jpg, img_006.jpg, img_014.jpg, img_031.jpg, img_035.jpg là chưa kiểm. Vì vậy, chưa có tỷ lệ lỗi thực tế đáng tin cậy để báo cáo.
- Top lỗi: chưa thể xếp hạng từ báo cáo tự động vì các dòng lỗi nhập bằng tiếng Việt tự do như thừa, bỏ sót, sai vị trí/kích thước không khớp enum mà script yêu cầu. Nhật ký review trước đó có các loại vấn đề như sai lớp, bỏ sót và box thừa, nhưng không thể dùng chúng thay cho kết quả QC mẫu độc lập hoặc gán ảnh ví dụ khi chưa có bảng QC hợp lệ.
- Giới hạn mẫu nhỏ: nếu tạm giả định 0/5 ảnh có lỗi là kết quả đúng, cận trên 95% theo Wilson vẫn là 43%. Mẫu 5 ảnh quá nhỏ để chứng minh tỷ lệ ảnh lỗi không vượt ngưỡng Release 2%. Hơn nữa, do lỗi định dạng phiếu và ảnh chưa được xác nhận hoàn tất, ngay cả kết quả 0/5 cũng chỉ là số tạm tính, không phải bằng chứng đạt QC.
- Script kiểm được: đếm và tổng hợp các lỗi đã được nhập đúng mã error_type, tính tỷ lệ trên số ảnh/đối tượng có trong phiếu.
- Script không tự kiểm được: chất lượng hình học thực tế của box/mask, đối tượng bị bỏ sót nếu người QC không phát hiện, việc người duyệt có làm đúng checklist hay không, và các lỗi bị nhập sai enum. Script chỉ tổng hợp dữ liệu trong phiếu, không thay thế việc xem ảnh.


7.2. Quyết định
Quyết định: REWORK — ngày 2026-11-17.

Lý do theo cây quyết định Hold → Rework → Release:
- Chưa chọn Hold vì log hiện có chưa chứng minh guideline hoặc dữ liệu nguồn bị hỏng đến mức không thể tiếp tục; tuy nhiên cần kiểm tra lại phiếu và quy tắc nhập trước khi đánh giá.
- Chọn Rework vì tỷ lệ bắt bẫy chỉ 1/12 = 8,3% (mẫu số 12 bẫy; ngưỡng yêu cầu 80%), trong đó các loại extra, loose và wrong_class đều lọt 3/3. Báo cáo QC còn cảnh báo các dòng nhập sai enum và cả 5 ảnh mẫu chưa được kiểm đầy đủ.
- Chưa đủ điều kiện Release: số 0/106 và 0/5 chưa đáng tin; cận trên 95% 43% cao hơn ngưỡng ảnh lỗi 2% rất nhiều.
Việc cần làm tiếp:
1. Sửa submission/qc/qc_review.csv, dùng đúng mã error_type theo hướng dẫn; không nhập nhãn tự do có dấu vào cột mã.
2. Mở và kiểm tra đủ 5 ảnh mẫu cùng toàn bộ đối tượng; ghi lỗi, loại lỗi và ví dụ ảnh/đối tượng cụ thể.
3. Chạy lại python3 scripts/qc_report.py; xác nhận không còn cảnh báo dòng bị bỏ qua và không còn ảnh chưa kiểm.
4. Rà soát lại checklist duyệt, đặc biệt các lỗi extra, loose, wrong_class và missing; sau đó lấy mẫu QC mới theo quy trình.
5. Chỉ xem xét Release khi có kết quả QC hợp lệ, đủ bằng chứng theo ngưỡng dự án và nêu rõ giới hạn thống kê.
## 8. Ghi chú cá nhân

- Dùng AI để hỗ trợ làm bài: có sử dụng ChatGPT hỗ trợ tổng hợp nhật ký, diễn đạt và hoàn thiện báo cáo. Các con số trong báo cáo được đối chiếu với log/script do học viên cung cấp; phần không có bằng chứng được ghi rõ là chưa xác nhận, không tự suy đoán.
- Gold có bị nhiễm không: chưa đủ thông tin để xác nhận học viên có xem pre-label hoặc hỏi AI trước khi hoàn tất gán gold hay không. Cần tự xác nhận trung thực trước khi nộp.
- Máy / GPU / CVAT: macOS trên MacBook Air, GPU Apple Silicon MPS. Log môi trường ghi Python 3.15.2 và cảnh báo nên dùng Python 3.10–3.13; CVAT chạy Docker nhưng chưa có phiên bản trong tài liệu.
- Cache data/cache/: chưa có thông tin xác nhận có dùng cache hay không.
- Một điều làm khác lần sau: hoàn tất checklist và kiểm tra nhãn độc lập trước khi nhìn pre-label; chuẩn hóa mã lỗi trong phiếu QC trước khi chạy báo cáo; không kết luận đạt QC chỉ dựa vào tỷ lệ accepted hoặc một mẫu rất nhỏ.