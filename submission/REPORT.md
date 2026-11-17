# Báo cáo Day 08 — 2A202603121 - Trịnh Xuân Toàn

## 1. Model và recipe (Part 2–3)

Các lần chạy ghi trong `model_log.csv` lấy từ `work/runs.csv` của Colab T4: YOLO26m và YOLOE-s trên 8 ảnh pilot; YOLO26m + SAM 2.1 trên 2 ảnh; sau đó YOLO26m/YOLOE-s trên 36 ảnh. Các ảnh pilot được dùng thử model, không dùng làm nhãn gold.

| Lớp | Model | Ngưỡng dùng khi merge | Căn cứ |
|---|---|---:|---|
| person, bicycle, motorbike, car, bus, truck | YOLO26m | mặc định 0.3 | Không có ngưỡng <= 0.5 đạt recall mục tiêu 90% trên gold; bus/truck quá ít mẫu để hiệu chỉnh riêng. |
| three_wheeler | YOLOE-s | 0.15 | Trên 7 vật gold, recall 100%, precision 78%; chỉ là ước lượng nhỏ. |

Lệnh gộp đã chạy:

```powershell
python scripts\merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work\prelabel\merged_review\pre.coco.json
```

SAM 2.1 (`sam2.1_b.pt`) được thử qua detector → SAM trên 2 ảnh, tạo 16 mask theo log Colab. Ba overlay click-SAM thật hiện được lưu trong `submission/masks/`: `tuktuk_img013.jpg` (lấy từ ZIP Colab), `tuk_tuk_img023.jpg` và `tuk_tuk_img004.jpg` (chạy bằng CPU local). Đã xem trực quan các overlay; đây là kết quả gợi ý, còn vùng rỗng/không đều nên không coi là mask hoàn hảo.

Ví dụ lỗi không sửa được chỉ bằng ngưỡng: ở `img_001`, YOLO26 gán xe bán tải là `truck`, còn taxonomy/gold là `car`. Cần kiểm tra và sửa lớp. Ngưỡng lớp được chọn trên đúng 4 ảnh gold nên precision/recall trên gold lạc quan; mẫu nhỏ không đại diện toàn bộ dữ liệu.

## 2. AI error profile

Nguồn: `submission/qc/review_stats.md`; các lỗi bẫy nhân tạo được loại khỏi bảng này.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 478 | 92.5% | 517 box máy |
| edited | 7 | 1.4% | 517 box máy |
| deleted | 32 | 6.2% | 517 box máy |
| added | 8 | 1.6% | 493 đối tượng sau duyệt |

Theo gold: máy có TP=52, FP=22, FN=43 trên 95 đối tượng gold; precision 52/74 = 70.3%, recall 52/95 = 54.7%. Trong review, lớp `three_wheeler` có 25 box bị xóa, 6 chỉnh và 2 thêm; đây là lớp cần kiểm kỹ vì định nghĩa ngoài tập COCO và dễ nhầm với xe khác.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Bắt được 4/12 bẫy (33.3%), thấp hơn mục tiêu 80%: extra 3/3, wrong_class 1/3, missing 0/3, loose 0/3. Các ảnh bẫy lọt được ghi trong `review_stats.md`. Accepted 92.5% không chứng minh đã review tốt: bẫy cho thấy lỗi thiếu/sai lớp/box lỏng vẫn có thể qua mắt khi người duyệt dựa vào nhãn máy.

Quy tắc review áp dụng: quét toàn ảnh trước khi hiện nhãn; kiểm từng lớp chứ không chỉ nhìn confidence; so sát biên box với vật trước khi accept. Kết quả hiện tại cho thấy cần rework và siết checklist.

## 4. Máy có đáng dùng không?

Thời gian học viên cung cấp: gán gold 20 phút/4 ảnh = 5 phút/ảnh; duyệt batch 20 phút/24 ảnh = khoảng 0.83 phút/ảnh. Đây là hai công việc khác nhau và batch review có thể chưa phản ánh đầy đủ thời gian rework/QC; vì vậy không kết luận mức tiết kiệm tổng thể chỉ từ phép chia này.

Kết luận: dùng model để tạo gợi ý ban đầu, nhưng không dùng đầu ra chưa review làm nhãn cuối. Căn cứ: 92.5% box máy được giữ nguyên trong phép ghép theo IoU, nhưng bắt bẫy chỉ 33.3% và QC mẫu có lỗi.

