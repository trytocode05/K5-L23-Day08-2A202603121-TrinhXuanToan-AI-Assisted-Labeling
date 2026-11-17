# Thông tin học viên

- Họ và tên: Trịnh Xuân Toàn
- MSHV: 2A202603121
- Lớp: chưa cung cấp
- Tài khoản GitHub: trytocode05
- Repo bài nộp: https://github.com/trytocode05/K5-L23-Day08-2A202603121-TrinhXuanToan-AI-Assisted-Labeling
- Commit chốt bài: chưa tạo
- Máy: Windows; suy luận model trên Google Colab T4; máy local dùng Python 3.12
- CVAT: CVAT Online (không dùng Docker local; chưa ghi phiên bản)

## Kết quả tóm tắt

- Máy: accepted 478/517 (92.5%), edited 7/517 (1.4%), deleted 32/517 (6.2%); added 8/493 đối tượng cuối (1.6%).
- So với gold: precision 52/74 = 70.3%; recall 52/95 = 54.7% (4 ảnh gold).
- Bắt bẫy: 4/12 = 33.3%.
- Thời gian tự báo: gán gold 20 phút/4 ảnh; review batch 20 phút/24 ảnh.
- QC mẫu: 2/5 ảnh có lỗi (40%); 2 ghi nhận lỗi trên 99 đối tượng được lấy mẫu (2.0%).
- Quyết định: Rework — QC mẫu có lỗi vượt ngưỡng 2% và bắt bẫy dưới mục tiêu 80%.
- Đã có 3 ảnh overlay click-SAM trong `submission/masks/`; còn cần rework nhãn/QC và tạo commit cuối.
