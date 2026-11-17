# Báo cáo Day 08 — 2A202603218 - Phạm Trọng Hiện

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1 / 8. Model và recipe (Part 2–3)

| Lớp                                              | Model + cỡ                                             | Ngưỡng            | Vì sao (dẫn `run_id` trong `model_log.csv`)                                                                                                                                                                                                                                                                                                             |
| ------------------------------------------------ | ------------------------------------------------------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| person / bicycle / motorbike / car / bus / truck | YOLO26-m                                               | 0.30 cho cả 6 lớp | `r01` và `r02` nhận được người và xe máy, nhưng tăng ngưỡng làm mất một số box. Recall cao nhất trong các ngưỡng đã xét của person là 16/23 (69,6%), motorbike là 8/9 (88,9%) và car là 7/8 (87,5%), đều dưới mục tiêu 90%, nên giữ mặc định 0.30. Bicycle, bus và truck có lần lượt 4, 1 và 0 vật gold, dưới mức tối thiểu 5 vật để chọn ngưỡng riêng. |
| three_wheeler                                    | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15              | `r03` nhận được xe tuk-tuk. Ngưỡng 0.15 cho recall 5/5 (100%) và precision 5/9 (55,6%). Tăng lên 0.20 làm recall giảm còn 4/5 (80%), nên chọn 0.15 để giữ đủ 5 xe trong gold.                                                                                                                                                                           |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):`python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn SAM 2.1 với trọng số `sam2.1_b.pt`, đã chạy được trên CPU ở `r04`, tạo 16 mask trên 2 ảnh với thời gian 18,32 giây/ảnh (`model_log.csv`). Mask bám theo người đứng bên phải trong `img_004.jpg`. Tốc độ này phù hợp để tiếp tục thử trên một số vật khó ở Part 5.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 gán tuk-tuk thành `car` trong `img_013.jpg` (`r04`). Đổi ngưỡng chỉ giữ hoặc loại box, không tạo ra lớp `three_wheeler` mà detector chưa có. Dùng kết quả YOLOE cho lớp này khi gộp nhãn, sau đó kiểm tra và sửa lớp khi duyệt trong CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Ở `img_011.jpg` của `r01`, người đứng sát mép phải ảnh có box `person` với confidence 0.49. Box này nhận đúng người nhưng bị loại khi tăng ngưỡng lên 0.50 ở `r02`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan. Cùng tập ảnh được dùng để chọn ngưỡng và đánh giá, nên precision/recall có thể cao hơn khi áp dụng cho ảnh mới. Bicycle, bus và truck chỉ có lần lượt 4, 1 và 0 vật gold, nên giữ ngưỡng mặc định. Three_wheeler vừa đủ 5 vật để chọn ngưỡng riêng, nhưng bỏ sót thêm một xe sẽ làm recall giảm 20 điểm phần trăm. Vì vậy, các ngưỡng hiện tại chỉ là ước lượng từ mẫu nhỏ (`qc/threshold_sweep.md`, `qc/threshold_sweep.csv`).

## 2 / 8. AI error profile (từ `qc/review_stats.md`)

Số liệu trên 24 ảnh batch từ `qc/review_stats.md`, đã loại các đối tượng quanh bẫy.

| Trạng thái | Số  | Tỉ lệ           | Mẫu số                  |
| ---------- | --- | --------------- | ----------------------- |
| accepted   | 435 | 435/522 = 83,3% | 522 box máy             |
| edited     | 24  | 24/522 = 4,6%   | 522 box máy             |
| deleted    | 51  | 51/522 = 9,8%   | 522 box máy             |
| added      | 39  | 39/499 = 7,8%   | 499 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler` bị xoá nhiều nhất, 25/43 box (58,1%). Ở `img_013.jpg`, máy tạo box trùng trên tuk-tuk và gán nhầm xe buýt thành three_wheeler.
- So với gold: precision của **máy** = **38/71 = 53,5%**, recall = **38/50 = 76,0%** (50 đối tượng trên 4 ảnh gold). Số đo có thể lạc quan vì chọn và chấm ngưỡng trên cùng tập gold.

