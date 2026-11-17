# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 517; đối tượng sau duyệt: 493; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 478 | 478/517 = 92.5% | 517 box máy |
| edited (sửa) | 7 | 7/517 = 1.4% | 517 box máy |
| deleted (xoá) | 32 | 32/517 = 6.2% | 517 box máy |
| added (vẽ thêm) | 8 | 8/493 = 1.6% | 493 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 4.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 265 | 1 | 1 | 0 |
| bicycle | 6 | 0 | 2 | 1 |
| motorbike | 136 | 0 | 0 | 1 |
| car | 53 | 0 | 1 | 1 |
| bus | 3 | 0 | 0 | 3 |
| truck | 3 | 0 | 3 | 0 |
| three_wheeler | 12 | 6 | 25 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 95 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 52 | 22 | 43 | 70.3% (KTC 95%: 59–79%) | 54.7% (KTC 95%: 45–64%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 1/3
  - missing: 3/3
  - extra: 3/3
  - lọt:  img_018.jpg (wrong_class), img_033.jpg (wrong_class).

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
