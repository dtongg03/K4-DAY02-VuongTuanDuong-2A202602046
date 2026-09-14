# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Vuong Tuan Duong<br>
**MSSV:** 2A202602046<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

> Suy ra hình thức từ minh chứng: `output/comparison_summary.json` ghi `"comparison_source": "teaching_reference"`. Nếu thực tế làm theo cặp, sửa dòng này thành `theo cặp` + mã cặp và đổi nguồn đối chiếu thành `peer`.

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp (`data/day2-cvat-input.zip` / `output/input_pool_audit.json`):
  - `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
  - `provided_input_is_unchanged: true`, `image_count: 4`
- Bốn mã ảnh (640×640, `data/image-manifest.csv` + `output/IMAGE_ATTRIBUTION.md`):
  - `drive_022` — `cb8297af4cc5bb660f9d56f0e31ed77f8fa1f95b31f47bb8b8a1cd08b2f94886` — train / comparison
  - `drive_033` — `8dc05a7a8f06ed137d08b643ef676629a2c4cf3b25051852de22e7400311e465` — train / comparison
  - `drive_038` — `35294a107b157646e0619ad985226ef424968d7798f90f7ded5b32f32cebbb86` — train / comparison
  - `drive_008` — `fde088a7a955343bb3366008d31e3c2a6ecf6ddbfd08e58ecd91f5e6d67a05d1` — val / comparison
  - Nguồn công bố: UA-DETRAC qua bản tái xuất `SQiFeng/traffic-vehicle-detection`, thẻ dữ liệu công bố CC BY 4.0; kho nguồn `VinUni-AI20k/Day2-TrackData-ObjectDetection@710d2c157c5750f71271354d0d31b24457c64e7c` đã đổi tên và không lưu mã bản ghi ảnh gốc — đây là giới hạn truy nguyên, không phải xác minh pháp lý độc lập.
- Số vật thể thực tế (bài của tôi, `output/my_export_audit.json` + `output/my_native_export_audit.json`):
  - **102 hộp / 4 ảnh** — `within_slide_workload_target: false` (mục tiêu slide 40–60, đây không phải ngưỡng đạt).
  - Phân bổ:
    - `drive_022`: 5 (car 3, truck 1, bus 1, van 0)
    - `drive_033`: 30 (car 27, truck 2, bus 1, van 0)
    - `drive_038`: 38 (car 29, truck 2, bus 5, van 2)
    - `drive_008`: 29 (car 22, truck 1, bus 3, van 3)
  - Tổng theo lớp: car 81, truck 6, bus 10, van 5.
  - Kích thước kiểm tra: 640×640 cho cả 4 ảnh, khớp manifest. `coordinate_format: normalized xywh`, `class_names: [car, truck, bus, van]`.
- Mã SHA-256 của gói YOLO của bạn (`mine-yolo.zip`):
  - `43ea3873703c6662f4625403f8054fbf0dd4e37df0dd55800cb762104a72da9f`
- Mã SHA-256 của gói CVAT gốc của bạn (`mine-native.zip`, `CVAT for images XML`):
  - `d113986de842faefccc8fd1b6c85983a29d483d6a0d6df181e9f69a4cbc6de75`
  - `object_count: 102`, khớp 100% tổng số hộp với gói YOLO.
  - Đủ 3 thuộc tính cho 102/102 hộp: `attribute_value_count_by_name: {visibility: 102, boundary: 102, review_state: 102}`, `required_attributes: [boundary, review_state, visibility]`.
  - Nhất quán liên định dạng: `cross_format_consistency.same_annotation_state: true`, `matched_box_count: 102`, `minimum_cross_format_iou: 0.9998959606715293`, `geometry_iou_floor: 0.995`. Chứng tỏ hai gói xuất từ cùng trạng thái công việc, chỉ sai số làm tròn.
- Nguồn đối chiếu: **bộ nhãn đối chiếu do người hướng dẫn thực hành (Lab Coach) cấp — `teaching_reference`**
- Mã SHA-256 của gói đối chiếu (`output/comparison_summary.json.comparison_export_sha256`):
  - `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
  - Tổng suy ra của gói đối chiếu: `matched_boxes (46) + unmatched_comparison (4) = 50` hộp (không có audit JSON riêng cho gói đối chiếu trong `/output`, chỉ có summary + CSV).
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu:
  - Chưa ghi trong `/output` — cần bổ sung thủ công (ví dụ: Lab Coach gửi sau phút 160, mở sau phút 180 theo GUIDE). Bài này giữ nguyên tắc: chỉ tải nguồn đối chiếu ở ô `5b` sau khi đã khóa `my_audit` và huấn luyện thử.

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

