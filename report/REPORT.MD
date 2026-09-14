# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Vuong Tuan Duong<br>
**MSSV:** 2A202602046<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp:
  - `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh:
  - `drive_022` — `cb8297af4cc5bb660f9d56f0e31ed77f8fa1f95b31f47bb8b8a1cd08b2f94886` — 640×640 — train
  - `drive_033` — `8dc05a7a8f06ed137d08b643ef676629a2c4cf3b25051852de22e7400311e465` — 640×640 — train
  - `drive_038` — `35294a107b157646e0619ad985226ef424968d7798f90f7ded5b32f32cebbb86` — 640×640 — train
  - `drive_008` — `fde088a7a955343bb3366008d31e3c2a6ecf6ddbfd08e58ecd91f5e6d67a05d1` — 640×640 — val
- Số vật thể thực tế:
  - 102 / 4 ảnh. `within_slide_workload_target: false`
  - `drive_022`: 5 (car 3, truck 1, bus 1, van 0)
  - `drive_033`: 30 (car 27, truck 2, bus 1, van 0)
  - `drive_038`: 38 (car 29, truck 2, bus 5, van 2)
  - `drive_008`: 29 (car 22, truck 1, bus 3, van 3)
- Mã SHA-256 của gói YOLO của bạn:
  - `43ea3873703c6662f4625403f8054fbf0dd4e37df0dd55800cb762104a72da9f`
- Mã SHA-256 của gói CVAT gốc của bạn:
  - `d113986de842faefccc8fd1b6c85983a29d483d6a0d6df181e9f69a4cbc6de75`
- Nguồn đối chiếu: bạn cùng cặp hoặc bộ tham chiếu do người hướng dẫn thực hành cấp:
  - `teaching_reference`
- Mã SHA-256 của gói đối chiếu:
  - `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu:


Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:


## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` mine 4 ↔ ref 2, IoU 0.894486 | tôi: `bus` / đối chiếu: `van` |  |  |
| `drive_022` mine 5 ↔ ref 5, IoU 0.831192 | tôi: `truck` / đối chiếu: `bus` |  |  |
| `drive_033` mine 1 ↔ ref 2, IoU 0.747317 | tôi: `truck` / đối chiếu: `bus` |  |  |
| `drive_038` mine 12 ↔ ref 15, IoU 0.886521 | tôi: `van` / đối chiếu: `truck` |  |  |
| `drive_038` mine 13 ↔ ref 9, IoU 0.94108 | tôi: `truck` / đối chiếu: `bus` |  |  |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:


## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| 102 hộp; `drive_033` 30, `drive_038` 38, `drive_008` 29 |  | `my_export_audit.json: within_slide_workload_target=false`; `comparison_summary.json: unmatched_mine=56, unmatched_comparison=4` |  |
| `drive_008` mine 18 ↔ ref 15, IoU 0.137804 |  | `comparison_iou.csv`: IoU thấp nhất trong 46 cặp |  |

- Số hộp `needs_review` trước và sau khi kiểm:
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ:

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`:
  - `row: [0, 0.267875, 0.504703, 0.106969, 0.064781]`
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - `lớp=0 (car) | tâm=(0.2679, 0.5047) | kích thước=(0.1070, 0.0648)`
  - `pixel xyxy: [137.2, 302.3, 205.7, 343.7]` — ảnh 640×640
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

  Kiểm định dạng chỉ kiểm miền giá trị, không kiểm nội dung. Bài này: IoU 0.94108 vẫn lệch lớp `truck`/`bus`.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện:
  - `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định:
  - `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`:
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
- Minh chứng nào có thể bác bỏ nhận định của bạn?
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

  `training_run.json: not_production_benchmark=true`. 3 train / 1 val, 8 epochs.

- Cấu hình (để đối chiếu):
  - `ultralytics 8.4.145`, `yolo11n.pt`, `sha256 0ebbc80d4a7680d14987a577cd21342b65ecfd94632bd9a8da63ae6417644ee1`, `epochs 8`, `seed 42`, `device 0`, `elapsed_seconds 38.09`

## 6. Đối chiếu nhãn

- Số hộp ghép được:
  - 46
- IoU trung bình và trung vị:
  - `mean 0.81386`, `median 0.838446`
- Mức đồng thuận lớp:
  - `0.73913` (34/46)
- Số hộp phía bạn không ghép được:
  - 56
- Số hộp phía đối chiếu không ghép được:
  - 4
- Một điểm khác biệt cụ thể:
  - `drive_008` mine 18 ↔ ref 15, IoU 0.137804, cùng lớp `car`
  - Lệch lớp: `truck→bus: 5`, `van→truck: 4`, `bus→van: 3`
- Quy tắc hoặc hành động sửa phát sinh:
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?


## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