Ba quy tắc giảm thiên kiến:
1. Rà toàn ảnh để tìm vật bị bỏ sót trước khi xem box máy.
2. Với từng box, xác minh đối tượng có thật, đúng lớp và box ôm sát phần nhìn thấy.
3. Với lớp ba bánh, vật nhỏ và cụm xe dày, đối chiếu guideline; đánh dấu `can_review` khi còn mơ hồ thay vì accept nhanh.

## 5. Mask SAM cho 3 vật

Đã tạo 3 overlay click-SAM với SAM 2.1 base:

| Ảnh | Số click | Quan sát |
|---|---:|---|
| img_013 | 3 (2 dương, 1 âm) | Bao vùng tuk-tuk lớn; mask còn các khoảng rỗng quanh người/ngăn xe, cần xem xét nếu dùng làm nhãn pixel chính xác. |
| img_023 | 3 (2 dương, 1 âm) | Tách được tuk-tuk phía trước; cần rà lại biên bánh và phần hàng/ghế. |
| img_004 | 3 (2 dương, 1 âm) | Bao tuk-tuk màu xanh; rà lại vùng kính/người và các chi tiết quanh bánh. |

Các ảnh overlay được lưu ở `submission/masks/`. SAM tạo mask theo điểm gợi ý, không xác nhận mask đúng tuyệt đối; người gán nhãn vẫn phải kiểm tra quy tắc về người lái, hàng hóa, kính và phần nhìn thấy.

## 6. Calibration

Đã ghi 3 ca trong `submission/qc/calibration_decisions.csv`:
- CAL-01: pickup ở `img_001` — theo taxonomy, lớp đúng là `car`.
- CAL-02: box lỏng bị lọt ở `img_006` — kiểm cạnh box trước khi accept.
- CAL-03: cụm xe hai bánh ở `img_036` — chỉ gán riêng vật rõ và đủ lớn; cụm không tách được dùng `ignore_region`.

Xe đạp điện chưa có ví dụ ảnh xác nhận trong dữ liệu đã xem; giữ quyết định theo từng ca và dùng `can_review` khi không đủ căn cứ. Các bổ sung có trong `guideline_v1.1_calibrated.md`.

## 7. QC và quyết định

Mẫu QC seed 42 gồm 5 ảnh ngẫu nhiên, 99 đối tượng; người kiểm là học viên. Ghi nhận: `img_006` có lỗi bỏ sót, `img_035` có lỗi sai lớp. Do đó 2/5 ảnh có lỗi (40%); phiếu ghi 2 lỗi trên 99 đối tượng (2.0%). Cận trên Wilson 95% cho tỉ lệ ảnh lỗi là 77%; mẫu nhỏ nên không chứng minh được mức lỗi thật của toàn bộ batch.

Script kiểm tính nhất quán COCO, phạm vi ảnh, lớp, kích thước và số liệu QC; không tự xác định vật thật, độ ôm sát box bằng mắt, hay việc người duyệt có làm đúng checklist không. Mẫu này do chính học viên review nên có thể còn thiên lệch.

**Quyết định: Rework**, ngày 2026-11-17. Lý do: lỗi QC 40% theo ảnh vượt ngưỡng 2% và bắt bẫy 33.3% thấp hơn mục tiêu 80%. Cần sửa các ảnh có lỗi trong CVAT, kiểm tra lại checklist và lấy mẫu QC lại trước khi chuyển Release.

## 8. Ghi chú cá nhân

- Có dùng Copilot để đọc cấu trúc repo, chạy validator/QC, tạo hai overlay SAM trên máy local và tổng hợp số liệu; đã xem ảnh overlay và đối chiếu số liệu script. Model pre-label được chạy trên Colab T4.
- Gold được gán tay trước bước pre-label toàn bộ ảnh; file gold cuối có 4 ảnh, 95 đối tượng và validator báo OK.
- Dùng CVAT Online; không chạy Docker CVAT trên máy local.
- Dữ liệu dự đoán đến từ các lần chạy Colab, không dùng cache `data/cache/`.
- Từng gặp export nhầm JSON: validator phát hiện file không đúng bộ gold; sau đó đã thay bằng export đúng 4 ảnh và xác nhận hợp lệ.
- Trước khi nộp còn cần sửa và export lại các lỗi QC, cập nhật số liệu sau rework và chạy `check_submission.py`.
