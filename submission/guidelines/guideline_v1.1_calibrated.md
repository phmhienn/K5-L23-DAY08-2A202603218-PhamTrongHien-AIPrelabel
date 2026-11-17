# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603218 - Phạm Trọng Hiện

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 7 | Confidence cao có đủ để chấp nhận box máy? | Kiểm tra lớp và vị trí box trước khi chấp nhận. Confidence chỉ hỗ trợ lọc box. | img_013.jpg, box three_wheeler 0.63 gán nhầm xe buýt | CAL-02 |

Các câu hỏi 1–6 chưa chốt trong lần calibration này.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg (CAL-01) | Tuk-tuk bị gán sai lớp | YOLO26 gán car | Sửa thành three_wheeler và kiểm tra riêng người lái | Áp dụng lại định nghĩa lớp ở mục 2 v1.0 |
| img_018.jpg (CAL-03) | Bẫy box lỏng còn lọt | Box đã nới rộng chưa được sửa đạt yêu cầu | Kiểm tra 4 cạnh và chỉnh sát phần nhìn thấy | Bổ sung bước kiểm 4 cạnh vào quy trình duyệt theo mục 4.1 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi về confidence, nhắc lại cách phân lớp tuk-tuk và bổ sung bước kiểm 4 cạnh box | CAL-01, CAL-02, CAL-03 |