## 3 / 8. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **11/12 = 91,7%**, còn lọt 1 bẫy box lỏng (`loose`) ở `img_018.jpg` (`qc/review_stats.md`). Có thể chưa kiểm tra kỹ 4 cạnh box khi duyệt.
- Accepted cao mà bắt bẫy thấp cho thấy có thể tin nhãn máy quá mức. Kết quả của tôi là accepted **446/522 = 85,4%** và bắt bẫy **11/12 = 91,7%**, nhưng bẫy box lỏng còn lọt cho thấy cần soi kỹ vị trí và kích thước box. Mẫu chỉ có 12 bẫy nên kết luận còn hạn chế.

## 4 / 8. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay gold khoảng 2 phút/4 ảnh = 0,5 phút/ảnh; duyệt 12 phút/24 ảnh = 0,5 phút/ảnh. Ảnh ít box khoảng 10 giây, ảnh nhiều box khoảng 1–2 phút. Gán gold trước giúp quen cảnh, có thể làm việc duyệt nhanh hơn.
- Kết luận: Có ích để tạo nhãn sơ bộ, với 446/522 box được giữ (85,4%). Riêng `three_wheeler` cần kiểm tra kỹ vì 25/43 box bị xoá (58,1%). Thời gian trung bình hiện tương đương gán tay, chưa cho thấy tiết kiệm thời gian; hai tập ảnh có độ khó khác nhau và thời gian gold là ước tính.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy và quét toàn ảnh trước khi bật box để tìm vật bị sót.
  2. Kiểm tra lớp và 4 cạnh của từng box, kể cả box confidence cao.
  3. Đối chiếu guideline khi gặp ca khó và đánh dấu `can_review` để xem lại trước khi export.

## 5 / 8. Mask SAM cho 3 vật (Part 5)

