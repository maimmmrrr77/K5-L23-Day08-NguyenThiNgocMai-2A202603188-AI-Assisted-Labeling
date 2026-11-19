# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603188 - Nguyễn Thị Ngọc Mai

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Căn cứ vào bàn đạp và sàn để chân: nếu có bàn đạp (kể cả xe đạp điện có bình ắc quy) gán `bicycle`; nếu không có bàn đạp, có sàn để chân phẳng hoặc yên liền lớn gán `motorbike`. Nếu chiều cao `< 20 px` hoặc bị che khuất không phân biệt được, gộp vào `ignore_region`. | `img_018.jpg` | `CAL-01` |
| 2 | Hàng nhô ra trên xe ba gác / xe đạp (`three_wheeler`, `bicycle`): box xe và mask có gồm hàng không? | Bounding box chỉ ôm sát khung xe, mái che và bánh xe, không kéo giãn theo hàng hóa/tre nhô quá xa. Với mask SAM (mục 4.2), loại trừ cả người lái (`person`) và hàng hóa nhô ngoài bằng điểm âm (`--neg`). | `img_013.jpg` | `CAL-02` |
| 3 | Khối xe máy đỗ dày đặc gần camera (không phải ở xa): gán từng xe hay dùng `ignore_region`? | Mọi xe máy ở gần camera còn thấy rõ khung đầu/yên/bánh và có chiều cao `≥ 20 px` đều phải vẽ tách từng box `motorbike` độc lập; chỉ gom bằng 1 box `ignore_region` khi các xe xếp chồng khít che khuất `> 50%` diện tích khiến không thể phân tách ranh giới từng chiếc bằng mắt. | `img_005.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Hai khung bao `three_wheeler` chồng đè lên cùng một chiếc xe tuk-tuk (`CAL-04`). | `YOLOE-26s` sinh 2 box `three_wheeler` trùng lặp tại cùng vị trí do ngưỡng `0.15` nhạy với các bộ phận xe. | Chỉ giữ lại 1 box ôm sát toàn bộ khung xe tuk-tuk nhất và xóa box trùng lặp dư thừa. | Bổ sung mục 3.2 (Quy tắc chống trùng lặp): không giữ 2 box cùng lớp có IoU > 0.6 trên cùng một đối tượng. |
| `img_022.jpg` | Box `person` bị nới lỏng viền ngang và cắt thấp đỉnh đầu — bẫy `loose` lọt ở vòng duyệt đầu (`CAL-05`). | Box sau bẫy bị kéo rộng sang trái và hụt phần đầu. | Thu hẹp cạnh trái vào sát thân người và kéo cạnh trên ôm trọn phần đầu để đạt IoU ≥ 0.9. | Bổ sung mục 3.1 (Quy tắc kiểm viền 4 cạnh): bắt buộc rà đủ 4 cạnh (đỉnh đầu, bàn chân, mép trái/phải) của mọi box `person`. |
| `img_033.jpg` | Xe máy nhỏ ở khu vực giữa ảnh cao ~`37.7 px` bị bỏ sót — bẫy `missing` lọt ở vòng duyệt đầu (`CAL-06`). | Bẫy xóa mất box `motorbike` kích thước nhỏ nằm xen giữa đám đông. | Vẽ bổ sung box `motorbike` độc lập. | Bổ sung mục 2.2 (Phân định ngưỡng `20 px`): mọi phương tiện nhìn rõ hình hài và cao `≥ 20 px` phải vẽ box riêng; chỉ dùng `ignore_region` khi `< 20 px` hoặc dính thành đám đông không thể tách. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 3 câu hỏi mở (phân biệt xe đạp điện/xe máy điện, tách hàng nhô ra trên box/mask, không miễn kiểm theo `conf`) và bổ sung 3 quy tắc từ thực tế review/QC (xóa box `three_wheeler` trùng đè của YOLOE, kiểm viền 4 cạnh chống bẫy `loose`, siết ngưỡng `20 px` chống sót xe máy nhỏ). | Từ ca khó `img_018.jpg`, `img_013.jpg` (`CAL-01`, `CAL-02`), lỗi trùng box của `YOLOE-26s` (`CAL-04`) và 2 ca bẫy lọt `img_022.jpg`, `img_033.jpg` (`CAL-03`, `CAL-05`, `CAL-06`). |
