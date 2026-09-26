# Edge-case library

Thư viện ca biên chuyên sâu về đèn giao thông cho xe tự hành — Nhóm TrafficVision-AI (Lead: ĐỖ TRUNG KIÊN). Toàn bộ 10 ca biên mẫu dưới đây đều thuộc các ảnh thực tế được nạp trực tiếp trong hệ thống CVAT (Task 26 Calibration và Task 27 Golden Blind).

---

CASE ID: EC01
Sample: BDD11
Scene: Giao lộ đô thị ban ngày có đèn tín hiệu người đi bộ
Observation: Ở góc vỉa hè bên phải giao lộ có cột gắn các đầu đèn tín hiệu dành cho người đi bộ qua đường (bóng đỏ và bóng xanh). Không có đèn tín hiệu điều khiển làn xe của mình.
Decision: LABEL
Expected: Vẽ các bounding box riêng biệt cho từng đầu đèn người đi bộ: Box 1: state=red, relevance=pedestrian; Box 2: state=green, relevance=pedestrian. Tuyệt đối không gán ego_relevant vì đây là đèn cho người đi bộ.
Rationale: Xe tự hành cần nhận biết trạng thái đèn người đi bộ để dự báo luồng người cắt ngang đường, nhưng không được hiểu nhầm thành đèn của luồng xe cơ giới để tránh phanh nhầm.
Common mistake: Gán nhầm relevance=ego_relevant cho đèn người đi bộ hoặc bỏ qua không vẽ vì nghĩ chỉ vẽ đèn ô tô.
Diversity: conflict

---

CASE ID: EC02
Sample: BDD18
Scene: Đường phố ban đêm ánh sáng phức tạp, đèn đường và đèn tín hiệu
Observation: Ban đêm vỏ hộp đèn đen bị chìm vào bóng tối, chỉ nhìn thấy 2 bóng đèn tròn xanh ngọc phát sáng lơ lửng điều khiển xe đi thẳng; góc vỉa hè bên phải có 2 đầu đèn tín hiệu người đi bộ (1 đầu đèn đỏ và 1 đầu đèn mờ state=unknown); phía trên cao có các đèn đường cao áp chiếu sáng đô thị màu vàng/cam uốn lượn.
Decision: LABEL
Expected: Vẽ đủ 4 bounding box:
1. Áp dụng Visible Lamp Rule cho 2 bóng đèn xanh: vẽ box nhỏ ôm khít quầng bóng sáng (~10x10 px), state=green, relevance=ego_relevant, occluded=true (do vỏ hộp bị chìm vào bóng tối).
2. Hai đầu đèn người đi bộ bên phải vỉa hè:
   - Box 3: state=red, relevance=pedestrian, occluded=false.
   - Box 4: state=unknown, relevance=pedestrian, occluded=false (đèn người đi bộ mờ bên cạnh).
3. Các bóng đèn đường cao áp màu vàng cam đơn lẻ: BỎ QUA (IGNORE — Tuyệt đối không vẽ box).
Rationale: Đảm bảo xe tự hành nhận diện đúng tín hiệu làn mình ban đêm mà không bị nhiễu bởi đèn đường và đèn hậu xe khác.
Common mistake: Vẽ trùm quầng sáng lóa khổng lồ, hoặc vẽ nhầm đèn cao áp vàng thành đèn vàng giao thông.
Diversity: low_visibility

---

CASE ID: EC03
Sample: BDD26
Scene: Đường phố ban đêm ánh sáng phức tạp, ngã tư gần và các đầu đèn nhỏ ở ngã tư xa
Observation: Ở ngã tư gần xe đang tới có 1 đầu đèn đi thẳng màu xanh (`green`) điều khiển xe mình và 1 đầu đèn người đi bộ màu đỏ (`red`) phía dưới. Đồng thời phía xa dọc tuyến đường (vị trí x ≈ 660–716, y ≈ 220–232) có thêm 3 đầu đèn ở sát ngưỡng 8 px gồm 2 đèn đỏ và 1 đèn xanh mờ ảo trong đêm.
Decision: LABEL
Expected: Vẽ đầy đủ 5 bounding box:
1. Nhóm ngã tư gần (2 box rõ nét):
   - Box 1 (Đèn đi thẳng chính diện): state=green, relevance=ego_relevant, needs_review=false.
   - Box 2 (Đèn người đi bộ): state=red, relevance=pedestrian, needs_review=false.
