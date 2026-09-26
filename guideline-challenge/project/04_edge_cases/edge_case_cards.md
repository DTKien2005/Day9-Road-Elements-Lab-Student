# Edge-case library

Thư viện ca biên chuyên sâu về đèn giao thông cho xe tự hành. Bộ dữ liệu chứa 9 ca biên mẫu phản ánh đầy đủ các thách thức thực tế trên đường phố.

---

CASE ID: EC01
Sample: BDD10
Scene: Giao lộ đô thị ban ngày có làn rẽ riêng
Observation: Trên cùng một cột có 2 đầu đèn: một đầu đèn tròn màu đỏ và một đầu đèn phụ hình mũi tên rẽ phải màu xanh.
Decision: LABEL
Expected: Tạo 2 bounding box riêng biệt: Box 1 (đầu tròn): state=red, relevance=ego_relevant; Box 2 (mũi tên): state=green, relevance=other_lane (với xe đi thẳng).
Rationale: Giúp AV phân biệt quyền ưu tiên của từng luồng xe. Nếu gom 1 box thì không mô tả được trạng thái xung đột này.
Common mistake: Vẽ 1 box to trùm cả 2 đầu đèn, hoặc nhầm màu xanh của mũi tên cho luồng đi thẳng.
Diversity: conflict

---

CASE ID: EC02
Sample: BDD11
Scene: Giao lộ ban ngày nhiều đầu đèn treo trên giàn gantry
Observation: Đầu đèn tín hiệu gắn trên giàn gantry có vỏ hộp nhìn rõ nhưng mất điện hoàn toàn không sáng bóng nào.
Decision: LABEL
Expected: Bounding box ôm khít đầu đèn, state=off, relevance=ego_relevant, occluded=false, needs_review=false.
Rationale: Xe tự hành cần nhận diện đầu đèn đang tắt để kích hoạt chế độ nhường đường an toàn khi qua ngã tư mất điện theo luật giao thông.
Common mistake: Bỏ qua không vẽ box vì nghĩ đèn tắt là không cần label.
Diversity: ambiguity

---

CASE ID: EC03
Sample: BDD18
Scene: Đường phố ban đêm ánh sáng phức tạp
Observation: Phía trên cao có các đốm sáng tròn màu vàng/cam phát ra từ các cột đèn cao áp chiếu sáng đô thị, không có cấu trúc hộp đèn chữ nhật.
Decision: IGNORE
Expected: Tuyệt đối không vẽ bounding box trên các bóng đèn đường này.
Rationale: Tránh tạo ra false positives khiến hệ thống nhận diện nhầm đèn vàng và phanh dừng xe vô cớ giữa đường trường.
Common mistake: Thấy đốm sáng tròn màu vàng lơ lửng là vẽ box và gán state=yellow.
Diversity: escalation

---

CASE ID: EC04
Sample: BDD26
Scene: Đường phố ban đêm có 2 giao lộ liên tiếp
Observation: Ở ngã tư gần xe đang tới có đèn xanh (`green`). Ở ngã tư phía sau cách 150m nhìn thấy rõ một đầu đèn khác đang đỏ (`red`).
Decision: LABEL
Expected: Box ngã tư gần: state=green, relevance=ego_relevant. Box ngã tư sau: state=red, relevance=other_lane.
Rationale: Đây là quy tắc cực kỳ quan trọng (Critical Rule). Đèn ngã tư sau không được phép điều khiển ngã tư trước. Nếu gán ego_relevant cho đèn sau, xe sẽ phanh gấp (phantom brake) giữa ngã tư trước.
Common mistake: Gán cả 2 đèn đều là ego_relevant hoặc chỉ label đèn ngã tư sau vì thấy nó màu đỏ.
Diversity: critical

---

CASE ID: EC05
Sample: BDD12
Scene: Cảnh đường phố ban ngày nhiều mây, tầm nhìn xa
Observation: Ở ngã tư xa xuất hiện một đầu đèn kích thước khoảng 10px, nhìn thấy rõ vỏ hộp nhưng mờ nhòe không thể đọc chắc chắn là màu vàng hay đỏ.
Decision: LABEL
Expected: Bounding box ôm khít đầu đèn, state=unknown, relevance=ego_relevant (hoặc unknown), needs_review=true.
Rationale: Giữ recall cho detector nhưng không ép annotator đoán mò trạng thái màu khi không đủ bằng chứng điểm ảnh.
Common mistake: Đoán mò màu theo cảm tính hoặc bỏ qua không vẽ box dù kích thước >= 8 px.
Diversity: small_far

---

CASE ID: EC06
Sample: BDD04
Scene: Phố đô thị ban ngày có cây xanh bên vỉa hè
Observation: Đầu đèn tín hiệu bên phải bị tán cây xanh che khuất mất khoảng 60% phần thân, chỉ lộ rõ bóng đèn đỏ đang sáng.
Decision: LABEL
Expected: Bounding box chữ nhật ước lượng toàn bộ khung đầu đèn (amodal box), state=red, relevance=ego_relevant, occluded=true.
Rationale: Báo hiệu cho mô hình biết vật thể đang bị che khuất một phần để hệ thống tracking không bị drop track.
Common mistake: Chỉ vẽ một box tí hon bao quanh bóng đèn tròn đỏ lộ ra, hoặc quên không tick occluded=true.
Diversity: occlusion

---

CASE ID: EC07
Sample: BDD17
Scene: Đường phố trời mưa, mặt đường ướt sũng
Observation: Ánh đèn đỏ của đèn giao thông phản chiếu một vệt dài rực rỡ trên mặt đường nhựa ướt và kính xe phía trước.
Decision: IGNORE
Expected: Không vẽ bounding box trên các vệt phản chiếu dưới mặt đường hay kính xe.
Rationale: Phản chiếu không phải là nguồn tín hiệu vật lý. Nếu vẽ box sẽ làm detector học sai đặc trưng nền.
Common mistake: Nhầm vệt phản chiếu là đèn giao thông phụ hoặc đèn tín hiệu thấp.
Diversity: escalation

---

CASE ID: EC08
Sample: BDD25
Scene: Giao lộ chạng vạng hoàng hôn phức tạp
Observation: Ngã năm với nhiều làn rẽ, đèn đi thẳng màu xanh trong khi đèn rẽ trái màu đỏ, xe tự hành đang nằm ở làn giữa có mũi tên hỗn hợp đi thẳng và rẽ trái.
Decision: LABEL
Expected: Vẽ tách biệt 2 box. Box đi thẳng: state=green, relevance=ego_relevant; Box rẽ trái: state=red, relevance=other_lane; tick needs_review=true.
Rationale: Ngữ cảnh đa làn hỗn hợp đòi hỏi ghi nhận bằng chứng đầy đủ và gắn cờ escalation cho chuyên gia an toàn đánh giá.
Common mistake: Nhầm lẫn giữa 2 đèn hoặc không gán cờ needs_review.
Diversity: critical

---

CASE ID: EC09
Sample: BDD14
Scene: Đường cao tốc ngoại ô ban ngày
Observation: Có biển báo chỉ dẫn hướng đi trên giàn treo lớn bắc ngang cao tốc nhưng hoàn toàn không có đèn giao thông.
Decision: IGNORE
Expected: Không tạo bất kỳ annotation nào.
Rationale: Mẫu kiểm tra âm tính (Negative sample) xác nhận hệ thống không bị nhầm lẫn giữa biển báo trên cao và đèn tín hiệu giao thông.
Common mistake: Gán nhầm biển báo hình chữ nhật thành đèn giao thông tắt (`state=off`).
Diversity: escalation

---
