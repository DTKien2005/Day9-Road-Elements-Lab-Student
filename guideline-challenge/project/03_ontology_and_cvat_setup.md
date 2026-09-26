# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` khớp hoàn toàn từng dòng ở đây.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | rectangle | class | n/a | n/a | false | Thực thể vật lý đầu đèn tín hiệu giao thông đường bộ độc lập. |
| `state` | n/a | attribute | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | true | Trạng thái phát sáng của đèn. Mutable vì đèn chuyển màu theo thời gian trong video. Default `__undefined__` bắt buộc annotator chủ động chọn. |
| `relevance` | n/a | attribute | `__undefined__`, `ego_relevant`, `other_lane`, `pedestrian`, `unknown` | `__undefined__` | false | Quyền điều khiển làn đường của xe tự hành. Immutable trong cùng một chuỗi approach tĩnh. |
| `occluded` | n/a | attribute | `false`, `true` | `false` | false | Đánh dấu đầu đèn bị che khuất từ 50% diện tích trở lên bởi cây cối, xe tải hoặc biển báo. |
| `needs_review` | n/a | attribute | `false`, `true` | `false` | false | Đánh dấu các trường hợp nghi vấn (lóa sáng ban đêm, đèn mờ ngã tư xa) để QA Reviewer kiểm tra lại. |
| `image_escalate` | tag | class (tag) | n/a | n/a | false | Tag cho toàn bộ ảnh khi frame bị hư hỏng, lóa trắng toàn màn hình hoặc không thể phân định ngữ cảnh giao thông. |

## Class hay attribute

- **Tại sao chỉ dùng 1 Class `traffic_light` và đưa `state`, `relevance` thành Attribute?**
  Nếu tách thành các Class riêng lẻ (như `traffic_light_red_ego`, `traffic_light_green_other`...), số lượng class sẽ bùng nổ tổ hợp (5 x 4 = 20 class). Điều này gây khó khăn nghiêm trọng cho annotator khi tìm phím tắt, tăng tỷ lệ bấm nhầm nhãn và làm mất liên kết hình học của cùng một đầu đèn qua chuỗi frame video. Dùng attribute giữ nguyên tracking ID của box và chỉ thay đổi thuộc tính `state`.
- **Rủi ro của Default Value:**
  Nếu đặt default là `green` hoặc `ego_relevant`, annotator khi vẽ nhanh có thể quên đổi thuộc tính, dẫn đến lỗi "silent green" hoặc "silent ego" cực kỳ nguy hiểm cho xe tự hành. Do đó, hệ thống đặt default là `__undefined__`. Nếu giá trị này còn sót lại trong file xuất XML, công cụ QA sẽ tự động gắn cờ lỗi chưa hoàn thành.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1` (hoặc `v2.76.0`)
- **Tên task calibration**: `team-traffic-light-calibration` (Task #26), Task Golden/Blind: `team-traffic-light-golden-blind` (Task #27)
- **Guide của task đã dán `02_guideline.md`?**: Đã dán đầy đủ vào mục Task Description / Guide.
- **Nhóm dùng Track hay Shape, vì sao:**
  - Đối với tập ảnh đơn BDD100K: Sử dụng chế độ **Shape** (từng box độc lập).
  - Đối với chuỗi frame video LISA: Sử dụng chế độ **Track** (vẽ box dạng Track để nội suy vị trí giữa các keyframe và theo dõi sự chuyển đổi trạng thái `state` trên cùng một Track ID).

## Setup test

Thành viên Đỗ Trung Kiên mở task thử nghiệm:
- Kiểm tra tạo box chữ nhật quanh đầu đèn bằng phím tắt `N`.
- Menu dropdown hiện đầy đủ `state`, `relevance` với default `__undefined__`.
- Thử nghiệm gán checkbox `occluded` và `needs_review` hoạt động trơn tru.
- Thử gán tag `image_escalate` thành công.
- Lưu ý rút ra: Khi vẽ nhiều đầu đèn sát nhau trên cùng một cột gantry, cần phóng to zoom 400% để vẽ tight box từng head, tránh vẽ 1 box to bao toàn bộ cụm.
