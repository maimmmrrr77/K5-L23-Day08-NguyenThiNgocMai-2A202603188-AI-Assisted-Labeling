# Thông tin học viên

- Họ và tên: Nguyễn Thị Ngọc Mai
- MSHV: 2A202603188
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: https://github.com/maimmmrrr77
- Repo bài nộp (URL): https://github.com/maimmmrrr77/K5-L23-Day08-NguyenThiNgocMai-2A202603188-AI-Assisted-Labeling
- Commit chốt bài (hash): 
- Máy: Window, GPU: không có
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 429/505 (85.0%) / 26/505 (5.1%) / 50/505 (9.9%) (mẫu số: 505 box máy); added: 80/535 (15.0%) (mẫu số: 535 đối tượng)
- So với gold: precision / recall của máy 75.3% (55/73) / 60.4% (55/91)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 8 phút/ảnh; duyệt có pre-label 4 phút/ảnh
- Lỗi QC: 14.3% theo ảnh (1/7), 0.8% theo đối tượng (1/120)
- **Quyết định:** Rework -lý do một dòng: Tỉ lệ ảnh lỗi trong mẫu QC đạt 1/7 (14.3%, cận trên Wilson 95% ~51.0%), vượt ngưỡng Release <= 2% do box id 271 (person) ở ảnh img_022.jpg bị lỏng viền cần chỉnh sửa lại.