| Ảnh         | SAM (phiên bản)         | Số click          | Mask gồm người lái / hàng hoá?          | Lỗi quan sát                         | File trong `masks/`    |
| ----------- | ----------------------- | ----------------- | --------------------------------------- | ------------------------------------ | ---------------------- |
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Còn dính người lái và người phía sau xe | Mask lấn sang người dù đã có điểm âm | `tuktuk_img013.jpg`    |
| img_011.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Đã tách người lái                       | Còn thiếu phần gương xe              | `motorbike_img011.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Chỉ lấy người đứng bên phải             | Chưa thấy lỗi lớn khi xem tổng thể   | `person_img004.jpg`    |

- Đối chiếu guideline mục 4.2: quy tắc bám phần nhìn thấy và tách riêng các vật giúp kiểm tra mask. Cần bổ sung ví dụ tách người lái khỏi xe bằng điểm âm.

## 6 / 8. Calibration (Part 6)

- Số quyết định: **3** (`calibration_decisions.csv`). Quan trọng nhất là **CAL-02**: luôn kiểm tra lớp và vị trí box dù confidence cao, như box sai lớp 0.63 ở `img_013.jpg`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: confidence chỉ hỗ trợ lọc box; kiểm tra đủ 4 cạnh khi duyệt. Nhắc lại quy tắc tuk-tuk thuộc `three_wheeler`.
- Nguyên nhân gốc: máy gán sai lớp, câu hỏi về confidence chưa chốt và bước kiểm box lỏng chưa hiệu quả.

## 7 / 8. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42), **104 đối tượng**.
- Lỗi theo **đối tượng**: **2/104 = 1,9%** · theo **ảnh**: **1/5 = 20%** (ngưỡng Release ≤ 2%), từ `qc/qc_report_auto.md`.
- Top lỗi: `img_003.jpg` có box thừa #14 trùng xe với #13; box #13 thiếu phần mái xe.
- **Giới hạn mẫu nhỏ**: cận trên 95% của tỉ lệ ảnh lỗi là **62%**. Mẫu 5 ảnh chưa đủ chứng minh tỉ lệ lỗi ≤ 2%.

**Quyết định: Rework**, ngày **2026-11-18**. Tỉ lệ ảnh lỗi 20% vượt ngưỡng 2%.
Việc cần làm tiếp: xoá box trùng #14 và chỉnh box #13 ở `img_003.jpg` theo ID trong release, xuất bản sửa `qc_fixed.json`, tạo lại release rồi kiểm QC vòng tiếp theo.

## 8 / 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng ChatGPT/Codex tìm hiểu chỉ số, hỗ trợ viết báo cáo, chạy script và xem ảnh QC/bonus. Số liệu đối chiếu với log và file JSON; nhận xét ảnh do AI hỗ trợ, cần người làm bài xác nhận.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Không
- Dùng cache `data/cache/` thay vì tự chạy? không
- Lỗi / bug script đã gặp (nếu có): Thiếu thư viện clip khi chạy yoloe
- Một điều bạn làm khác lần sau: Sẽ tìm hiểu sâu hơn các chỉ số và định nghĩa

### Bonus — So sánh Grounding DINO, OWLv2 và YOLOE

Dùng lại các lượt đã chạy trên cùng 8 ảnh pilot, thiết bị CPU, chỉ giữ lớp `three_wheeler`. Thời gian lấy từ `run.json` của từng lượt, đã lưu ở `qc/bonus_model_runs.csv`.

| Model | Conf đã chạy | Box/8 ảnh | Giây/ảnh |
|---|---|---|---|
| Grounding DINO | 0.30 | 18 | 31,62 |
| OWLv2 | 0.20 | 44 | 18,79 |
| YOLOE-26s | 0.25 | 8 | 1,21 |

| Ảnh | Box DINO / OWLv2 / YOLOE | Nhận xét khi xem ảnh |
|---|---|---|
| `img_002.jpg` | 2 / 3 / 4 | Cả ba tìm được tuk-tuk xanh chính (1 xe). DINO gán nhầm xe máy bên trái; OWLv2 và YOLOE gán nhầm xe bán tải bên phải. |
| `img_011.jpg` | 2 / 1 / 1 | DINO gán nhầm hai xe máy thành three_wheeler. OWLv2 và YOLOE đặt box vào xe bị cắt ở mép trái, cần kiểm tra loại xe do thiếu phần nhìn thấy. |
| `img_016.jpg` | 4 / 1 / 0 | Không thấy xe ba bánh rõ để đối chiếu. DINO gán nhầm taxi và xe máy; OWLv2 gán nhầm xe máy tiền cảnh. YOLOE không tạo box. |
| `img_032.jpg` | 2 / 13 / 3 | Cả ba tìm được xích lô hoa ở trước và xích lô bên trái. DINO bỏ sót nhóm xe bên phải; YOLOE tìm thêm một xe phải; OWLv2 tạo nhiều box chồng nhau và box trên người. |

- **Quyết định:** giữ YOLOE cho nhãn sơ bộ three_wheeler vì nhanh nhất trong các lượt này và ít box thừa trên `img_016.jpg`. DINO/OWLv2 cung cấp thêm ca đối chiếu nhưng vẫn cần sửa nhãn và box.
- **Giới hạn:** ngưỡng confidence của ba model khác nhau; các điểm số không so trực tiếp được. Thời gian là từng lượt chạy, chưa đo lặp có warm-up. Nhận xét trên 4 ảnh do AI hỗ trợ xem, chưa có nhãn chuẩn pilot độc lập để tính precision/recall.
- **Bằng chứng:** `qc/bonus_image_counts.csv` có số box từng ảnh; ba file dự đoán gốc ở `prelabel/bonus/` để vẽ lại và kiểm chứng.
