# Peer feedback + owner response

- **Nhóm peer:** dongtinh
- **Nguồn bài blind:** CVAT Task #28 / Job #22
- **Nguồn calibration peer:** CVAT Task #29 / Job #23
- **Export chấm:** `peer_output/dongtinh_blind.zip`

Không có câu hỏi hay phản hồi văn bản trực tiếp được cung cấp trong lần chấm này. Năm mục dưới đây là **owner audit từ export CVAT thật**, không phải trích dẫn lời peer.

## 1. Năm câu hỏi usability

1. **Rule nào thể hiện rõ nhất?** Peer để BDD14 và BDD24 ở 0 box, phân biệt đúng đèn đường/ảnh âm tính; ở BDD26, hai đèn đỏ giao lộ xa được gán `other_lane`, tránh phantom braking.
2. **Chỗ nào còn khó?** Thuộc tính `occluded` và `needs_review` chưa nhất quán: hai ego head ban đêm ở BDD18 không được tick `occluded`; các đầu đèn nhỏ xa ở BDD26 không được tick `needs_review`.
3. **Sample nào làm lộ chỗ thiếu?** BDD26 cho thấy peer bỏ sót đầu đèn xanh nhỏ ở xa và thiếu cờ review. BDD15 trong calibration cũng bị thiếu một đầu đèn `other_lane`.
4. **Thông tin CVAT nào dễ điền sai?** `state`/`occluded` ở BDD11 và `relevance` ở BDD12 khác owner; đây là các thuộc tính cần ví dụ trực quan hơn.
5. **Thay đổi nào giúp người mới?** Bổ sung checklist sau mỗi ảnh: quét đủ head nhỏ; far intersection dùng `other_lane`; mọi head 8–15 px hoặc màu/làn chưa chắc phải `needs_review=true`; chỉ tick `occluded` khi đạt ngưỡng 50%.

## 2. Owner phân loại

| Evidence | Nguyên nhân | Xử lý | Căn cứ |
|---|---|---|---|
| BDD18: 4 box đúng loại nhưng hai ego head có `occluded=false` | execution_error | coaching | Guideline Visible Lamp Rule yêu cầu `occluded=true` khi vỏ hộp chìm trong bóng tối. |
| BDD25: far green được gán `other_lane` nhưng thiếu `needs_review`; pedestrian là red thay vì unknown so với Task #27 | data_ambiguity | add_escalation | Task #27; đối chiếu CVAT cho thấy state/relevance agreement 75%. |
| BDD26: 4 box thay vì 5; thiếu far green và các far red không có `needs_review` | execution_error | coaching | Task #27 có 5 box; guideline mục near/far yêu cầu quét đủ head và review head nhỏ xa. |
| BDD15 calibration: 2 box thay vì 3 | execution_error | coaching | `06_calibration_measure.csv`: count 3 so với 2. |
| BDD11/BDD12 calibration: khác state/occluded/relevance | guideline_gap | accept + revise | `06_calibration_measure.csv`; thêm ví dụ state off và điều kiện dùng pedestrian/unknown. |

## 3. Sai lệch gold phát hiện sau freeze

Gold và `sample_pack.csv` được giữ nguyên. Các decision sau không khớp Task #27/guideline; peer làm đúng dữ liệu hiện hành nhưng vẫn phải nhận `correct=0` theo frozen gold. Ghi chú trong `transfer_score.csv` bắt đầu bằng `gold sai:`.

| Decision | Sai lệch frozen gold | Bằng chứng peer |
|---|---|---|
| BDD18 / D04 | Gold yêu cầu `red + ego_relevant`; Task #27/guideline có hai head `green + ego_relevant`. | Peer gán đúng hai green ego head. |
| BDD24 / D06–D07 | Gold yêu cầu box unknown + review; Task #27/guideline xác định 0 box. | Peer để 0 box. |
| BDD25 / D08 | Gold yêu cầu `red + other_lane`; Task #27 không có head này. | Peer có far green `other_lane`, không có red other_lane. |

## 4. Kết luận

Peer nắm tốt inclusion/exclusion và near/far semantics. Lỗi còn lại tập trung ở recall cho head nhỏ và các cờ QA. Các lỗi frozen gold được debrief riêng, không quy thành lỗi guideline hay năng lực peer.
