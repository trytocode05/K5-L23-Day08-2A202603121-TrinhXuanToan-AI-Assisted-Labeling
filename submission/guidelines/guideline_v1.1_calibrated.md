# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603121 - Trịnh Xuân Toàn

> Bổ sung cho guideline v1.0; không thay thế các định nghĩa/lớp gốc. Các ca dưới đây gắn với `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|
| Xe đạp điện hay xe máy điện? | Xét cấu tạo và bằng chứng nhìn thấy trong từng ảnh; không suy luận chỉ từ dáng xe. Nếu không đủ căn cứ, chọn lớp tin nhất và tick `can_review`. Chưa có ca xe điện cụ thể được xác nhận trong bộ ảnh. | Chưa có ảnh xác nhận | Chưa chốt |
| Cụm xe hai bánh dày đặc gần camera | Gán riêng từng xe nếu phân biệt được và chiều cao box đạt 20 px; nếu không thể tách đáng tin cậy, dùng một `ignore_region`. | img_036 | CAL-03 |

## 2. Ca hiệu chỉnh từ dữ liệu

| Ảnh | Tình huống | Bằng chứng | Quyết định / quy tắc bổ sung | `case_id` |
|---|---|---|---|---|
| img_001 | Xe bán tải | Model gán `truck`; gold gán `car`, phù hợp taxonomy (pickup thuộc car). | Đối chiếu định nghĩa lớp trước khi accept; pickup thuộc `car`. | CAL-01 |
| img_006 | Bẫy box lỏng bị lọt | Báo cáo review ghi nhận bẫy `loose` không được bắt. | So cạnh box với biên vật; không accept chỉ theo confidence hoặc tên lớp. | CAL-02 |
| img_036 | Xe hai bánh dày/nhỏ | Ca khó về việc tách từng vật; quy tắc kích thước tối thiểu là 20 px. | Chỉ gán từng xe rõ và đủ lớn; vùng dày không thể tách dùng `ignore_region`; ca không chắc tick `can_review`. | CAL-03 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ pickup thuộc `car`; thêm bước kiểm box lỏng; chốt cách xử lý cụm xe dày. Câu hỏi xe điện vẫn để mở do chưa có ví dụ ảnh xác nhận. | CAL-01, CAL-02, CAL-03 |
