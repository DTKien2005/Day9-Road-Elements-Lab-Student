# Annotation guideline — Traffic Light State & Ego Relevance

**Version:** v3

Guideline này là tài liệu kỹ thuật bắt buộc cho toàn bộ annotators và nhóm kiểm định chéo (peer test). Không áp dụng bất kỳ quy tắc nói miệng nào ngoài văn bản này (No hidden rules).

---

## 1. Objective + scope

- **Mục tiêu:** Cung cấp dữ liệu ground truth huấn luyện hệ thống Traffic Light Recognition (TLR) và Motion Planning cho xe tự hành (AV cấp độ 4). Hệ thống cần biết chính xác: *Có đèn không? Ở đâu? Màu gì? Và đèn đó có điều khiển làn xe của mình hay không?*
- **Đối tượng trong scope:**
  - Mọi đầu đèn tín hiệu giao thông đường bộ độc lập (`signal head`), bao gồm đèn tròn, đèn mũi tên, đèn kiểm soát làn, đèn người đi bộ và đèn đang tắt.
  - Áp dụng cho cả giao lộ gần (near intersection) và giao lộ kế tiếp phía sau (far intersection).
- **Đối tượng ngoài scope (IGNORE):**
  - Đèn chiếu sáng đường phố (đèn đường đơn độc không có hộp tín hiệu), đèn hậu ô tô, đèn phanh, đèn biển quảng cáo/neon.
  - Vết phản chiếu của đèn giao thông trên kính xe buýt, mặt kính tòa nhà hoặc mặt đường ướt.
  - Cột đèn (pole), thanh giàn ngang (gantry) và các biển báo gắn kèm.
  - Đèn tín hiệu cực nhỏ ở quá xa (kích thước cạnh lớn nhất < 8 px).

---

## 2. Annotation unit

- **Đơn vị gán nhãn:** Mỗi **đầu đèn tín hiệu vật lý riêng lẻ (`signal head`)** là một instance độc lập tương ứng với **một Bounding Box (`rectangle`)**.
- **Quy tắc cụm nhiều đầu đèn (Multi-head cluster / Gantry):**
  - Nếu trên cùng một giá treo hoặc cùng một cột có nhiều đầu đèn cạnh nhau (ví dụ: 1 đầu đèn đi thẳng, 1 đầu đèn rẽ trái, 1 đầu đèn phụ mũi tên) → **VẼ CÁC BOX TÁCH BIỆT** cho từng đầu đèn.
  - Tuyệt đối không vẽ một box to gom toàn bộ cụm đèn hoặc gom nhiều đầu đèn vào một hộp duy nhất.

---

## 3. Geometry rule

- **Dạng hình học:** `rectangle` (hộp chữ nhật đứng hoặc ngang).
- **Độ khít (Tightness):** Box phải ôm sát mép ngoài của vỏ đầu đèn (housing/visor). Không chừa khoảng trống thừa xung quanh và không cắt lẹm vào phần phát sáng của bóng đèn.
- **Không bao gồm cột:** Box chỉ bao trùm đầu đèn, mép box dừng lại ở chỗ tiếp giáp với tay đỡ kim loại hoặc cột gantry.
- **Dung sai (Tolerance):** Độ lệch cho phép <= 3 px mỗi cạnh trên độ phân giải 1280 x 720. Lệch > 5 px bị tính là lỗi Major.

---

## 4. Taxonomy & CVAT Attributes

Mỗi đối tượng `traffic_light` bắt buộc phải được gán đầy đủ 4 thuộc tính (không được để lại giá trị mặc định `__undefined__`):

