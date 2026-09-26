# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong `02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng calibration report, câu hỏi trong clarification log, feedback của peer).

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| `v1` | Xây dựng 10 mục khung sườn, định nghĩa ontology 1 class `traffic_light` và 4 attributes (`state`, `relevance`, `occluded`, `needs_review`) cùng tag `image_escalate`. | Khởi tạo dự án theo slide lý thuyết DTLD và yêu cầu bài toán Perception AV. | `01_problem_statement.md`, `03_cvat_labels.json` |
| `v2` | Bổ sung quy tắc cấm lấy cột gantry, quy tắc phân biệt ngã tư gần vs ngã tư xa (Near vs Far Intersection), làm rõ ngưỡng kích thước >= 8 px và xử lý cụm nhiều đầu đèn, đèn tắt. | Khắc phục bất đồng nghiêm trọng giữa các annotator trong pha Calibration nội bộ. | `06_calibration_report.csv` (các dòng BDD11, BDD12, BDD15) |
| `v3` | Làm rõ `relevance=other_lane` cho giao lộ kế tiếp, checklist quét head nhỏ và yêu cầu `needs_review`/`occluded`; ghi riêng các decision frozen gold sai mà không sửa gold sau freeze. | Export thật từ nhóm dongtinh ở Task #28 cho thấy thiếu far green và cờ QA; calibration Task #29 cho thấy bất đồng state/occluded/relevance. | `06_calibration_measure.csv`, `07_blind_handoff/transfer_score.csv`, `07_blind_handoff/peer_feedback.md` |
