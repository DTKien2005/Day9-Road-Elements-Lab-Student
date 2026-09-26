# Edge-case library

Thư viện ca biên chuyên sâu về đèn giao thông cho xe tự hành — Nhóm TrafficVision-AI (Lead: ĐỖ TRUNG KIÊN). Toàn bộ 9 ca biên mẫu dưới đây đều thuộc các ảnh thực tế được nạp trực tiếp trong hệ thống CVAT (Task 26 Calibration và Task 27 Golden Blind).

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
Observation: Ban đêm vỏ hộp đèn đen bị chìm vào bóng tối, chỉ nhìn thấy 2 bóng đèn tròn xanh ngọc phát sáng lơ lửng điều khiển xe đi thẳng; góc vỉa hè bên phải có 1 đầu đèn người đi bộ màu đỏ; phía trên cao có các đèn đường cao áp chiếu sáng đô thị màu vàng/cam uốn lượn.
Decision: LABEL
Expected:
1. Áp dụng Visible Lamp Rule cho 2 bóng đèn xanh: vẽ box nhỏ ôm khít quầng bóng sáng (~10x10 px), state=green, relevance=ego_relevant, occluded=true (do vỏ hộp bị chìm vào bóng tối).
2. Đầu đèn người đi bộ bên phải vỉa hè: state=red, relevance=pedestrian, occluded=false.
3. Các bóng đèn đường cao áp màu vàng cam đơn lẻ: BỎ QUA (IGNORE — Tuyệt đối không vẽ box).
Rationale: Đảm bảo xe tự hành nhận diện đúng tín hiệu làn mình ban đêm mà không bị nhiễu bởi đèn đường và đèn hậu xe khác.
Common mistake: Vẽ trùm quầng sáng lóa khổng lồ, hoặc vẽ nhầm đèn cao áp vàng thành đèn vàng giao thông.
Diversity: low_visibility

---

CASE ID: EC03
Sample: BDD26
Scene: Đường phố ban đêm ánh sáng phức tạp, ngã tư gần và các đầu đèn nhỏ ở ngã tư xa
Observation: Ở ngã tư gần xe đang tới có 1 đầu đèn đi thẳng màu xanh (`green`) điều khiển xe mình và 1 đầu đèn người đi bộ màu đỏ (`red`) phía dưới. Đồng thời phía xa dọc tuyến đường (vị trí x ≈ 660–716, y ≈ 220–232) có thêm 3 đầu đèn kích thước nhỏ (~6–8 px) gồm 2 đèn đỏ và 1 đèn xanh mờ ảo trong đêm.
Decision: LABEL
Expected: Vẽ đầy đủ 5 bounding box:
1. Nhóm ngã tư gần (2 box rõ nét):
   - Box 1 (Đèn đi thẳng chính diện): state=green, relevance=ego_relevant, needs_review=false.
   - Box 2 (Đèn người đi bộ): state=red, relevance=pedestrian, needs_review=false.
2. Nhóm ngã tư xa / dọc đường (3 box nhỏ, mờ ảo trong đêm):
   - Box 3: state=red, relevance=ego_relevant, needs_review=true.
   - Box 4: state=red, relevance=ego_relevant, needs_review=true.
   - Box 5: state=green, relevance=unknown, needs_review=true.
Rationale: Ghi nhận trọn vẹn cả tín hiệu điều khiển ngã tư gần lẫn các nguồn tín hiệu phát hiện được ở ngã tư xa, đồng thời kích hoạt cờ needs_review=true cho các đèn nhỏ mờ xa để chuyên gia thẩm định.
Common mistake: Chỉ vẽ 2 đèn to ở gần mà bỏ sót toàn bộ các đầu đèn ở xa, hoặc quên không gắn cờ needs_review=true cho đèn mờ xa.
Diversity: critical; small_far

---

CASE ID: EC04
Sample: BDD12
Scene: Cảnh đường phố ban ngày nhiều mây, tầm nhìn xa
Observation: Ở ngã tư xa xuất hiện đầu đèn tín hiệu người đi bộ / tín hiệu giao thông kích thước nhỏ (~10-15 px), nhìn thấy rõ viền hộp nhưng khá mờ.
Decision: LABEL
Expected: Bounding box ôm khít đầu đèn, state=red, relevance=pedestrian (hoặc unknown nếu mờ không chắc màu), occluded=false.
Rationale: Duy trì độ nhạy (Recall) cho hệ thống nhận diện từ xa nhưng phân định đúng đối tượng người đi bộ.
Common mistake: Bỏ qua không vẽ vì nghĩ ở quá xa hoặc nhầm thành đèn của xe mình.
Diversity: small_far

---

CASE ID: EC05
Sample: BDD15
Scene: Giao lộ đô thị ban ngày, đèn ở xa bị che khuất một phần
Observation: Phía bên kia ngã tư có các đầu đèn tín hiệu điều khiển giao thông nhưng bị che khuất một phần bởi kết cấu hoặc tầm nhìn xa mờ nhòe.
Decision: LABEL
Expected: Vẽ box chữ nhật ước lượng ôm đầu đèn:
- Đầu đèn nhìn rõ màu: state=red, relevance=ego_relevant.
- Đầu đèn bị mờ/che khuất: state=unknown, relevance=ego_relevant, occluded=true hoặc needs_review=true.
Rationale: Giúp mô hình học được đặc trưng đèn bị che khuất một phần (Amodal detection) tại các giao lộ có tầm nhìn hạn chế.
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
Observation: Có 2 đầu đèn đi thẳng màu xanh treo trên cao điều khiển làn xe mình, và 1 đầu đèn người đi bộ màu đỏ bên phải.
Decision: LABEL
Expected: Vẽ tách biệt 3 box:
- 2 đầu đèn đi thẳng: state=green, relevance=ego_relevant, occluded=false.
- 1 đầu đèn người đi bộ: state=red, relevance=pedestrian, occluded=false.
Rationale: Giao lộ có nhiều luồng giao thông cùng lúc, cần phân tách rõ luồng xe tự hành và luồng người đi bộ.
Common mistake: Gom các đầu đèn gần nhau thành 1 box to, hoặc nhầm đèn người đi bộ đỏ thành tín hiệu cấm xe mình.
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
