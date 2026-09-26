# Problem statement + downstream contract

## Bài toán

Gán một bounding box cho từng đầu đèn giao thông (`traffic_light`) và ghi trạng thái `state` (`red`, `yellow`, `green`, `off`, `unknown`) cùng mức liên quan `relevance` (`ego_relevant`, `other_lane`, `pedestrian`, `unknown`). Phạm vi tập trung vào giao lộ nhiều đầu đèn và điều kiện khó như ban đêm, hoàng hôn, tuyết, che khuất và hai giao lộ liên tiếp.

## Bốn câu hỏi downstream contract

1. **Ai sử dụng dữ liệu?** Module nhận biết đèn giao thông và motion planner của hệ thống xe tự hành. Planner dùng màu đèn và mức liên quan tới ego lane để quyết định dừng, đi hoặc nhường đường.

2. **Output annotation nào thực sự cần?** Mỗi signal head là một rectangle riêng, có `state`, `relevance`, `occluded` và `needs_review`. Ảnh hỏng hoặc không thể đọc ngữ cảnh được gắn tag `image_escalate`. Mọi quyết định LABEL, IGNORE, UNKNOWN và ESCALATE đều phải nhìn thấy hoặc kiểm tra được trong export CVAT for Images 1.1.

3. **Failure nào nghiêm trọng nhất?** False green cho ego lane có thể khiến xe đi vào giao lộ khi phải dừng. Ngược lại, gán đèn đỏ của làn khác hoặc giao lộ phía xa thành `ego_relevant` có thể gây phantom braking. Hai lỗi này được xếp critical và không được lọt qua quality gate.

4. **Escalation path là gì?** Khi không đủ bằng chứng về màu hoặc làn điều khiển, annotator dùng `state=unknown`, `relevance=unknown`, `needs_review=true`; nếu toàn ảnh không dùng được thì gắn `image_escalate`. QA Lead duyệt trước, AV Safety Engineer quyết định cuối với ca critical hoặc chưa thể resolve.

## Scope và dữ liệu

- **Label:** đầu đèn xe cơ giới, người đi bộ hoặc xe đạp nhìn thấy đủ cấu trúc và đạt ngưỡng 8 px; mỗi đầu đèn một box.
- **Ignore:** đèn đường, đèn xe, quảng cáo, phản chiếu, cột/gantry và đốm sáng dưới 8 px không đủ bằng chứng.
- **Nguồn:** ảnh lớp học từ BDD100K và chuỗi LISA; dùng 4 ảnh example, 6 calibration và 5 blind như `sample_pack.csv`.
