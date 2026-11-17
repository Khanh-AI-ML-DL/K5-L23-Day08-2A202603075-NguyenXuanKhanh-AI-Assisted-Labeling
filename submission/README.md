# Day 08 Lab — 2A202603075 - Nguyễn Xuân Khánh

Xem cấu trúc bắt buộc ở [../SUBMISSION.md](../SUBMISSION.md). Các file bạn **tự điền** (khuôn có sẵn ở đây):

| File | Điền gì | Ví dụ một dòng                                                                                                                                                         |
|---|---|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `model_log.csv` | mỗi lần chạy `prelabel.py` một dòng; copy cột có sẵn từ `work/runs.csv`, tự viết 3 cột nhận xét | `r03,2,yoloe,"--prompts 'tuk-tuk=three_wheeler' --conf 0.25",pilot,9,0.3,"tìm 4/5 tuk-tuk ở img_002","bỏ sót xích lô img_011; box lỏng ~10%","dùng cho three_wheeler"` |
| `time_log.csv` | thời gian **thật** | `gold_manual,4,25,"gán tay 4 ảnh"` · `batch_review,24,80,"duyệt 24 ảnh"`                                                                                               |
| `REPORT.md` | báo cáo chính (8 mục, khuôn trong file) |                                                                                                                                                                        |
| `guidelines/guideline_v1.1_calibrated.md` | **chỉ phần bổ sung** so với v1.0 (khuôn trong file) |                                                                                                                                                                        |
| `qc/calibration_decisions.csv` | mỗi quyết định calibration **một dòng** | `CAL-01,img_013\|img_004,loi_may,"YOLO26 gọi tuk-tuk là car","luôn gán three_wheeler","mục 2 v1.0",v1.1`                                                               |
| `qc/qc_review.csv` | do `qc_sample.py` tạo sẵn; bạn điền `error_type` |                                                                                                                                                                        |
| `masks/*.jpg` | ảnh mask từ `sam2_points.py --out submission/masks --name ...` (≥ 3) | `Mask: 317255 px (45.4% ảnh), 1 polygon -> submission/masks/tuktuk_img013.jpg ` ,`Mask: 93190 px (13.3% ảnh), 1 polygon -> submission/masks/tuktuk_img004_sam2.jpg` ,`Mask: 84532 px (12.1% ảnh), 1 polygon -> submission/masks/tuktuk_img017_sam2.jpg`  |

Các file còn lại do **script** sinh ra: **không sửa tay**.