1. Trình tự notebook yêu cầu hoàn tất CVAT độc lập, chạy ô `3a/3b/4` (kiểm YOLO + native + huấn luyện) trước ô `5a/5b/5c` (chọn + kiểm + đối chiếu nguồn ngoài).
2. Đã khóa SHA-256 gói YOLO riêng `43ea38...da9f` trong `training_run.json.export_sha256`; ô `5b` có kiểm `if sha256(comparison) == my_audit.archive_sha256: raise` — summary hiện tại cho SHA đối chiếu khác hoàn toàn SHA bài mình, nên không dùng lại bài mình.
3. Hai gói của mình được xác minh `same_annotation_state: true` trước khi đối chiếu, loại trừ việc sửa lệch giữa YOLO và native sau khi khóa.
4. Đối chiếu chỉ dùng IoU hình học để ghép (`matching: ghép tối ưu theo IoU hình học, không dùng lớp khi ghép`), không sửa nhãn gốc để làm đẹp chỉ số.

## 2. Quyết định phân lớp

Tổng theo lớp (bài tôi): car 81 / truck 6 / bus 10 / van 5. Thứ tự lớp cố định `0 car, 1 truck, 2 bus, 3 van`.

| Ảnh/vật thể (dẫn từ `comparison_iou.csv`) | Lớp của tôi | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` mine_row 4 ↔ ref 2, IoU 0.894486 | `bus` (ref: `van`) | thân xe khách dài, nhiều cửa sổ/hàng ghế, xe vàng-xanh khớp bus nối dài | bus = thân xe khách dài, nhiều cửa sổ; van = thân hộp nhỏ kín, không thân bus. Quyết định bus, `review_state=confident` nếu đủ bằng chứng, ngược lại `needs_review`. |
| `drive_022` mine_row 5 ↔ ref 5, IoU 0.831192 | `truck` (ref: `bus`) | xe trắng nhỏ phía xa bên phải, chỉ thấy đầu + thân hộp lửng | truck cần thùng/ben/sàn hàng hoặc thiết bị công vụ rõ ràng; nếu chỉ thấy thân hộp kín một khối → van, nếu thân dài nhiều cửa → bus. Đây là ca thiếu bằng chứng, phải `unclear/needs_review`, không đoán theo kích thước hộp. |
| `drive_033` mine_row 1 ↔ ref 2, IoU 0.747317 | `truck` (ref: `bus`) | xe trắng trục dài giữa đường | Cùng quy tắc truck/bus/van ở trên; cần phóng 100% kiểm thùng rời vs thân liền khối. |
| `drive_033` mine_row 3 ↔ ref 12, IoU 0.865562 | `bus` (ref: `van`) | xe đỏ sọc trắng bên phải | bus vs van: kiểm chiều dài thân + dải cửa sổ liên tục. |
| `drive_038` mine_row 12 ↔ ref 15, IoU 0.886521 | `van` (ref: `truck`) | xe đỏ đầu xe tải nhẹ | van = hộp nhỏ kín một khối; truck = khoang hàng tách biệt/ben rõ. IoU cao nhưng khác lớp → lỗi lớp điển hình, không phải lỗi hình học. |
| `drive_038` mine_row 13 ↔ ref 9, IoU 0.94108 | `truck` (ref: `bus`) | xe trắng xe công vụ có thùng hở phía sau | truck nếu thùng/ben/thiết bị công vụ rõ; bus nếu thân khách dài. IoU 0.94 cho thấy hộp gần trùng khít nhưng lớp vẫn lệch → minh chứng IoU cao ≠ đúng. |
| `drive_008` mine_row 2/3/8 ↔ ref 12/3/5 | `van` (ref: `truck`) | 3/4 ca van→truck trong toàn bộ đối chiếu nằm ở ảnh val | Cần kiểm lại ranh van kín một khối vs xe tải có sàn/thùng tách biệt ở ảnh đông, nhiều che khuất. |

Mẫu nhầm lẫn trong 46 hộp ghép được (12 hộp lệch lớp): `truck→bus: 5`, `van→truck: 4`, `bus→van: 3`. Phía tôi trong vùng ghép: car 34, truck 5, van 4, bus 3; phía đối chiếu: car 34, bus 5, truck 4, van 3. Car khớp tuyệt đối về số lượng trong vùng ghép (34-34) nhưng vẫn có 56 hộp của tôi không ghép được (xem mục 6).

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Ví dụ `drive_038` mine_row 13 (`truck`, IoU 0.94108 với ref `bus`): **lớp** trả lời câu hỏi “đó là loại xe gì” (`truck` vs `bus`), còn **thuộc tính** trả lời “nhìn thấy rõ đến đâu / có bị mép cắt không / có chắc không” (ví dụ `visibility=occluded`, `boundary=inside`, `review_state=needs_review`). Hai hộp có thể cùng lớp `car` nhưng khác `visibility` (rõ vs bị che), hoặc cùng `visibility=clear` nhưng khác lớp (`van` vs `truck`). YOLO chỉ lưu lớp + hộp (`class x_center y_center width height`), không lưu 3 thuộc tính — vì vậy phải xuất thêm `CVAT for images 1.1` và `my_native_export_audit.json` mới kiểm được thuộc tính. Định dạng đúng không thay thế được quyết định ngữ nghĩa.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| 102 hộp, vượt mục tiêu 40–60; `drive_033` 30, `drive_038` 38, `drive_008` 29; 56 hộp không ghép được với tham chiếu (20 ở 033, 21 ở 038, 15 ở 008) | phạm vi / độ đầy đủ (nghi ngờ gán thừa vật thể quá nhỏ/mờ ở xa) | `my_export_audit.json` (`within_slide_workload_target: false`) + `comparison_summary.json` (`unmatched_mine: 56` vs `unmatched_comparison: 4`) + `comparison_overlay.png`: vùng xa ở 033/038/008 dày đặc hộp đỏ (tôi) không có hộp xanh (tham chiếu) tương ứng | Đề xuất sửa trong CVAT, không sửa TXT tay: lọc các hộp xa < ~15–20 px hoặc mờ không đủ căn cứ phân lớp theo phiếu quy tắc “Vật thể quá nhỏ hoặc mờ… không đoán; ghi lý do”; chuyển `review_state=needs_review`, phóng 100%, chỉ giữ lại hộp đủ bằng chứng phân lớp; xuất lại cả hai gói từ cùng trạng thái và chạy lại `verify_same_annotation_state` (yêu cầu min IoU ≥0.995). Ghi lại số trước/sau trong lần xuất tiếp theo. |
| `drive_008` mine_row 18 ↔ ref 15, IoU 0.137804, cùng lớp `car` | hình học (hộp trùng lặp / sai phạm vi, ghép sai do IoU <0.2) | `comparison_iou.csv` sắp xếp IoU tăng dần: đây là IoU thấp nhất toàn bộ 46 cặp; overlay `drive_008` cho thấy cụm hộp nhỏ chồng chéo ở xa | Kiểm trong CVAT có hộp trùng/nhân đôi vật thể xa không; gộp/xóa trùng, vẽ sát phần nhìn thấy, không ước lượng phần che; xuất lại và đối chiếu lại để IoU cụm này tăng hoặc thành unmatched có lý do. |
| 12/46 hộp ghép lệch lớp, tập trung truck/bus/van (ví dụ 038-12 van vs truck IoU 0.8865; 038-13 truck vs bus IoU 0.9410) | lớp | `comparison_iou.csv` + confusion ở mục 2; IoU cao nhưng `class_agree=False` | Áp lại định nghĩa: truck cần thùng/ben/sàn/thiết bị rõ; bus cần thân dài nhiều cửa; van hộp nhỏ kín một khối. Với ca không đủ bằng chứng, đặt `visibility=unclear`, `review_state=needs_review` và ghi lý do vào nhật ký, không đoán theo màu/kích thước hộp. Sửa trong CVAT rồi xuất lại. |

- Số hộp `needs_review` trước và sau khi kiểm:
  - `/output` hiện tại chỉ cho tổng `attribute_value_count_by_name = 102` cho mỗi tên thuộc tính (tức 102/102 hộp có đủ 3 thuộc tính), **không bóc tách theo giá trị** (`confident` vs `needs_review`, `clear` vs `occluded/unclear`, v.v.). Không thể suy ra số `needs_review` từ JSON hiện có. Cần mở `mine-native.zip/annotations.xml` đếm `<attribute name=\"review_state\">needs_review</attribute>` trước/sau để điền con số thật. Đã xác minh không thiếu thuộc tính (thiếu sẽ báo `cần đúng attributes`).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ:
  - Ví dụ `drive_022` mine 5 (`truck` vs ref `bus`, IoU 0.831) và cụm xe xa ở `drive_033/038`: ảnh nhỏ, chỉ thấy một phần đầu xe, không phân biệt được thùng rời vs thân liền khối. Cách xử lý đúng theo GUIDE: giữ hộp, đặt `review_state=needs_review`, ghi ảnh + mã vật thể + dấu hiệu khi phóng 100% + lý do thiếu bằng chứng, gửi Lab Coach kèm crop 100% và hai SHA gói xuất, không đoán, không dùng dự đoán tự động làm đáp án, không đưa dữ liệu cấm.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`:
  - **Hạn chế số liệu:** `output/my_export_audit.json` và `my_native_export_audit.json` chỉ lưu tổng hợp + `class_count`, không lưu từng `bbox_xywh_normalized` / `bbox_xyxy_pixels`. `comparison_iou.csv` chỉ lưu IoU và lớp, không lưu tọa độ. Vì vậy **không thể trích nguyên văn dòng YOLO đầu tiên từ `/output` hiện tại** mà không bịa số liệu. Dòng thật phải lấy từ `mine-yolo.zip` (mở `labels/drive_022.txt` dòng 1) hoặc từ biến `my_records[0]` trong notebook ô `3c`.
  - Cấu trúc chuẩn đã kiểm bởi `audit_yolo_export`: đúng 5 trường, `class_id` nguyên trong `[0,3]`, `x_center,y_center ∈ [0,1]`, `w,h ∈ (0,1]`, hộp không vượt biên (dung sai 1e-6), `names = [car, truck, bus, van]`.
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - Công thức với ảnh 640×640 (áp dụng cho mọi dòng, đã dùng trong `lab_utils._parse_label_file`):
    - `x1 = (x_center − width/2) × 640`, `y1 = (y_center − height/2) × 640`
    - `x2 = (x_center + width/2) × 640`, `y2 = (y_center + height/2) × 640`
    - `bbox_xyxy_pixels = [x1, y1, x2, y2]`, kẹp về `[0,640]`.
  - Ví dụ cách đọc (không phải số liệu bài này, chỉ minh họa công thức): `2 0.5 0.4 0.2 0.3` → lớp `2=bus`, tâm `(320.0, 256.0)` px, rộng `128.0` px, cao `192.0` px → `xyxy = [256.0, 160.0, 384.0, 352.0]`. Để điền báo cáo thật, chạy ô `3c` và chép nguyên `row: [...]`, `lớp=...`, `pixel xyxy: [...]` vào đây.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?
  - Kiểm định dạng chỉ kiểm 5 số, miền giá trị và biên ảnh — không kiểm ngữ nghĩa. Minh chứng trong bài: `drive_038` mine 13 vs ref 9 IoU 0.94108 (hộp gần trùng khít, chắc chắn qua mọi kiểm định dạng) nhưng lệch lớp `truck` vs `bus`; `drive_038` mine 12 vs ref 15 IoU 0.886521 lệch `van` vs `truck`. Sai phạm vi (gộp nhiều xe, gán người/xe máy, gán vật quá nhỏ không căn cứ) và sai hình học (hộp lỏng, chứa nhiều nền, đoán phần che, trùng lặp như ca IoU 0.1378 ở `drive_008-18`) đều vẫn cho dòng 5 số hợp lệ. Vì vậy “qua audit YOLO” ≠ “nhãn đúng”.

## 5. Huấn luyện và dự đoán thử

Theo `output/training_run.json` (khớp `requirements.txt` và notebook ô 4):

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Cấu hình: `ultralytics 8.4.145`, `yolo11n.pt` (`sha256 0ebbc80d4a7680d14987a577cd21342b65ecfd94632bd9a8da63ae6417644ee1`), `epochs 8`, `seed 42`, `device 0` (GPU), `imgsz 640`, `export_sha256 43ea3873703c6662f4625403f8054fbf0dd4e37df0dd55800cb762104a72da9f` (đúng SHA gói YOLO của tôi), `elapsed_seconds 38.09`, `purpose: chỉ dùng để phản hồi và tìm lỗi dữ liệu`, `not_production_benchmark: true`.
- Mô tả một dự đoán trong `detect_result.jpg`:
  - Ảnh `detect_result.jpg` (98308 bytes) khi mở ở độ phân giải hiện tại **không thể hiện hộp dự đoán rõ ràng** (không thấy overlay nhãn/confidence như ảnh `comparison_overlay.png`). Khả năng: (a) mô hình thử nghiệm 8 epochs trên 3 ảnh chưa phát hiện được vật thể ở ngưỡng `conf=0.25`, (b) ảnh lưu thiếu lớp vẽ, hoặc (c) hộp quá nhỏ/mờ không quan sát được ở thumbnail. Không bịa mô tả “phát hiện N xe, confidence X” khi ảnh không cho thấy. Cần mở ảnh gốc 640×640 và log `results` trong Colab để xác nhận số hộp/conf trước khi kết luận.
  - Nếu lấy overlay đối chiếu làm tham khảo định tính cho ảnh `drive_008` (không phải output huấn luyện): cảnh giao lộ đông, nhiều xe con ở gần + xe tải/bus/van ở giữa, cụm xe nhỏ ở xa dễ miss — đúng vùng mô hình nhỏ thường bỏ sót khi chỉ huấn luyện 3 ảnh.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
  - Nếu mô hình miss nhiều xe xa/nhỏ: gợi ý rà lại quy tắc phạm vi “vật thể quá nhỏ/mờ không đủ căn cứ thì không đoán” — bài tôi đang gán tới 102 hộp (nhiều hơn tham chiếu 50), có thể đang giữ cả vùng mà mô hình 8-epoch không học được; cần thống nhất ngưỡng giữ/loại và ghi `unclear/needs_review` nhất quán.
  - Nếu mô hình nhầm truck/bus/van: gợi ý kiểm lại định nghĩa 3 lớp này (đúng cụm nhầm lẫn lớn nhất ở mục 2: 12/46 lệch, toàn bộ là hoán vị truck/bus/van).
- Minh chứng nào có thể bác bỏ nhận định của bạn?
  - Log huấn luyện đầy đủ (`results.csv`, `confusion_matrix`, `val mAP`), file `best.pt` + ngưỡng `conf` khác, ảnh `detect_result.jpg` gốc có hộp + nhãn confidence, và việc chạy lại predict trên cả 3 ảnh train để xem miss có hệ thống hay chỉ ở ảnh val. Nếu ảnh gốc thực ra có hộp mà thumbnail làm mờ, nhận định “không có hộp” sẽ bị bác bỏ — cần kiểm lại file gốc.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?
  - Chỉ 3 ảnh train / 1 ảnh val, cùng 4 cảnh giao thông liên quan thời gian, không tập test độc lập, không đa dạng bối cảnh/thời tiết/góc máy; 8 epochs, `freeze=10`, `patience=3` chỉ để kiểm đường ống dữ liệu. `training_run.json` đã ghi rõ `purpose` chẩn đoán và `not_production_benchmark: true`. mAP/ảnh dự đoán trên 4 ảnh không chấm chất lượng người gán nhãn và không chứng minh khả năng triển khai.

## 6. Đối chiếu nhãn

Nguồn: `output/comparison_summary.json` + `output/comparison_iou.csv` + `output/comparison_overlay.png`. Phương pháp: `ghép tối ưu theo IoU hình học, không dùng lớp khi ghép` (Hungarian, `linear_sum_assignment` trên ma trận IoU), `comparison_iou_floor: 0.01` — đây là tham số kỹ thuật để loại ghép nhiễu, **không phải ngưỡng đạt** (`floor_is_official_pass_threshold: false`).

- Số hộp ghép được: **46** (`matched_boxes`)
- IoU trung bình và trung vị: **mean 0.81386, median 0.838446**
- Mức đồng thuận lớp: **0.73913 = 34/46** (34 cặp cùng lớp, 12 cặp lệch)
- Số hộp phía bạn không ghép được: **56** (`unmatched_mine`)
- Số hộp phía đối chiếu không ghép được: **4** (`unmatched_comparison`)
- Chi tiết theo ảnh (từ CSV, `unmatched_mine = mine − matched`):
  - `drive_022`: mine 5, matched 5 (mean 0.8085, min 0.7374, max 0.8945, agree 3/5), unmatched_mine 0 — ảnh duy nhất khớp phủ hoàn toàn.
  - `drive_033`: mine 30, matched 10 (mean 0.7876, min 0.6719, max 0.8656, agree 8/10), unmatched_mine 20.
  - `drive_038`: mine 38, matched 17 (mean 0.8432, min 0.7060, max 0.9411, agree 14/17), unmatched_mine 21 — IoU trung bình cao nhất nhưng vẫn lệch lớp ở các hộp IoU cao.
  - `drive_008`: mine 29, matched 14 (mean 0.7989, min 0.1378, max 0.9604, agree 9/14), unmatched_mine 15 — chứa IoU thấp nhất toàn bộ.
  - Tổng kiểm: 0+20+21+15 = 56, khớp summary; mine 102 vs tham chiếu suy ra 50.
- Một điểm khác biệt cụ thể:
  - Chọn ca `drive_008` mine_row 18 ↔ comparison_row 15: cùng lớp `car` nhưng **IoU 0.137804** (thấp nhất, thấp hơn hẳn cặp thấp thứ hai 0.6719). Overlay `drive_008` cho thấy cụm xe xa ở phía trên ảnh với nhiều hộp đỏ nhỏ chồng chéo — khả năng tôi gán dư/trùng một xe xa mà tham chiếu gộp hoặc loại vì quá nhỏ/mờ. Ngược lại, ca `drive_038` mine 13 ↔ ref 9 IoU 0.94108 nhưng lệch `truck` vs `bus` cho thấy lỗi lớp độc lập với hình học.
  - Hoán vị lớp hệ thống: `truck→bus (5)`, `van→truck (4)`, `bus→van (3)` — không phải lỗi ngẫu nhiên mà là ranh giới truck/bus/van chưa nhất quán.
- Quy tắc hoặc hành động sửa phát sinh:
  - Không sửa TXT tay. Trong CVAT: (1) xóa/gộp hộp trùng ở cụm xa `drive_008` phía trên, vẽ sát phần nhìn thấy; (2) áp lại một câu quy tắc duy nhất cho truck/bus/van: “chỉ gán `truck` khi thấy thùng/ben/sàn tách biệt hoặc thiết bị công vụ rõ; chỉ gán `bus` khi thấy thân dài + dải cửa sổ/hàng ghế liên tục; còn lại thân hộp nhỏ kín một khối → `van`; thiếu bằng chứng → `visibility=unclear`, `review_state=needs_review`”; (3) xuất lại cả hai gói từ cùng trạng thái, yêu cầu `minimum_cross_format_iou ≥0.995` và đối chiếu lại để `unmatched_mine` giảm có lý do, không chạy theo IoU trung bình.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?
  - Đồng thuận đo khả năng tái lập quy tắc giữa hai nguồn độc lập, không phải đối chiếu với sự thật mặt đất. Minh chứng: 12/46 cặp ghép IoU cao vẫn lệch lớp (ví dụ IoU 0.94 vẫn sai lớp); 56 hộp của tôi không ghép được có thể cả hai bên cùng thiếu hoặc cùng thừa mà IoU không phản ánh; tham chiếu tự ghi `interpretation_warning: Bộ nhãn đối chiếu hỗ trợ phản hồi sau bài làm độc lập; đây không phải kết luận chất lượng sản xuất.` Dùng IoU trung bình để xếp hạng ai đúng hoặc dùng mAP 4 ảnh để chấm người gán là sai phương pháp theo RUBRIC.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ (`GUIDELINE_MINI_SHEET.md` — cần điền từ `guideline-mini-sheet.md`, không để `CHƯA ĐIỀN`).
- [x] Có kết quả kiểm hai gói xuất (`input_pool_audit.json`, `my_export_audit.json`, `my_native_export_audit.json` + `IMAGE_ATTRIBUTION.md` trong `day2_lab_outputs/`).
- [x] Có thông tin lần huấn luyện và ảnh dự đoán (`training_run.json`, `detect_result.jpg`).
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu (`comparison_summary.json`, `comparison_iou.csv`, `comparison_overlay.png`).
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình (chỉ đưa 9 file output + `REPORT.md` + `GUIDELINE_MINI_SHEET.md`; ZIP chuyển tệp từ Colab chỉ để tải về, không phải bài nộp; nộp đường dẫn kho trên VLearn).
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là tính nhất quán liên định dạng tuyệt đối (`same_annotation_state: true`, 102/102 hộp, min IoU 0.999896) kết hợp đối chiếu độc lập trung thực (SHA khóa khác SHA tham chiếu, mean IoU 0.81386/median 0.8384/agreement 73.9% trên 46 cặp, phơi rõ 56 hộp thừa và cụm nhầm truck/bus/van) — cho thấy đường ống dữ liệu đúng nhưng quy tắc phạm vi + ranh 3 lớp xe lớn còn lỏng, đặc biệt bài này gán gấp đôi tham chiếu (102 vs 50) và vượt mục tiêu 40–60. Câu hỏi cho Lab Coach: với cụm xe xa <20 px ở `drive_033/038/008` (nguồn chính của 56 unmatched), ngưỡng “đủ bằng chứng phân lớp” nên là kích thước tối thiểu, mức che khuất, hay yêu cầu thấy chi tiết thùng/cửa sổ — và có nên loại hàng loạt các hộp này trong lần xuất lại hay giữ với `unclear/needs_review` để phản ánh đúng thực tế hiện trường?
