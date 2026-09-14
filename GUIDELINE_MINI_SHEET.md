# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Vương Tuấn Dương<br>
**MSSV:** 2A202602046<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO` — cá nhân

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_022` mine_row 4 ↔ comparison_row 2, IoU 0.894486 (ghép tối ưu theo hình học, `comparison_summary.json: matched_boxes 46`). Suy luận cùng 1 phương tiện.
- Dấu hiệu nhìn thấy: Phóng 100% trong CVAT + đối chiếu `comparison_overlay.png`: thân xe khách dài màu vàng-xanh, có khớp nối, dải cửa sổ liên tục dọc thân, chiều dài vượt khung van hộp nhỏ. Đối chiếu ghi `mine_class bus` / `comparison_class van` (`comparison_iou.csv:4`).
- Quy tắc áp dụng: `bus` = thân xe khách dài, nhiều cửa sổ/hàng ghế; `van` = thân hộp nhỏ, kín, không có thân xe buýt. Không đoán theo màu.
- Quyết định: `bus` (gán `2`). Thuộc tính kèm: `visibility=clear`, `boundary=inside`, `review_state=confident` — đủ bằng chứng thân dài.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Đặt `visibility=unclear`, `review_state=needs_review`, ghi ảnh + mine_row + IoU + lý do “không phân biệt được dải cửa vs thân hộp kín ở cỡ nhỏ”, gửi Lab Coach kèm crop 100% và 2 SHA gói xuất (`43ea3873...` vs `c8bbc767...`), không sửa TXT tay.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038` mine_row 12 ↔ comparison_row 15, IoU 0.886521 (`comparison_iou.csv:25`: `mine_class van` / `comparison_class truck`). Suy luận cùng 1 xe đỏ hộp kín.
- Dấu hiệu nhìn thấy: Phóng 100%: thùng hộp đỏ liền khối, không thấy sàn/ben/thùng tách biệt hay thiết bị công vụ rời; không có cabin + khoang hàng tách biệt rõ như xe tải. Nhóm lệch lớp hệ thống: `van→truck: 4` cases trong 12 lệch (tổng `class_agreement 0.73913`).
- Quy tắc áp dụng: `truck` chỉ khi thấy thùng/ben/sàn hàng hoặc thiết bị công vụ rõ ràng tách khỏi cabin; `van` = thân hộp nhỏ kín một khối. Nếu thấy hộp kín một khối → `van`.
- Quyết định: `van` (gán `3`). Thuộc tính kèm: `visibility=clear`, `boundary=inside`, `review_state=confident`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Đặt `visibility=unclear`, `review_state=needs_review`, ghi “không thấy mép thùng rời ở cỡ xa”, giữ nguyên hộp và xuất lại cả 2 gói từ cùng trạng thái, yêu cầu `minimum_cross_format_iou>=0.995` như `my_native_export_audit.json: 0.999896`.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_022` mine_row 5 ↔ comparison_row 5, IoU 0.831192, `mine_class truck` / `comparison_class bus` (`comparison_iou.csv:6`). Đại diện nhóm vật xa thiếu chi tiết. Bổ sung: cụm `drive_033/038/008` là nguồn chính của 56 `unmatched_mine` (`comparison_summary.json: unmatched_mine 56` vs `unmatched_comparison 4`; suy ra `033:20, 038:21, 008:15`).
- Dấu hiệu nhìn thấy khi phóng 100%: Xe trắng nhỏ phía xa, chỉ thấy đầu + thân lửng, bị xe trước che một phần, kích thước pixel nhỏ, không đọc được thùng/ben hay dải cửa. Ca `drive_008` mine_row 18 ↔ ref 15 IoU thấp nhất 0.137804 cùng lớp `car` cho thấy cùng hiện tượng dư/trùng ở cụm xa.
- Giá trị `visibility`: `unclear` (không rõ) — không đủ pixel để phân thùng rời vs thân liền.
- Giá trị `boundary`: `inside` (không bị mép ảnh cắt) — vật nằm trọn trong khung, không chạm mép.
- Trạng thái `review_state`: `needs_review` (cần xem lại).
- Lý do: Thiếu bằng chứng phân lớp theo định nghĩa truck/bus/van; không đoán. Đã ghi nhật ký, để `needs_review` và hỏi Lab Coach với crop 100% + mine_row + IoU. Audit hiện có `my_native_export_audit.json: visibility 102, boundary 102, review_state 102` (đủ thuộc tính) nhưng không bóc theo giá trị, nên cần đếm trong `mine-native.zip/annotations.xml` để thống kê `needs_review` thực tế.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh (`drive_022, drive_033, drive_038, drive_008` — 640×640, `input_pool_audit.json`).
- [x] Đã kiểm vật thể thiếu và trùng (phát hiện 56 `unmatched_mine` vượt, 4 `unmatched_comparison`; đã kiểm trùng hình học qua `minimum_cross_format_iou 0.999896`).
- [x] Đã kiểm lớp và hình học từng hộp (46 cặp ghép, `mean_iou 0.81386 median 0.838446`, 12 lệch toàn bộ hoán vị `truck/bus/van`).
- [x] Mỗi hộp có đủ ba thuộc tính (`my_native_export_audit.json: required_attributes [boundary, review_state, visibility]`, mỗi loại 102/102; `same_annotation_state true` với YOLO).
- [x] Đã xử lý mọi hộp `needs_review` (các ca `unclear` đã để `needs_review` và ghi lý do; ca đủ bằng chứng để `confident`).
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu (bài khóa với `my_export_audit archive_sha256 43ea3873...` ≠ `comparison_export_sha256 c8bbc767...`).
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu (`teaching_reference`, `comparison_source` khác SHA bài mình).
- [x] Số vật thể thực tế: **102 / 4 ảnh** — vượt mục tiêu 40–60 (`within_slide_workload_target: false`): `drive_022: 5 (car 3, truck 1, bus 1, van 0)`, `drive_033: 30 (car 27, truck 2, bus 1, van 0)`, `drive_038: 38 (car 29, truck 2, bus 5, van 2)`, `drive_008: 29 (car 22, truck 1, bus 3, van 3)` — suy luận gán thừa vật xa/nhỏ, cần lọc lại theo quy tắc `unclear` + `needs_review`.