2. Nhóm ngã tư xa / dọc đường (3 box nhỏ, mờ ảo trong đêm):
   - Box 3: state=red, relevance=other_lane, needs_review=true.
   - Box 4: state=red, relevance=other_lane, needs_review=true.
   - Box 5: state=green, relevance=unknown, needs_review=true.
Rationale: Ghi nhận trọn vẹn tín hiệu ngã tư gần và các signal head đủ ngưỡng ở ngã tư xa, nhưng bắt buộc tách far intersection khỏi ego lane để tránh phantom braking; các đèn nhỏ mờ xa phải có needs_review=true.
Common mistake: Gán đèn đỏ ngã tư xa là ego_relevant, bỏ sót đầu đèn xa đủ ngưỡng, hoặc quên needs_review=true.
Diversity: critical; small_far

---

CASE ID: EC04
Sample: BDD12
Scene: Cảnh đường phố ban ngày nhiều mây, tầm nhìn xa
Observation: Ở ngã tư xa xuất hiện 2 đầu đèn tín hiệu kích thước nhỏ (~10-15 px): 1 đầu đèn người đi bộ màu đỏ và 1 đầu đèn bị che khuất mờ ảo bên cạnh.
Decision: LABEL
Expected: Vẽ 2 bounding box:
- Box 1: state=red, relevance=pedestrian, occluded=false.
- Box 2: state=unknown, relevance=unknown, occluded=true.
Rationale: Duy trì độ nhạy (Recall) cho hệ thống nhận diện từ xa và ghi nhận thuộc tính che khuất một phần.
Common mistake: Bỏ qua không vẽ vì nghĩ ở quá xa hoặc nhầm thành đèn của xe mình.
Diversity: small_far

---

CASE ID: EC05
Sample: BDD15
Scene: Giao lộ đô thị ban ngày, đèn ở xa bị che khuất một phần
Observation: Phía bên kia ngã tư có các đầu đèn tín hiệu điều khiển giao thông nhưng bị che khuất một phần bởi kết cấu hoặc tầm nhìn xa mờ nhòe.
Decision: LABEL
Expected: Vẽ 3 box chữ nhật ôm các đầu đèn:
- Box 1: state=red, relevance=ego_relevant (đèn đỏ xe đi thẳng nhìn rõ).
- Box 2: state=unknown, relevance=ego_relevant (đèn xe ở xa).
- Box 3: state=unknown, relevance=other_lane (đèn thuộc làn đường khác).
Rationale: Giúp mô hình học được đặc trưng đèn bị che khuất một phần (Amodal detection) và phân định luồng xe tại các giao lộ có tầm nhìn hạn chế.
Common mistake: Bỏ qua các đầu đèn bị che khuất hoặc vẽ box quá rộng bao trùm vật cản.
Diversity: occlusion

---

CASE ID: EC06
Sample: BDD17
Scene: Đường phố trời mưa, mặt đường nhựa ướt sũng phản chiếu ánh đèn
Observation: Các đầu đèn tín hiệu trên cao (xanh đi thẳng, đỏ làn khác, đèn người đi bộ) chiếu sáng và in vệt phản chiếu dài rực rỡ dưới mặt đường ướt.
Decision: LABEL
Expected:
- Chỉ vẽ các đầu đèn vật lý thật trên giá treo / cột cao: 2 đèn xanh đi thẳng (state=green, relevance=ego_relevant), 1 đèn đỏ làn khác (state=red, relevance=other_lane), các đèn người đi bộ (state=red/unknown, relevance=pedestrian).
- TUYỆT ĐỐI BỎ QUA các vệt phản chiếu loang loáng dưới mặt đường nhựa ướt.
Rationale: Phản chiếu không phải là nguồn tín hiệu vật lý. Nếu vẽ box sẽ làm mô hình học sai đặc trưng nền và phanh nhầm xuống mặt đường.
Common mistake: Vẽ box lên vệt sáng đỏ/xanh dưới mặt đường ướt.
Diversity: escalation

