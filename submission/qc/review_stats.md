# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 505; đối tượng sau duyệt: 535; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 429 | 429/505 = 85.0% | 505 box máy |
| edited (sửa) | 26 | 26/505 = 5.1% | 505 box máy |
| deleted (xoá) | 50 | 50/505 = 9.9% | 505 box máy |
| added (vẽ thêm) | 80 | 80/535 = 15.0% | 535 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 69.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 259 | 4 | 8 | 19 |
| bicycle | 6 | 0 | 2 | 3 |
| motorbike | 119 | 10 | 6 | 20 |
| car | 36 | 1 | 2 | 27 |
| bus | 2 | 0 | 1 | 2 |
| truck | 1 | 0 | 5 | 3 |
| three_wheeler | 6 | 11 | 26 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 91 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 55 | 18 | 36 | 75.3% (KTC 95%: 64–84%) | 60.4% (KTC 95%: 50–70%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - missing: 2/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_022.jpg (loose), img_033.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
