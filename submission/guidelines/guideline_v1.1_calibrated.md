# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603075 - Nguyễn Xuân Khánh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Mỗi ca khó khớp một dòng trong `qc/calibration_decisions.csv`.
> Dòng gắn **[ĐỀ XUẤT]** là bản nháp, cần bạn xem lại ảnh và xác nhận trước khi nộp.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|--|--|
| 1 | Mask phương tiện có gồm người lái và hàng hoá không? | **[ĐỀ XUẤT]** Không gồm người lái (đã có nhãn `person`). | img_013 | CAL-02 |
| 2 | các câu hỏi khác ở mục 6 của v1.0: chốt, hoặc ghi chưa chốt và vì sao |  |  |  |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|--|---|---|---|---|
| | Ba-gác/tuk-tuk bị gọi lớp khác (5 ca đổi lớp; three_wheeler bị xoá 7/37) | YOLO26 không có lớp ba-gác nên gọi car/truck/motorbike; YOLOE cho gợi ý three_wheeler | **[ĐỀ XUẤT]** Xe ba bánh luôn là `three_wheeler` | Mục lớp three_wheeler (CAL-01) |
| img_033, img_022, img_035 | Bẫy sai lớp, lọt 3/3 | Máy giữ nguyên nhãn, tôi chấp nhận không tự kiểm lớp | **[ĐỀ XUẤT]** Tự gọi tên lớp từng box trước khi chấp nhận | Mục quy trình review, bước sai nhãn (CAL-03) |
| img_003, img_024, img_017 | Bẫy box lỏng, lọt 3/3; không box nào được sửa (edited 0/519) | Máy vẽ box lỏng, tôi giữ nguyên | **[ĐỀ XUẤT]** Box phải ôm sát; lỏng thấy được thì sửa (ngưỡng accepted IoU >= 0.9) | Mục chất lượng box (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt phạm vi mask (không gồm người lái); xe ba bánh luôn là `three_wheeler`; bắt buộc tự kiểm lớp từng box; định nghĩa box ôm sát | CAL-02, CAL-01, CAL-03, CAL-04 |