---

CASE ID: EC07
Sample: BDD25
Scene: Giao lộ chạng vạng hoàng hôn có nhiều luồng đèn
Observation: Có 2 đầu đèn đi thẳng màu xanh treo trên cao điều khiển làn xe mình, 1 đầu đèn người đi bộ mờ bên phải, và 1 đầu đèn xanh ở xa phía sau.
Decision: LABEL
Expected: Vẽ tách biệt 4 box:
- Box 1 & 2 (2 đầu đèn đi thẳng): state=green, relevance=ego_relevant, occluded=false.
- Box 3 (Đèn người đi bộ): state=unknown, relevance=pedestrian, occluded=false.
- Box 4 (Đèn xanh ở xa): state=green, relevance=unknown, needs_review=true.
Rationale: Giao lộ có nhiều luồng giao thông cùng lúc, cần phân tách rõ luồng xe tự hành và luồng người đi bộ, gắn cờ review cho đèn xa.
Common mistake: Gom các đầu đèn gần nhau thành 1 box to, hoặc nhầm đèn người đi bộ thành tín hiệu xe mình.
Diversity: critical

---

CASE ID: EC08
Sample: BDD14
Scene: Đường cao tốc ngoại ô ban ngày có biển báo chỉ dẫn trên cao
Observation: Có biển báo chỉ dẫn hướng đi trên giàn treo lớn bắc ngang cao tốc nhưng hoàn toàn KHÔNG CÓ đèn tín hiệu giao thông nào.
Decision: IGNORE
Expected: Không tạo bất kỳ annotation nào (0 boxes).
Rationale: Mẫu kiểm tra âm tính (Negative sample) xác nhận annotator và mô hình AI không bị nhầm lẫn giữa biển báo trên cao và đèn tín hiệu giao thông.
Common mistake: Gán nhầm biển báo hình chữ nhật thành đèn giao thông tắt (`state=off`).
Diversity: negative

---

CASE ID: EC09
Sample: BDD24
Scene: Cảnh đô thị ban ngày mùa đông tuyết phủ, độ tương phản thấp
Observation: Cảnh đường tuyết mùa đông với ánh sáng tán xạ mạnh, không có đầu đèn tín hiệu giao thông hợp lệ nào điều khiển làn xe mình.
Decision: IGNORE
Expected: Không tạo bounding box (0 boxes).
Rationale: Kiểm tra khả năng loại bỏ cảnh nhiễu thời tiết tuyết rơi, tránh việc annotator cố đoán mò hoặc vẽ box vào các vật thể lạ bị tuyết bám.
Common mistake: Đoán mò và vẽ box vào các mảng tuyết trắng bám trên thanh ngang.
Diversity: ambiguity

---

CASE ID: EC10
Sample: BDD13
Scene: Cảnh đường phố đô thị ban ngày rõ nét
Observation: Ở góc vỉa hè bên phải có đầu đèn tín hiệu người đi bộ nhưng đang tắt hoàn toàn không sáng bóng nào (mất điện hoặc chưa bật).
Decision: LABEL
Expected: Vẽ 1 bounding box ôm khít đầu đèn: state=off, relevance=pedestrian, occluded=false, needs_review=false.
Rationale: Xe tự hành cần nhận diện đầu đèn đang tắt (`state=off`) để kích hoạt chế độ nhường đường an toàn khi qua giao lộ mất điện theo luật giao thông.
Common mistake: Bỏ qua không vẽ box vì nghĩ đèn tắt là không cần label.
Diversity: ambiguity