| Thuộc tính | Kiểu nhập | Giá trị hợp lệ | Mặc định | Ý nghĩa & Hướng dẫn chọn |
| :--- | :--- | :--- | :--- | :--- |
| **`state`** | select | `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | Màu đèn đang sáng rõ ràng. Dùng `off` nếu đèn tắt. Dùng `unknown` nếu mờ/lóa/che khuất không xác định được. |
| **`relevance`** | select | `ego_relevant`, `other_lane`, `pedestrian`, `unknown` | `__undefined__` | Thẩm quyền điều khiển đối với quỹ đạo xe tự hành (ego vehicle). |
| **`occluded`** | checkbox | `true`, `false` | `false` | Đánh dấu `true` khi đầu đèn bị che khuất từ 50% diện tích trở lên bởi vật cản. |
| **`needs_review`** | checkbox | `true`, `false` | `false` | Đánh dấu `true` khi gặp ca biên nghi vấn cần QA Lead hoặc Safety Engineer duyệt. |

---

## 5. Inclusion / Exclusion & Edge Cases chuyên sâu

### 5.1 Khi đèn hiện đồng thời 2 hoặc 3 tín hiệu (Multi-signal / Transition / Arrow)
- **Trường hợp A: Nhiều tín hiệu trên các đầu đèn (heads) riêng biệt**
  - *Hiện tượng*: Một đầu đèn tròn màu đỏ và bên cạnh có một đầu đèn phụ hình mũi tên màu xanh rẽ phải.
  - *Cách xử lý*: Vẽ **2 box độc lập**:
    - Box 1 (đèn tròn): `state=red`, `relevance=ego_relevant` (nếu xe đi thẳng) hoặc `other_lane` (nếu xe rẽ).
    - Box 2 (mũi tên rẽ): `state=green`, `relevance=ego_relevant` (nếu xe chuẩn bị rẽ phải) hoặc `other_lane` (nếu xe đi thẳng).
- **Trường hợp B: Cùng 1 đầu đèn nhưng sáng đồng thời 2 bóng (Transition state Red + Yellow)**
  - *Hiện tượng*: Chuẩn tín hiệu Châu Âu/Đức chuyển từ Đỏ sang Xanh sẽ bật đồng thời Red + Yellow trong khoảng 1 giây.
  - *Cách xử lý*: Nguyên tắc an toàn cao nhất (**Safety First**) — khi đèn còn tín hiệu đỏ, xe chưa được phép di chuyển:
    - Gán `state=yellow` (nếu theo chuẩn nhận diện chuyển trạng thái của hệ thống), hoặc nếu nghi ngờ lỗi phần cứng chập bóng → gán `state=red`, tick `needs_review=true`.
- **Trường hợp C: Chập điện sáng cả 3 bóng cùng lúc**
  - *Cách xử lý*: Gán `state=unknown`, tick `needs_review=true`.

### 5.2 Đèn không sáng bóng nào (Unlit / Off light)
- *Hiện tượng*: Ban ngày nhìn rõ vỏ hộp đèn màu đen/vàng nhưng không có bóng nào bật sáng, hoặc ban đêm đèn ngã tư bị mất điện tắt tối om.
- *Cách xử lý*:
  - **BẮT BUỘC vẽ box** ôm sát khung đầu đèn.
  - Gán `state=off`.
  - Gán `relevance=ego_relevant` nếu đèn nằm thẳng trên làn xe, hoặc `other_lane` nếu nằm trên làn rẽ. Nếu không rõ cấu trúc làn, gán `relevance=unknown`.
  - *Ý nghĩa*: Giúp xe nhận diện ngã tư mất điện để tự động giảm tốc độ và nhường đường theo luật.

### 5.3 Đèn ở xa dễ nhầm với đèn đường (Street light) hoặc đèn hậu (Tail light)
- *Quy tắc bằng chứng hình thể (Visual Structure Evidence)*:
  - Chỉ label khi nhìn thấy **cấu trúc hộp đèn tín hiệu** (vỏ hộp chữ nhật, mào che visor hoặc thanh treo gantry).
  - Một đốm sáng tròn vàng/cam lơ lửng trên cao từ cột chiếu sáng cao áp → **IGNORE (KHÔNG LABEL)**.
  - Đốm sáng đỏ nằm ngang tầm mắt trên đuôi xe tải/xe con → **IGNORE** (đèn xe).
  - Nếu ở xa chỉ thấy đốm sáng nghi ngờ:
    - Kích thước < 8 px: **IGNORE**.
    - Kích thước >= 8 px nhưng không chắc chắn: Vẽ box, gán `state=unknown`, `relevance=unknown`, tick `needs_review=true`.

### 5.4 Đèn ở xa bị mờ / nhòe chuyển động (Small / Far / Blur)
- **Kích thước < 8 px (cạnh dài nhất):** **IGNORE** (không vẽ box).
- **Kích thước từ 8 px đến dưới 15 px:**
  - Nhìn rõ hình dáng đầu đèn nhưng quá mờ không đọc chắc màu đèn: Vẽ box, gán `state=unknown`.
  - Nếu xe đang đi thẳng trên làn thẳng tắp hướng về ngã tư đó: Gán `relevance=ego_relevant` (hoặc `unknown` nếu còn cách quá xa).
- **Kích thước >= 15 px:** Bắt buộc phải đọc và phân loại chính xác `red`, `yellow`, `green` hoặc `off`.

### 5.5 Hai đèn xuất hiện: Đèn trước hiện Xanh, Đèn ngã tư sau hiện Đỏ (Near vs Far Intersection)
- *Hiện tượng*: Xe đang tiếp cận ngã tư gần (thấy đèn xanh), nhưng qua ngã tư đó nhìn xa thấy ngã tư kế tiếp đang có đèn đỏ.
- *Quy tắc phân định thẩm quyền (DTLD Standard)*:
  - **Đèn ở ngã tư gần (Foreground / Near intersection):** Là đèn đang điều khiển hành vi hiện tại của xe → Label với `relevance=ego_relevant`, `state=green`.
  - **Đèn ở ngã tư sau (Background / Far intersection):** Dù cùng hướng nhìn thẳng nhưng thuộc nút giao tiếp theo → Vẫn vẽ box (nếu >= 8 px), gán `state=red`, nhưng **BẮT BUỘC gán `relevance=other_lane`** (coi như không áp dụng cho hành vi tức thời của xe).
  - *Cảnh báo nguy hiểm*: Nếu gán đèn ngã tư sau là `ego_relevant`, xe tự hành sẽ phanh gấp giữa ngã tư trước (phantom braking), gây tai nạn dồn toa từ xe phía sau!

### 5.6 Đèn ban đêm nhìn từ xa không thấy vỏ hộp đèn (Invisible Housing / Nighttime Visible Lamp)
- *Hiện tượng (điển hình như ảnh BDD18)*: Ban đêm nền trời tối đen, vỏ hộp đèn (housing) màu đen và mào che visor hoàn toàn chìm vào bóng tối. Mắt người và camera chỉ nhìn thấy một đốm sáng tròn phát quang màu xanh (hoặc đỏ), không thể thấy khung viền hộp đèn ở đâu.
- *Quy tắc gán nhãn (Visible Lamp Rule theo slide mục 69)*:
  - Khi vỏ hộp đèn hoàn toàn tàng hình trong bóng tối, **vẽ bounding box ôm chặt lấy vùng bóng đèn phát sáng (visible lamp / core illuminated lens)**.
  - **Dung sai:** Chỉ lấy phần lõi sáng tròn có màu rõ ràng (thường từ 8 px đến 12 px), **tuyệt đối không vẽ lan rộng ra ngoài quầng sáng chói (glare / halo)** và không cần phải "đoán mò" kích thước hộp đen tàng hình.
  - **Phân biệt nguồn sáng ban đêm:**
    - **Đèn giao thông thật:** Đốm sáng màu xanh ngọc (traffic green) treo trên cao giữa làn đường (như ở BDD18: tọa độ x khoảng 585 và 662, y khoảng 307) → Vẽ box, gán `state=green`, `relevance=ego_relevant`.
    - **Đèn xe phía trước (Tail lights):** Toàn bộ các đốm sáng màu đỏ ở tầm thấp (ngang đuôi ô tô ở BDD18) → **IGNORE** (không vẽ box).
    - **Đèn chiếu sáng đô thị (Street lamps):** Đốm sáng vàng/cam trên cột cao uốn cong → **IGNORE** (không vẽ box).

---

## 6. Visibility & Occlusion

- **Bị che khuất (Occluded):** Nếu đầu đèn bị che từ 50% đến 80% diện tích bởi cành cây, xe tải phía trước hoặc biển báo → Vẽ box ước lượng toàn bộ đầu đèn (amodal bounding box phần đầu đèn) và tick `occluded=true`.
- **Bị che > 80% hoặc chỉ hở một đốm sáng nhỏ không thấy vỏ đèn:** **IGNORE** (không đủ chứng cứ hình học).
- **Cắt mép ảnh (Truncation):** Nếu đầu đèn bị cắt ngang mép ảnh nhưng phần còn lại >= 50%, vẽ box ôm phần nhìn thấy và tick `occluded=true`.

---

## 7. Ambiguity & Escalation Decision Tree

Quy trình 5 bước xác định Relevance khi gặp tình huống mơ hồ:
1. **Bước 1:** Đèn có nằm trong tầm nhìn phía trước và nhìn thấy cấu trúc đầu đèn (>= 8 px) không? Nếu không → IGNORE.
2. **Bước 2:** Đèn gắn ở đâu?
   - Treo trực tiếp trên giá long môn ngay trên làn xe của mình → Hướng tới `ego_relevant`.
   - Treo trên cột bên trái/phải có kèm biển phụ hoặc mũi tên rẽ → Hướng tới `other_lane`.
   - Có hình người đi bộ / xe đạp → Gán `relevance=pedestrian`.
3. **Bước 3:** Quỹ đạo của xe (ego lane) đang đi thẳng hay rẽ? Đối chiếu hướng đi của làn với tín hiệu đèn.
4. **Bước 4:** Đèn thuộc ngã tư hiện tại hay ngã tư kế tiếp? Nếu ngã tư kế tiếp → `other_lane`.
5. **Bước 5 (Escalation):** Nếu sau 4 bước vẫn không thể xác định do giao lộ chéo ngã năm hoặc chói lóa → Đặt `relevance=unknown`, tick `needs_review=true`. Nếu toàn bộ frame bị chói lóa mù sương không thấy đường → Gán tag `image_escalate`.

---

## 8. Temporal rule (Cho video / sequence LISA)

- **Track Mode:** Với chuỗi ảnh video, dùng công cụ **Track** để cùng một đầu đèn duy trì một `Track ID` duy nhất suốt hành trình tiếp cận.
- **Tính nhất quán của Relevance:** `relevance` của cùng một đầu đèn không được phép thay đổi nhảy cóc qua các frame liên tiếp (ví dụ: không thể frame trước là `ego_relevant`, frame sau lại thành `other_lane`).
- **Chuyển trạng thái màu (State Transition):** Trạng thái `state` là mutable, phải tuân theo chu trình đèn giao thông thực tế: Green → Yellow → Red hoặc Red → Green. Nếu thấy bước nhảy bất thường (ví dụ: Green nhảy thẳng sang Red chỉ trong 1 frame), phải kiểm tra lại xem có bị lóa sáng hay gán nhầm sang đầu đèn khác không.
- **Tạm thời bị che (Intermittent Occlusion):** Nếu đèn bị xe tải che mất trong 2–3 frame, không được tự ý "bịa" màu nếu hoàn toàn không nhìn thấy ánh sáng; hãy ẩn track hoặc gán `state=unknown`.

---

## 9. Examples

| sample_id | Vị trí / Đối tượng quan sát | Expected Output | Quy tắc áp dụng |
|---|---|---|---|
| `BDD02` | Đầu đèn treo giữa làn đi thẳng | `traffic_light`, box khít đầu đèn, `state=green`, `relevance=ego_relevant`, `occluded=false` | Đèn chuẩn ban ngày làn đi thẳng |
| `BDD04` | Đèn bên phải bị cành cây che một nửa | `traffic_light`, `state=red`, `relevance=ego_relevant`, `occluded=true` | Quy tắc che khuất >= 50% |
| `BDD10` | Cột đèn có 2 đầu đèn: 1 đèn tròn, 1 đèn mũi tên rẽ trái | Vẽ 2 box riêng: Box tròn (`red`, `ego_relevant`), Box rẽ (`green`, `other_lane`) | Quy tắc Multi-head cluster |
| `LISA01` | Đèn giao lộ đang tiếp cận, frame đầu tiên | Track `traffic_light`, `state=green`, `relevance=ego_relevant` | Quy tắc chuỗi video tiếp cận |

---

## 10. Common mistakes (Lỗi thường gặp cần tránh)

1. **Lấy cả cột đèn vào box:** Vẽ một box kéo dài từ đầu đèn xuống tận chân cột hoặc lấy cả giá treo kim loại → **Sai hoàn toàn geometry**. Chỉ vẽ quanh hộp đầu đèn.
2. **Gán nhầm đèn rẽ thành ego_relevant:** Xe đi thẳng nhưng lại gán đèn đỏ rẽ trái thành `ego_relevant` → **Lỗi chí mạng (Critical error)** gây phanh oan.
3. **Nhầm đèn đường / đèn hậu xe là đèn giao thông:** Ban đêm thấy đốm sáng đỏ là vẽ box → Phải tìm cấu trúc hộp đèn; đốm sáng đơn lẻ phải bỏ qua.
4. **Nhầm đèn của ngã tư phía sau:** Gán `ego_relevant` cho đèn đỏ ở cách đó 150m phía sau ngã tư xanh hiện tại → Gây lỗi hành vi xe tự hành.
5. **Để sót giá trị `__undefined__`:** Quên không chọn dropdown state/relevance sau khi vẽ hình chữ nhật.
