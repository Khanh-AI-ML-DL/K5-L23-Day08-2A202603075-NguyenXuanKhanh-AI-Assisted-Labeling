# Thông tin học viên

- Họ và tên: Nguyễn Xuân Khánh
- MSHV: 2A202603075
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Khanh-AI-ML-DL
- Repo bài nộp (URL): https://github.com/Khanh-AI-ML-DL/K5-L23-Day08-2A202603075-NguyenXuanKhanh-AI-Assisted-Labeling
- Commit chốt bài (hash): https://github.com/Khanh-AI-ML-DL/K5-L23-Day08-2A202603075-NguyenXuanKhanh-AI-Assisted-Labeling
- Máy: macOS (MacBook Air), GPU: MPS (Apple Silicon)
- CVAT (Docker): Localhost

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

-Kết quả tóm tắt (điền cuối bài; số lấy từ submission/qc/*, kèm mẫu số)
- Máy: accepted / edited / deleted = 507 / 0 / 12 (mẫu số: 519 box máy; tương ứng 97,7% / 0,0% / 2,3%); added = 10 (mẫu số: 517 đối tượng sau duyệt, 1,9%).
- So với gold: precision / recall của máy = 66,2% / 68,1% (precision: 49 TP / 74 box máy; recall: 49 TP / 72 đối tượng gold; 4 ảnh; kết quả lạc quan vì ngưỡng được chọn trên chính gold).
- Bắt bẫy: 1/12 = 8,3% (KTC 95%: 1–35%; mẫu số 12 bẫy).
- Thời gian: gán tay chưa có số liệu; duyệt có pre-label 0,58 phút/ảnh (14 phút / 24 ảnh).
- QC: script tạm tính 0/106 = 0,0% theo đối tượng và 0/5 = 0,0% theo ảnh, nhưng không hợp lệ để kết luận vì nhiều error_type sai enum bị bỏ qua và 5 ảnh mẫu vẫn được ghi là chưa kiểm. Cận trên 95% của tỷ lệ ảnh lỗi nếu tạm giả định 0/5 là đúng: 43%; mẫu quá nhỏ để chứng minh ngưỡng ≤ 2%.
- Quyết định: REWORK — bắt bẫy 8,3% (1/12), dưới ngưỡng 80%; phiếu QC có lỗi định dạng và mẫu chưa được kiểm đầy đủ. Cần sửa phiếu, kiểm tra đủ ảnh mẫu, chạy lại QC rồi mới xem xét Release.
