# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong `02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng calibration report, câu hỏi trong clarification log, feedback của peer).

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| `v1` | Xây dựng 10 mục khung sườn, định nghĩa ontology 1 class `traffic_light` và 4 attributes (`state`, `relevance`, `occluded`, `needs_review`) cùng tag `image_escalate`. | Khởi tạo dự án theo slide lý thuyết DTLD và yêu cầu bài toán Perception AV. | `01_problem_statement.md`, `03_cvat_labels.json` |
| `v2` | Bổ sung quy tắc cấm lấy cột gantry, quy tắc phân biệt ngã tư gần vs ngã tư xa (Near vs Far Intersection), làm rõ ngưỡng kích thước $\ge 8\text{ px}$ và xử lý cụm nhiều đầu đèn, đèn tắt. | Khắc phục bất đồng nghiêm trọng giữa các annotator trong pha Calibration nội bộ. | `06_calibration_report.csv` (các dòng BDD11, BDD12, BDD15) |
| `v3` | Hoàn thiện quy tắc loại bỏ đốm sáng đèn cao áp đô thị ban đêm, hướng dẫn chi tiết cách tránh nhầm vệt phản chiếu mặt đường ướt trời mưa, bổ sung escalation path rõ ràng. | Nhóm peer phản hồi trong blind handoff về việc dễ nhầm đèn đường và phản chiếu. | `07_blind_handoff/peer_feedback.md`, `07_blind_handoff/clarification_log.csv` |
