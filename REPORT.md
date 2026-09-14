# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Vương Tuấn Dương<br>
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

Suy luận từ số liệu: `training_run.json.export_sha256` trùng `my_export_audit.archive_sha256` (`43ea38...da9f`) — gói YOLO đã khóa trước khi huấn luyện. `comparison_export_sha256` (`c8bbc7...00d2b`) khác hoàn toàn SHA bài mình — không dùng lại bài mình. `my_native_export_audit.cross_format_consistency.same_annotation_state=true`, 102/102 hộp, min IoU 0.999896 — hai gói của mình cùng trạng thái trước khi đối chiếu.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` mine 4 ↔ ref 2, IoU 0.894486 | tôi: `bus` / đối chiếu: `van` | Suy luận từ IoU 0.89 + overlay: cùng 1 xe bus khớp nối vàng-xanh cỡ lớn, thân dài nhiều cửa sổ | bus = thân khách dài nhiều cửa; van = hộp nhỏ kín, không thân bus. Quyết định `bus` |
| `drive_022` mine 5 ↔ ref 5, IoU 0.831192 | tôi: `truck` / đối chiếu: `bus` | Suy luận: cùng 1 xe trắng nhỏ phía xa, chỉ thấy đầu + thân lửng, thiếu chi tiết thùng/cửa | truck cần thùng/ben/sàn rời rõ; bus cần thân dài nhiều cửa. Thiếu bằng chứng → `unclear` + `needs_review` |
| `drive_033` mine 1 ↔ ref 2, IoU 0.747317 | tôi: `truck` / đối chiếu: `bus` | Suy luận: cùng 1 xe trắng trục dài giữa đường | Kiểm thùng rời vs thân liền khối ở crop 100% trước khi chốt |
| `drive_038` mine 12 ↔ ref 15, IoU 0.886521 | tôi: `van` / đối chiếu: `truck` | Suy luận: cùng 1 xe đỏ, hộp gần trùng nhưng khác lớp | van = hộp kín một khối; truck = khoang hàng tách biệt/ben rõ |
| `drive_038` mine 13 ↔ ref 9, IoU 0.94108 | tôi: `truck` / đối chiếu: `bus` | Suy luận: hộp gần trùng khít (IoU cao nhất nhóm lệch lớp), xe công vụ trắng có thùng sau | truck nếu thùng/thiết bị rõ; bus nếu thân khách dài. Đây là ca ranh giới điển hình |

Mẫu chung từ `comparison_iou.csv` (46 cặp, 12 lệch): `truck→bus: 5`, `van→truck: 4`, `bus→van: 3`. Car khớp 34/34 trong vùng ghép.

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Cặp `drive_038` mine 13 (`truck`) vs ref 9 (`bus`), IoU 0.94108: lớp khác nhau nhưng hình học gần như nhau. Ngược lại, thuộc tính không nằm trong YOLO — `my_export_audit.json` chỉ có `class_names` + `bbox`, không có trường attributes; `my_native_export_audit.json` mới có `required_attributes: [boundary, review_state, visibility]` đủ 102/102 hộp. Cùng lớp `car` vẫn có thể khác `visibility` (`clear`/`occluded`/`unclear`) hoặc `boundary` (`inside`/`truncated`).

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| 102 hộp; `drive_033` 30, `drive_038` 38, `drive_008` 29; vượt mục tiêu 40–60 | phạm vi — suy luận gán thừa vật xa/nhỏ | `within_slide_workload_target=false`; `unmatched_mine=56` (033:20, 038:21, 008:15) vs `unmatched_comparison=4`; overlay 033/038/008 dày hộp đỏ không có hộp xanh tương ứng | Đề xuất: lọc hộp xa/mờ không đủ căn cứ phân lớp, giữ lại hộp đủ bằng chứng, xuất lại cả 2 gói từ cùng trạng thái |
| `drive_008` mine 18 ↔ ref 15, IoU 0.137804, cùng lớp `car` | hình học — suy luận trùng lặp/sai hộp ở cụm xa | IoU thấp nhất 46 cặp, thấp hơn hẳn cặp thứ hai (0.6719) | kiểm trùng, vẽ sát phần nhìn thấy |
| 12/46 lệch lớp, toàn bộ hoán vị truck/bus/van | lớp | `class_agreement=0.73913`; IoU cao vẫn lệch (0.94108, 0.886521) | áp lại định nghĩa truck/bus/van, ca thiếu bằng chứng để `unclear` + `needs_review` |

- Số hộp `needs_review` trước và sau khi kiểm:
  - Không có trong `/output`. Audit chỉ cho tổng `visibility/boundary/review_state: 102` (tức đủ thuộc tính), không bóc theo giá trị. Cần đếm trong `mine-native.zip/annotations.xml`.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ:
  - Suy luận từ số liệu: cụm xe xa ở `drive_033/038/008` (nguồn của 56 `unmatched_mine`) + ca `drive_022` mine 5 (IoU 0.831, `truck`/`bus`). Không phân biệt được thùng rời vs thân liền ở kích thước nhỏ. Xử lý: để `review_state=needs_review`, ghi ảnh + mine_row + IoU + lý do, gửi Lab Coach kèm crop 100% và 2 SHA gói xuất.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`:
  - `row: [0, 0.267875, 0.504703, 0.106969, 0.064781]`
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - `lớp=0 (car) | tâm=(0.2679, 0.5047) | kích thước=(0.1070, 0.0648)`
  - `pixel xyxy: [137.2, 302.3, 205.7, 343.7]` — ảnh 640×640
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

  Kiểm định dạng chỉ kiểm 5 số và miền `[0,1]`. Suy luận từ bài này: cặp IoU 0.94108 qua mọi kiểm định dạng nhưng lệch lớp `truck`/`bus`; 56 hộp không ghép được và ca IoU 0.1378 cùng lớp `car` cho thấy sai phạm vi/hình học vẫn cho dòng hợp lệ.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện:
  - `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định:
  - `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`:
  - Quan sát `detect_result.jpg` (98308 bytes): không thấy hộp/nhãn/confidence rõ như `comparison_overlay.png`. Suy luận: mô hình thử 8 epochs trên 3 ảnh chưa phát hiện được ở `conf=0.25` hoặc ảnh lưu thiếu lớp vẽ. Cần mở ảnh gốc + log `results` để xác nhận.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
  - Suy luận từ số liệu: nếu miss xe xa/nhỏ — kiểm lại quy tắc phạm vi (bài gán 102 vs tham chiếu 50); nếu nhầm truck/bus/van — kiểm lại định nghĩa 3 lớp này (12/46 lệch, toàn bộ hoán vị nhóm này).
- Minh chứng nào có thể bác bỏ nhận định của bạn?
  - `results.csv` / `confusion_matrix` / mAP val, ảnh gốc có hộp + confidence, chạy lại predict trên cả 3 ảnh train và thử ngưỡng `conf` khác. Nếu ảnh gốc có hộp mà thumbnail làm mờ, nhận định “không có hộp” bị bác bỏ.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

  `training_run.json: not_production_benchmark=true`, `purpose` chỉ phản hồi/tìm lỗi dữ liệu. 3 train / 1 val cùng 4 cảnh giao lộ, 8 epochs, `seed 42`.

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
  - 56 (022:0, 033:20, 038:21, 008:15 — suy từ mine − matched)
- Số hộp phía đối chiếu không ghép được:
  - 4
- Một điểm khác biệt cụ thể:
  - `drive_008` mine 18 ↔ ref 15, IoU 0.137804, cùng lớp `car` — thấp nhất, lệch hẳn nhóm còn lại (min còn lại 0.6719). Suy luận: dư/trùng 1 xe xa mà tham chiếu loại.
  - Lệch lớp hệ thống: `truck→bus: 5`, `van→truck: 4`, `bus→van: 3`; ví dụ IoU 0.94108 vẫn lệch `truck`/`bus`.
- Quy tắc hoặc hành động sửa phát sinh:
  - Suy luận từ số liệu: một câu duy nhất — chỉ `truck` khi thấy thùng/ben/sàn tách biệt; chỉ `bus` khi thân dài + dải cửa liên tục; còn lại hộp kín một khối → `van`; thiếu bằng chứng → `unclear` + `needs_review`. Không sửa TXT tay, xuất lại 2 gói, yêu cầu `minimum_cross_format_iou>=0.995`.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

  Suy luận từ số liệu: 12/46 cặp IoU cao vẫn lệch lớp; 56 hộp ngoài vùng ghép không vào mean/median; `comparison_summary.json` tự ghi `floor_is_official_pass_threshold=false` và `interpretation_warning` không phải kết luận sản xuất. Đồng thuận đo tái lập quy tắc, không đo đúng/sai so với sự thật mặt đất.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Suy luận từ số liệu: mạnh nhất là `same_annotation_state=true` (102/102, min 0.999896) + SHA khóa khác SHA tham chiếu + phơi rõ 56 thừa và 12 lệch truck/bus/van trên 46 cặp (mean 0.81386). Câu hỏi: ngưỡng giữ/loại xe xa ở `033/038/008` nên theo kích thước pixel, độ che, hay bắt buộc thấy thùng/cửa — và nên loại hay giữ với `unclear` + `needs_review`?
