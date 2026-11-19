# Báo cáo Day 08 — Nguyễn Thị Ngọc Mai

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | `YOLO26m` (`yolo26m.pt`) | `car = 0.5`; các lớp còn lại dùng mặc định `0.3` | Từ `r01`, `r02`, `r05` (`model_log.csv`) và `submission/qc/threshold_sweep.md`: `YOLO26m` nhận diện tốt các lớp giao thông phổ biến. Quét ngưỡng trên 4 ảnh gold cho thấy `car = 0.5` đạt recall `90%` (9/10 vật, precision `69%`); `person` (tối đa recall `74%` trên 42 vật), `bicycle` (tối đa `60%` trên 10 vật) và `motorbike` (tối đa `76%` trên 21 vật) không ngưỡng nào `<= 0.5` đạt recall `>= 90%` nên giữ mặc định `0.3`; `bus` (0 vật) và `truck` (1 vật) có `< 5` mẫu trên gold nên giữ ngưỡng mặc định `0.3` để tránh overfit. |
| three_wheeler | `YOLOE-26s` (`yoloe-26s-seg.pt`), prompt: `three_wheeler` | `0.15` | `YOLO26m` là detector danh mục đóng 80 lớp COCO nên không có lớp `three_wheeler`, trong khi `YOLOE-26s` (open-vocabulary) tìm được tuk-tuk. Trên 4 ảnh gold (7 vật), ngưỡng `0.15` đạt recall `100%` (7/7 vật) và precision `78%`. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn SAM 2.1 (`sam2.1_b.pt`) làm mô hình chính chạy trên Google Colab GPU T4 vì máy cá nhân chạy Windows không có GPU rời. Chọn `SAM 2.1` cho chất lượng đường biên mask bám sát các chi tiết khung kim loại, bánh xe và khoảng hở của xe tuk-tuk tốt hơn `MobileSAM`, không bị vướng rào cản quyền truy cập như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi lệch hệ quy chiếu danh mục (ontology mismatch): `YOLO26m` không có khái niệm `three_wheeler` nên luôn gán nhầm xe tuk-tuk thành `car` hoặc `truck` ở mọi mức `--conf` (`r01`, `r02`). Tăng hay giảm ngưỡng chỉ đánh đổi giữa độ phủ và nhiễu chứ không tạo ra lớp mới. Cách khắc phục là kết hợp mô hình open-vocabulary `YOLOE-26s` với prompt chữ `three_wheeler` và đặt đứng trước `YOLO26m` trong `merge_prelabels.py` (tại `IoU >= 0.6`) để ưu tiên nhãn `three_wheeler` thắng nhãn `car/truck` trùng vị trí.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: Trên img_013.jpg, YOLO26m gán nhầm chiếc xe tuk-tuk thành car với độ tin cậy `0.42.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Vì bộ ngưỡng `--thr "car=0.5,three_wheeler=0.15"` được tìm và đánh giá in-sample trên cùng 4 ảnh gold, các chỉ số precision/recall trên gold mang tính lạc quan và có khoảng tin cậy 95% rất rộng. Ngoài ra, 4 ảnh gold không có mẫu `bus` (0/4) và chỉ có 1 `truck` (1/4), 7 `three_wheeler`, 10 `car` nên ngưỡng ước lượng chỉ là xấp xỉ thô, không đại diện hết toàn bộ 24 ảnh batch.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 429 | 85.0% | 505 box máy |
| edited | 26 | 5.1% | 505 box máy |
| deleted | 50 | 9.9% | 505 box máy |
| added | 80 | 15.0% | 535 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler là lớp máy hoạt động kém nhất. Nguyên nhân: hạ ngưỡng three_wheeler xuống 0.15 giúp YOLOE-26s không bỏ sót tuk-tuk nhưng làm phát sinh nhiều box chồng đè lên cùng một xe hoặc nhận nhầm xe đẩy hàng. Như trên `img_013.jpg và img_002.jpg
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12; loại nào lọt: 2/12 bẫy, gồm img_022.jpg (loose) và img_033.jpg(missing); vì sao (bạn đã bỏ bước nào của checklist)? Bỏ qua Bước 1 (ẩn nhãn/quét toàn ảnh bằng mắt trước) và bước 4 (soi kỹ 4 cạnh viền từng box).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ `accepted` cao đi kèm tỉ lệ bắt bẫy vòng đầu thấp chứng tỏ người duyệt đang phụ thuộc thụ động vào gợi ý của máy — có xu hướng bấm giữ nguyên hầu hết các khung bao trông có vẻ hợp lý thay vì chủ động kiểm tra độc lập từng đối tượng và từng cạnh viền.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 8 phút/ảnh vs duyệt 4 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): Việc gán tay 4 ảnh gold trước giúp tôi thuộc sẵn bảng nhãn và quy tắc `ignore_region`, tạo ra một phần hiệu ứng quen tay khi sang duyệt 24 ảnh batch; tuy nhiên 24 ảnh batch có mật độ xe dày đặc và cảnh đêm phức tạp hơn nên tốc độ nhanh gấp đôi (`4` phút/ảnh so với `8` phút/ảnh) chủ yếu đến từ việc máy đã vẽ sẵn `85.0%` (`429/505`) khung bao chuẩn.
- Kết luận: **Có đáng dùng, nhưng phải áp dụng ngưỡng riêng theo lớp và bắt buộc duyệt kỹ**, dựa trên số liệu: pre-label giúp **giảm 50% thời gian thao tác** (`4` phút/ảnh vs `8` phút/ảnh) và giữ nguyên được `85.0%` (`429/505`) box máy vẽ sẵn. Tuy nhiên, nếu dùng thô không duyệt sẽ gặp `9.9%` (`50/505`) box thừa/sai lớp (đặc biệt ở `three_wheeler`) và bỏ sót `12.9%` (`69/535`) đối tượng thật.
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét mắt toàn ảnh trước khi nhìn box máy: Trên mỗi ảnh CVAT, thực hiện đếm và xác định vị trí các cụm người/xe từ trái sang phải trong 15–20 giây đầu tiên trước khi chỉnh sửa box có sẵn (kiểm tra bằng việc phát hiện được các ca sót `added = 80/535`).
  2. Kiểm tra đủ 4 cạnh viền cho mọi box mục tiêu: Bắt buộc rà lần lượt cạnh trên (đỉnh đầu/mái xe), cạnh dưới (bánh xe/bàn chân) và hai cạnh bên của từng box trước khi chuyển ảnh, không chấp nhận box hở nền rộng (kiểm tra bằng số lượng box được chỉnh sửa `edited = 26/505` và bắt được `2/3` bẫy `loose`).
  3. Rà lọc theo từng lớp bằng thanh công cụ Filters và chia chặng nghỉ: Sau khi duyệt xong một ảnh nhiều đối tượng, dùng bộ lọc Filters trên CVAT kiểm tra riêng từng lớp dễ nhầm (`three_wheeler`, `bicycle`, `motorbike`) và nghỉ mắt 3–5 phút sau mỗi 8–10 ảnh để tránh mệt mỏi thị giác (kiểm tra bằng việc bắt được `3/3` bẫy `wrong_class` và `3/3` bẫy `extra`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `SAM 2.1 (sam2.1_b.pt)` | 3 click (`2` dương + `1` âm) | Không (đã tách riêng người lái bằng click âm `--neg` theo mục 4.2) | Khoét tách được phần lưng áo người lái khỏi thân xe tuk-tuk, viền mái che và bánh xe bám sát; còn dính một mảnh nhỏ ở vùng tay cầm sát người lái. | `[tuktuk_img013.jpg]` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc tại mục 4.2 quy định rõ mask của phương tiện chỉ ôm khung xe, mái che và bánh xe; người lái (`person`) và hàng hóa nhô ngoài phải được tách riêng"* giúp xác định chính xác vị trí đặt điểm âm (`--neg`). Tuy nhiên, guideline v1.0 còn thiếu quy định rõ đối với kính chắn gió / cửa sổ xe và khoảng hở khung sắt của xe tuk-tuk (nên bao gồm kính liền khối trong mask xe nhưng khoét rỗng khoảng hở khung sắt nhìn xuyên nền) — nội dung này đã được bổ sung vào `guideline_v1.1_calibrated.md`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: 6 quyết định (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): CAL-01
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: 
  - Chốt 3 câu hỏi mở từ mục 6 của v1.0: phân định `bicycle` và `motorbike` dựa theo bàn đạp/sàn để chân và ngưỡng `20 px` (`CAL-01`, `img_018.jpg`); không kéo giãn box theo hàng nhô ra và loại trừ người lái/hàng hóa trên mask SAM (`CAL-02`, `img_013.jpg`); quy định ngưỡng tách box độc lập cho khối xe máy đỗ dày gần camera khi không bị che khuất `> 50%` (`CAL-03`, `img_005.jpg`).
  - Bổ sung 3 quy tắc thực hành mới: chống trùng lặp box cùng lớp có `IoU > 0.6` (`CAL-04`, `img_013.jpg`), bắt buộc kiểm viền 4 cạnh cho mọi box `person` (`CAL-05`, `img_022.jpg`), và siết ngưỡng chiều cao `>= 20 px` cho xe máy nhỏ (`CAL-06`, `img_033.jpg`).
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>
  - Máy: `YOLOE-26s` sinh box trùng đè ở ngưỡng `0.15`
  - Thao tác người duyệt: bị automation bias khiến bỏ lọt bẫy viền lỏng.

## 7. QC và quyết định (Part 6)

- Mẫu: 7 ảnh ( 2 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 1/120 = 0.8%· theo **ảnh**: 1/7 = 14.3% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: sai_vi_tri_kich_thuoc: Trên ảnh img_022.jpg, box id 271.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Mẫu QC chỉ gồm `7/24` ảnh batch nên độ bất định thống kê rất lớn. Với `1/7 = 14.3%` ảnh có lỗi, khoảng tin cậy 95% (Wilson) có cận trên lên tới khoảng 51.0% (ngay cả khi `0/7` ảnh lỗi thì cận trên Wilson 95% vẫn là `35%`). Do đó, kết quả này chỉ đủ để phát hiện lỗi hiện hữu chứ chưa đủ chứng minh toàn lô 24 ảnh đã sạch dưới ngưỡng `2%`.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script không tự phát hiện được box nào bị hở viền (`loose`) nếu chiều cao vẫn `>= 20 px`, không biết người duyệt có thực sự ẩn box máy để quét ảnh trước theo checklist hay không, và không phát hiện được các đối tượng mà cả máy lẫn người duyệt đều cùng bỏ sót

**Quyết định: Rework**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release): 
- *Kiểm Hold:* Không có câu hỏi guideline nào chưa chốt (`guideline_gap = 0`, đã cập nhật v1.1) và không có ảnh gốc hỏng (`data_problem = 0`) → **Không Hold**.
- Kiểm Rework: Tỉ lệ bắt bẫy đạt `10/12 = 83.3% >= 80%`, nhưng tỉ lệ ảnh có lỗi trên mẫu QC là `1/7 = 14.3%`, vượt quá ngưỡng cho phép để Release (`<= 2%`) do còn tồn tại lỗi `sai_vi_tri_kich_thuoc` ở box `#271` trên ảnh `img_022.jpg` → Chốt `Rework`.
Nếu Rework / Hold, việc cần làm tiếp: Mở lại Task `batch-review` trên CVAT, chỉnh sửa lại 4 cạnh viền của box `#271 (person)` trên ảnh `img_022.jpg` cho ôm sát người (đồng thời rà lại các box nhỏ `< 20 px` trên `img_018.jpg`, `img_025.jpg`, `img_027.jpg`, `img_033.jpg` để chuyển sang `ignore_region` nếu cần), export lại `submission/labels/batch_reviewed.json`, chạy lại `build_release.py` và kiểm QC xác nhận `0` lỗi trước khi Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng Gemini, ChatGPT để hỗ trợ kiểm tra tiến độ từng checkpoint, giải thích lỗi cú pháp PowerShell.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không, bốn ảnh gold được tôi gán thủ công hoàn toàn bằng tay trên CVAT dựa trên `guideline_v1.0_label_contract.md` và export xong `submission/labels/gold_manual.json` trước khi chạy bất kỳ mô hình AI hay xem dữ liệu pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Window, không có GPU, CVAT bản Docker v2.74.1
- Dùng cache `data/cache/` thay vì tự chạy? Không.Tôi tự chạy toàn bộ các bước inference `YOLO26m`, `YOLOE-26s` và `SAM 2.1` trên Google Colab GPU T4 và tải kết quả thực tế về thư mục `work/prelabel/` và `submission/masks/` của mình.
- Lỗi / bug script đã gặp (nếu có): Không
- Một điều bạn làm khác lần sau: Ngay từ vòng duyệt đầu tiên trên CVAT, tôi sẽ tuân thủ nghiêm ngặt quy tắc ẩn toàn bộ nhãn gợi ý của máy để quét ảnh bằng mắt trong 15–20 giây đầu và rà đủ 4 cạnh viền từng box.
