# Problem statement + downstream contract

## Bài toán

Gán nhãn phát hiện vị trí (bounding box), trạng thái tín hiệu (`state`: red, yellow, green, off, unknown) và mức độ liên quan tới xe tự hành (`relevance`: ego_relevant, other_lane, pedestrian, unknown) cho từng đầu đèn giao thông (`traffic_light`) độc lập tại các nút giao thông đô thị phức tạp, điều kiện ánh sáng đa dạng (ngày, đêm, hoàng hôn, tuyết) và phân biệt rõ ràng giữa các giao lộ liên tiếp.

## Downstream contract

1. **Downstream task / model / user là ai?**
   Module Perception (Traffic Light Recognition) kết hợp AV Motion Planner của hệ thống xe tự hành cấp độ 4 (Autonomous Vehicle Level 4). Planner sử dụng trạng thái màu đèn và độ liên quan làn để ra quyết định Stop/Go, Yield và chuyển làn tại giao lộ.

2. **Output annotation nào thực sự cần?**
   - **Geometry**: Bounding box hình chữ nhật (`rectangle`) bao chặt từng đầu đèn tín hiệu riêng lẻ (`signal head`).
   - **Class**: `traffic_light`.
   - **Attributes**:
     - `state`: Trạng thái màu đang sáng (`red`, `yellow`, `green`, `off`, `unknown`).
     - `relevance`: Quyền điều khiển đối với làn đường của xe (`ego_relevant`, `other_lane`, `pedestrian`, `unknown`).
     - `occluded`: Cờ nhị phân (`true`/`false`) khi đầu đèn bị che khuất $\ge 50\%$.
     - `needs_review`: Cờ đánh dấu nghi vấn (`true`/`false`) cho QA reviewer.
   - **Tag ảnh**: `image_escalate` cho các khung hình bất khả kháng (chói lóa toàn cảnh, hỏng dữ liệu).

3. **Failure nào gây hậu quả lớn nhất? (Critical Failure)**
   - **False Green cho Ego Lane**: Nhận diện đèn đỏ của làn xe đang chạy thành xanh (`state=green` hoặc gán nhầm `relevance=ego_relevant` từ một đèn xanh của làn rẽ), dẫn đến việc xe tự hành lao thẳng vào giao lộ gây va chạm trực diện với luồng xe đối diện.
   - **Phanh khẩn cấp vô lý (Phantom Braking)**: Nhầm đèn đỏ của giao lộ phía sau (far intersection) hoặc đèn đỏ của làn rẽ (`other_lane`) thành đèn đỏ điều khiển ego lane (`ego_relevant`), khiến xe phanh gấp nguy hiểm giữa dòng giao thông đang chạy tốc độ cao.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   Khi không đủ bằng chứng hình học hoặc ngữ cảnh thời gian (đèn mờ nhòe $< 8\text{ px}$, lóa đèn xe ban đêm, góc giao lộ không rõ ràng):
   - Đặt `state=unknown` và `relevance=unknown`.
   - Đánh dấu checkbox `needs_review=true`.
   - Nếu toàn bộ ảnh bị mù sáng/cháy sáng không thể đọc được cảnh: gán tag `image_escalate`.
   - QA Lead và AV Safety Engineer là người duyệt tầng cuối trong Review Phase.

## Scope

- **Trong scope (bắt buộc label):**
  - Mọi đầu đèn tín hiệu giao thông đường bộ nhìn thấy được (chiều dài cạnh lớn nhất $\ge 8\text{ px}$), bao gồm đèn tròn thông thường, đèn mũi tên, đèn kiểm soát làn, đèn người đi bộ và đèn đang tắt (`off`).
  - Giao lộ gần (near/foreground intersection) và giao lộ kế tiếp nhìn thấy được trong tầm nhìn (far/background intersection).
- **Ngoài scope (ignore - tuyệt đối không vẽ box):**
  - Đèn chiếu sáng đô thị (đèn đường đơn lẻ không có hộp đèn giao thông), đèn hậu xe hơi (tail lights), đèn phanh, đèn biển quảng cáo/neon.
  - Hình phản chiếu của đèn giao thông trên mặt đường ướt, vũng nước hoặc kính xe buýt/tòa nhà.
  - Cột trụ (pole), khung giàn treo (gantry), biển tên đường hoặc camera giám sát gắn kèm.
  - Đèn tín hiệu quá nhỏ mờ ($< 8\text{ px}$) không còn nhận ra cấu trúc cụm đèn.
- **Geometry tolerance:**
  - Box phải ôm chặt phần vỏ hộp đèn (`signal head`), không lấy mào che nắng quá rộng hay thanh đỡ.
  - Sai số cho phép: $\le 3\text{ px}$ mỗi cạnh trên ảnh chuẩn $1280 \times 720$.

## Output chấm được

Mọi quyết định trong blind test đều được phản ánh tường minh qua file CVAT XML / CVAT for Images 1.1:
- `LABEL`: Tạo 1 shape `rectangle` gán label `traffic_light`.
- `IGNORE`: Không tồn tại box trên đối tượng rác (đèn đường, đèn xe, phản chiếu).
- `UNKNOWN`: Box tồn tại với attribute `state="unknown"` và `relevance="unknown"`.
- `ESCALATE`: Box có attribute `needs_review="true"` hoặc ảnh có tag `image_escalate`.

## Dữ liệu và giới hạn

- **Nguồn ảnh:** `bdd100k` (chứa các cảnh thành phố ngày, đêm, mưa, tuyết, hoàng hôn) và clip `lisa` (chuỗi frame liên tiếp có đèn chuyển màu).
- **Số lượng sử dụng:** 4 ảnh `example`, 6 ảnh `calibration`, 5 ảnh `blind`.
- **Giới hạn đã biết:** Ảnh BDD100K là ảnh đơn độc lập (frame tĩnh), không thể nội suy temporal. Chuỗi LISA là video liên tiếp từ một xe tiếp cận giao lộ, giúp kiểm chứng tính nhất quán thời gian (temporal consistency).
