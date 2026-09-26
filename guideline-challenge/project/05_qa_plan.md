# QA plan + quality gates

Kế hoạch đảm bảo chất lượng và cổng kiểm soát cho dự án gán nhãn Đèn Giao Thông Xe Tự Hành.

## Flow

Quy trình vận hành: Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

- **Ai review, review bao nhiêu:** QA Lead (Lê Văn C) review độc lập 100% các mẫu có gắn cờ `needs_review=true`, 100% mẫu ban đêm/thời tiết xấu, và lấy mẫu ngẫu nhiên 20% toàn bộ các ảnh thông thường.
- **Chọn sample theo rule nào:** Phân tầng dựa trên rủi ro (Risk-stratified sampling): Ưu tiên cao nhất cho ảnh có tag `conflict`, `low_visibility`, `small_far`, sau đó đến các ảnh có nhiều hơn 3 đầu đèn.
- **Issue được ghi ở đâu, đóng thế nào:** Ghi nhận trực tiếp vào Review Log (Google Sheets / CSV) gồm `sample_id`, `box_id`, `severity`, `defect_type`, `annotator`, `suggested_action`. Issue chỉ được đóng khi annotator sửa trực tiếp trên CVAT và QA Lead bấm verify.
- **Khi phát hiện guideline gap thì update và version ra sao:** Khi có trên 2 annotator bất đồng về cùng 1 quy tắc (hoặc peer reviewer phát hiện case chưa có trong guideline), QA Lead triệu tập họp khẩn 10 phút, cập nhật quy tắc vào `02_guideline.md`, tăng số version (`v1` → `v2` → `v3`) và ghi nhật ký vào `08_revision_log.md`.

## Defect severity

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| **Critical** | Sai lệch trực tiếp đe dọa an toàn tính mạng: Đảo ngược màu đỏ/xanh cho ego lane, hoặc gán nhầm đèn rẽ thành đèn điều khiển ego lane gây phanh gấp / đâm va. | Gán đèn đỏ rẽ trái thành `relevance=ego_relevant`, nhận nhầm đèn đỏ thành xanh. | Dừng batch, hoàn trả 100% task của annotator để làm lại (re-annotate toàn bộ). |
| **Major** | Sai lệch hình học hoặc phân loại đáng kể nhưng không dẫn tới va chạm ngay: Bỏ sót đầu đèn >= 15 px, vẽ box bao cả cột đèn, nhầm đèn đường thành đèn giao thông. | Vẽ 1 box ôm cả cột đèn cao 100px; nhầm đèn cao áp thành traffic light. | Trả về cho annotator sửa lại các mẫu lỗi trong vòng 30 phút. |
| **Minor** | Sai lệch nhỏ trong dung sai cho phép: Box lệch từ 3 - 5 px, quên tick `occluded` khi đèn bị che khoảng 50%. | Box thừa mép 4px, quên tick che khuất. | QA sửa trực tiếp tại chỗ (in-line fix) và nhắc nhở annotator. |
| **Question** | Điểm ảnh quá mờ nhòe hoặc chói lóa không thể khẳng định chắc chắn bằng mắt thường. | Đốm sáng nhỏ 9px ở ngã tư xa trong đêm tuyết. | Escalation lên Safety Engineer để đối chiếu dữ liệu bản đồ số (HD Map) hoặc log xe. |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| **Critical Defect Rate (CDR)** | (Số lỗi Critical / Tổng số decisions) * 100% | Bắt buộc phải bằng 0%. Xe tự hành không được phép chấp nhận bất kỳ lỗi đỏ/xanh nào. |
| **Relevance Accuracy** | (Số đèn gán đúng relevance / Tổng số đèn hợp lệ) * 100% | Đánh giá khả năng hiểu ngữ cảnh giao lộ và phân định làn của người gán nhãn. |
| **Mean IoU (Geometry)** | Trung bình IoU của các box so với Gold Box | Đảm bảo vị trí đầu đèn chuẩn xác để thuật toán crop ảnh zoom-in hoạt động tốt. |

- **Metric high-risk tách riêng:**
  - **Critical Defect Escape Rate:** Tỷ lệ lỗi Critical lọt qua khâu Review sang khâu Production → Mục tiêu tuyệt đối: 0.0%.

## Quality gate

```text
PASS if:
  - Critical Defect Rate == 0%
  - Relevance Accuracy >= 95%
  - Geometry Mean IoU >= 0.85
  - Tỷ lệ giá trị __undefined__ còn sót lại == 0%

REWORK if:
  - Critical Defect Rate > 0% (dù chỉ 1 lỗi)
  - Hoặc Relevance Accuracy < 95%
  - Hoặc Geometry Mean IoU trong khoảng [0.75, 0.85)

REJECT / ESCALATE if:
  - Toàn bộ batch có trên 3 lỗi Critical
  - Hoặc Mean IoU < 0.75
  - Hành động: Hủy bỏ kết quả của annotator, huấn luyện lại quy tắc (re-training), chuyển batch cho annotator senior thực hiện.
```

- **Trade-off cost/risk:** Nhóm ưu tiên **Safety over Cost**. Thà chấp nhận tốn thời gian gán `needs_review` và QA kiểm tra thủ công nhiều lần còn hơn để lọt một lỗi nhận định sai đèn đỏ cho xe tự hành.
