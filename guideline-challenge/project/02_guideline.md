# HƯỚNG DẪN GÁN NHÃN VÀ QUY TRÌNH CVAT — TRAFFIC LIGHT STATE & RELEVANCE
**Chương trình:** VinUni AI20K — Road Elements Lab (Day 9)  
**Nhóm:** TrafficVision-AI — **Lead: ĐỖ TUẤN KIÊN**  
**Version:** v3  
**Phạm vi áp dụng:** Dữ liệu Thử thách Day 9 (Road Elements - Traffic Light State & Ego Relevance):
1. **Calibration Task:** [CVAT Task 26](http://localhost:8080/tasks/26) (6 ảnh BDD100K + LISA: `team-traffic-light-calibration`)
2. **Golden / Blind Task:** [CVAT Task 27](http://localhost:8080/tasks/27) (5 ảnh BDD100K: `team-traffic-light-golden-blind`)

**Tài liệu tham chiếu gốc:**
- `traffic_light_annotation_guide.md` (DriveU Traffic Light Dataset - Chuẩn DTLD & Slide bài giảng VinUni)
- `Week-Work/Week2/GUIDELINE.md` (VinUni AI20K Guideline Standard)
- Quy chế vận hành: Road Elements Guideline Design Challenge Playbook (Rubric 100 điểm)

---

## MỤC LỤC
1. [Nguyên tắc Cốt lõi & Ba Cảnh Báo Sống Còn](#1-nguyên-tắc-cốt-lõi--ba-cảnh-báo-sống-còn)
2. [Phạm vi Dự án (Objective & Scope)](#2-phạm-vi-dự-án-objective--scope)
3. [Đơn vị Gán nhãn (Annotation Unit) & Quy tắc Cụm Đầu Đèn](#3-đơn-vị-gán-nhãn-annotation-unit--quy-tắc-cụm-đầu-đèn)
4. [Quy chuẩn Hình học (Geometry Rule & Dung sai)](#4-quy-chuẩn-hình-học-geometry-rule--dung-sai)
5. [Hệ thống Phân loại (Taxonomy) & Thuộc tính CVAT](#5-hệ-thống-phân-loại-taxonomy--thuộc-tính-cvat)
6. [Quy tắc Bao hàm & Loại trừ (Inclusion / Exclusion)](#6-quy-tắc-bao-hàm--loại-trừ-inclusion--exclusion)
7. [Quy tắc Tầm nhìn & Che khuất (Visibility & Occlusion)](#7-quy-tắc-tầm-nhìn--che-khuất-visibility--occlusion)
8. [Cây Quyết định Xử lý Mơ hồ (Ambiguity & Escalation Decision Tree)](#8-cây-quyết-định-xử-lý-mơ-hồ-ambiguity--escalation-decision-tree)
9. [Quy tắc Nhất quán Thời gian (Temporal Consistency cho Video)](#9-quy-tắc-nhất-quán-thời-gian-temporal-consistency-cho-video)
10. [Bảng Xử lý Các Tình huống Đặc biệt & Edge Cases](#10-bảng-xử-lý-các-tình-huống-đặc-biệt--edge-cases)
11. [Danh mục Mẫu Điển hình (Representative Examples)](#11-danh-mục-mẫu-điển-hình-representative-examples)
12. [Quy trình Thao tác Kỹ thuật trên CVAT (SOP)](#12-quy-trình-thao-tác-kỹ-thuật-trên-cvat-sop)
13. [Quy trình Thẩm định (Review SOP) & Checklist Nghiệm thu của Lead](#13-quy-trình-thẩm-định-review-sop--checklist-nghiệm-thu-của-lead)
14. [Quy chế Vận hành: Clarification Log & Sổ Quyết định](#14-quy-chế-vận-hành-clarification-log--sổ-quyết-định)

---

## 1. NGUYÊN TẮC CỐT LÕI & BA CẢNH BÁO SỐNG CÒN

> [!CAUTION]
> **BA CẢNH BÁO CỰC KỲ QUAN TRỌNG TỪ CHUẨN AN TOÀN XE TỰ HÀNH**:
> 1. **KHÔNG BAO GIỜ GÁN NHẦM ĐÈN ĐỎ CỦA EGO LANE THÀNH XANH**: Đây là lỗi chí mạng (Critical Failure) có thể dẫn tới va chạm trực diện khi xe tự hành lao vào giao lộ. Tương tự, không được nhầm đèn xanh của làn rẽ (`other_lane`) thành đèn đi thẳng (`ego_relevant`).
> 2. **CẤM GÁN EGO_RELEVANT CHO ĐÈN NGÃ TƯ KẾ TIẾP PHÍA SAU**: Đèn ở ngã tư sau (cách 100 - 150m) dù cùng hướng nhìn thẳng nhưng thuộc giao lộ tiếp theo, **BẮT BUỘC gán `relevance=other_lane`**. Gán `ego_relevant` sẽ kích hoạt phanh ma (Phantom Braking) làm xe dừng khựng nguy hiểm giữa ngã tư trước!
> 3. **DEFAULT `__undefined__` BẮT BUỘC PHẢI CHỌN LẠI**: Schema CVAT cố ý để giá trị mặc định là `__undefined__`. Nếu file export còn sót lại giá trị này, công cụ QA sẽ tự động đánh fail bài nộp vì annotator chưa hoàn thành thao tác.

### Nguyên tắc chung:
- **Prediction không phải Ground Truth**: Mô hình tự động hay nhãn pre-label chỉ để tham khảo. Annotator phải dùng mắt và tư duy ngữ cảnh để xác định đúng thực tế.
- **Không đoán mò**: Khi điểm ảnh quá mờ nhòe (< 8 px) hoặc lóa chói không phân biệt được màu $\rightarrow$ đặt `state=unknown`, `relevance=unknown` và tick `needs_review=true`.
- **Zoom tối thiểu 300% - 400%**: Khi vẽ bounding box quanh đầu đèn, bắt buộc phải phóng to để box ôm sát mép vỏ hộp đèn, không lấy cột và không chừa viền thừa.

---

## 2. PHẠM VI DỰ ÁN (OBJECTIVE & SCOPE)

- **Mục tiêu kỹ thuật:** Cung cấp dữ liệu ground truth huấn luyện hệ thống Traffic Light Recognition (TLR) và AV Motion Planner (xe tự hành cấp độ 4). Hệ thống cần trả lời chính xác: *Có đèn tín hiệu không? Vị trí ở đâu? Đang sáng màu gì? Và có áp dụng cho làn xe mình đang chạy hay không?*
- **Đối tượng trong phạm vi (In-Scope - BẮT BUỘC LABEL):**
  - Mọi đầu đèn tín hiệu giao thông đường bộ độc lập nhìn thấy được (kích thước cạnh lớn nhất >= 8 px).
  - Bao gồm: đèn tròn tiêu chuẩn, đèn mũi tên chỉ hướng, đèn kiểm soát làn, đèn người đi bộ, và đèn đang tắt (`off`).
  - Giao lộ hiện tại (gần) và các giao lộ tiếp theo nhìn thấy trong tầm nhìn (xa).
- **Đối tượng ngoài phạm vi (Out-of-Scope - IGNORE - CẤM VẼ BOX):**
  - Đèn chiếu sáng đô thị (đèn đường cao áp đơn độc không có hộp tín hiệu giao thông).
  - Đèn hậu xe hơi (tail lights), đèn phanh, đèn xi-nhan xe cộ phía trước.
  - Hình phản chiếu của đèn giao thông trên kính xe buýt, mặt kính tòa nhà hoặc vũng nước mặt đường ướt.
  - Cột trụ (pole), khung giàn ngang (gantry) và các biển báo gắn kèm.
  - Đèn tín hiệu cực nhỏ ở quá xa (kích thước < 8 px).

---

## 3. ĐƠN VỊ GÁN NHÃN (ANNOTATION UNIT) & QUY TẮC CỤM ĐẦU ĐÈN

- **Đơn vị gán nhãn:** Mỗi **đầu đèn tín hiệu vật lý riêng lẻ (`signal head`)** là một instance độc lập tương ứng với **một Bounding Box (`rectangle`)**.
- **Quy tắc cụm nhiều đầu đèn (Multi-head cluster / Gantry):**
  - Trên cùng một giá treo hoặc cùng một cột nếu có nhiều đầu đèn cạnh nhau (ví dụ: 1 đầu đèn đi thẳng, 1 đầu đèn rẽ trái, 1 đầu đèn phụ mũi tên) $\rightarrow$ **VẼ CÁC BOX TÁCH BIỆT** cho từng đầu đèn.
  - Tuyệt đối không vẽ một box to gom toàn bộ cụm đèn hoặc gom nhiều đầu đèn vào một hộp duy nhất.

---

## 4. QUY CHUẨN HÌNH HỌC (GEOMETRY RULE & DUNG SAI)

- **Dạng hình học:** `rectangle` (hộp chữ nhật đứng hoặc ngang).
- **Độ ôm khít (Tightness):** Box phải ôm sát mép ngoài của vỏ hộp đầu đèn (housing/hood/visor). Không chừa khoảng trống thừa xung quanh và không cắt lẹm vào phần phát sáng của bóng đèn.
- **Không bao gồm cột:** Box chỉ bao trùm đầu đèn, mép box dừng lại ở chỗ tiếp giáp với tay đỡ kim loại hoặc cột gantry.
- **Dung sai hình học (Tolerance):**
  - Độ lệch cho phép: <= 3 px mỗi cạnh trên độ phân giải chuẩn 1280 x 720.
  - Độ lệch từ 3 - 5 px: tính lỗi Minor.
  - Độ lệch > 5 px hoặc vẽ bao cả cột đèn: tính lỗi Major.

---

## 5. HỆ THỐNG PHÂN LOẠI (TAXONOMY) & THUỘC TÍNH CVAT

Mỗi đối tượng `traffic_light` bắt buộc phải được gán đầy đủ 4 thuộc tính (không được để lại giá trị mặc định `__undefined__`):

| Thuộc tính | Kiểu nhập | Giá trị hợp lệ | Mặc định | Ý nghĩa & Hướng dẫn chọn |
| :--- | :---: | :--- | :---: | :--- |
| **`state`** | select | `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | Màu đèn đang sáng rõ ràng. Dùng `off` nếu đèn tắt. Dùng `unknown` nếu mờ/lóa/che khuất không xác định được. |
| **`relevance`** | select | `ego_relevant`, `other_lane`, `pedestrian`, `unknown` | `__undefined__` | Thẩm quyền điều khiển đối với quỹ đạo xe tự hành (ego vehicle). |
| **`occluded`** | checkbox | `true`, `false` | `false` | Đánh dấu `true` khi đầu đèn bị che khuất từ 50% diện tích trở lên bởi vật cản. |
| **`needs_review`** | checkbox | `true`, `false` | `false` | Đánh dấu `true` khi gặp ca biên nghi vấn cần QA Lead duyệt. |

- **Tag mức ảnh:** `image_escalate` gán cho toàn bộ ảnh khi frame bị hư hỏng, lóa trắng toàn màn hình hoặc không thể phân định ngữ cảnh giao thông.

---

## 6. QUY TẮC BAO HÀM & LOẠI TRỪ (INCLUSION / EXCLUSION)

- **Bao hàm (Inclusion):**
  - Đèn giao thông đang sáng màu rõ ràng.
  - Đèn giao thông ban ngày hoặc ban đêm đang tắt nguồn (`state=off`).
  - Đèn giao thông bị che khuất một phần (50% - 80%) nhưng vẫn nhận ra cấu trúc.
  - Đèn giao thông ở nút giao tiếp theo phía sau (gán `relevance=other_lane`).
- **Loại trừ (Exclusion):**
  - Đèn đường cao áp chiếu sáng đô thị.
  - Đèn hậu xe hơi, đèn phanh xe ô tô/xe buýt phía trước.
  - Vệt phản chiếu ánh sáng trên mặt đường ướt hoặc cửa kính.
  - Khung giàn kim loại, cột trụ, biển tên đường.

---

## 7. QUY TẮC TẦM NHÌN & CHE KHUẤT (VISIBILITY & OCCLUSION)

- **Che khuất một phần (Occluded):** Nếu đầu đèn bị che từ 50% đến 80% diện tích bởi cành cây, xe tải phía trước hoặc biển báo $\rightarrow$ Vẽ box ước lượng toàn bộ đầu đèn (amodal box) và tick `occluded=true`.
- **Che khuất nặng (> 80%):** Nếu chỉ hở một đốm sáng nhỏ li ti không còn nhìn thấy cấu trúc vỏ hộp $\rightarrow$ **IGNORE** (không đủ chứng cứ).
- **Cắt mép ảnh (Truncation):** Nếu đầu đèn bị cắt ngang mép ảnh nhưng phần còn lại >= 50%, vẽ box ôm phần nhìn thấy và tick `occluded=true`.

---

## 8. CÂY QUYẾT ĐỊNH XỬ LÝ MƠ HỒ (AMBIGUITY & ESCALATION DECISION TREE)

Khi gặp tình huống phức tạp nhiều luồng giao thông, thực hiện đúng **Quy trình 5 bước**:
1. **Bước 1:** Đèn có nằm trong tầm nhìn phía trước và nhìn thấy cấu trúc đầu đèn (>= 8 px) không? Nếu không $\rightarrow$ **IGNORE**.
2. **Bước 2:** Đèn gắn ở đâu?
   - Treo trực tiếp trên giá long môn ngay trên làn xe của mình $\rightarrow$ Hướng tới `ego_relevant`.
   - Treo trên cột bên trái/phải có kèm biển phụ hoặc mũi tên rẽ $\rightarrow$ Hướng tới `other_lane`.
   - Có hình người đi bộ / xe đạp $\rightarrow$ Gán `relevance=pedestrian`.
3. **Bước 3:** Quỹ đạo của xe (ego lane) đang đi thẳng hay rẽ? Đối chiếu hướng đi của làn với tín hiệu đèn.
4. **Bước 4:** Đèn thuộc ngã tư hiện tại hay ngã tư kế tiếp? Nếu ngã tư kế tiếp $\rightarrow$ `other_lane`.
5. **Bước 5 (Escalation):** Nếu sau 4 bước vẫn không thể xác định do giao lộ chéo ngã năm hoặc chói lóa $\rightarrow$ Đặt `state=unknown`, `relevance=unknown`, tick `needs_review=true`. Nếu toàn bộ frame bị chói lóa mù sương không thấy đường $\rightarrow$ Gán tag `image_escalate`.

---

## 9. QUY TẮC NHẤT QUÁN THỜI GIAN (TEMPORAL CONSISTENCY CHO VIDEO)

- **Track Mode:** Với chuỗi video liên tiếp (như clip LISA), sử dụng công cụ **Track** để cùng một đầu đèn vật lý duy trì duy nhất một `Track ID` suốt hành trình tiếp cận.
- **Tính nhất quán của Relevance:** `relevance` của cùng một đầu đèn không được phép thay đổi nhảy cóc qua các frame liên tiếp (ví dụ: không thể frame trước là `ego_relevant`, frame sau lại thành `other_lane`).
- **Chuyển trạng thái màu (State Transition):** Trạng thái `state` là mutable, phải tuân theo chu trình đèn giao thông thực tế: Green $\rightarrow$ Yellow $\rightarrow$ Red hoặc Red $\rightarrow$ Green. Bước nhảy bất thường (như Green nhảy thẳng sang Red trong 1 frame) là dấu hiệu lỗi cần kiểm tra lại.
- **Tạm thời bị che (Intermittent Occlusion):** Nếu đèn bị xe tải che mất trong 2–3 frame, không được tự ý "bịa" màu nếu hoàn toàn không nhìn thấy ánh sáng; hãy ẩn track hoặc gán `state=unknown`.

---

## 10. BẢNG XỬ LÝ CÁC TÌNH HUỐNG ĐẶC BIỆT & EDGE CASES

| Tình huống | Hiện tượng & Rủi ro | Quy tắc xử lý chuẩn |
|---|---|---|
| **Đèn hiện 2-3 tín hiệu (Đầu đèn rời)** | Cột có 1 đầu đèn tròn đỏ và 1 đầu đèn mũi tên xanh rẽ phải | • **Vẽ 2 bounding box riêng biệt**.<br>• Box tròn: `state=red`, `relevance=ego_relevant` (nếu đi thẳng) hoặc `other_lane`.<br>• Box mũi tên: `state=green`, `relevance=other_lane` (nếu đi thẳng) hoặc `ego_relevant` (nếu rẽ). |
| **Đèn chuyển trạng thái (Red + Yellow)** | Cùng 1 đầu đèn sáng đồng thời 2 bóng Red + Yellow trong 1 giây | • Áp dụng nguyên tắc **Safety First** (xe chưa được đi).<br>• Gán `state=yellow` (nếu có chuẩn chuyển trạng thái), hoặc gán `state=red` và tick `needs_review=true`. |
| **Đèn không sáng bóng nào (Unlit / Off)** | Ban ngày thấy rõ vỏ hộp nhưng không bóng nào sáng; hoặc ban đêm mất điện | • **BẮT BUỘC vẽ box** ôm sát khung đầu đèn.<br>• Gán `state=off`.<br>• `relevance=ego_relevant` nếu nằm trên làn xe mình, hoặc `other_lane`. Giúp xe biết ngã tư mất điện để giảm tốc độ. |
| **Đèn ban đêm không thấy vỏ hộp (Invisible Housing - BDD18)** | Ban đêm nền trời đen đặc, chỉ thấy bóng đèn xanh/đỏ phát sáng, không thấy viền hộp đen | • Áp dụng **Visible Lamp Rule (slide mục 69)**.<br>• **Vẽ box ôm chặt lấy vùng bóng đèn phát sáng thực tế** (lõi sáng tròn, kích thước 8x8 đến 10x10 px).<br>• **Không vẽ lan ra quầng sáng chói (glare/halo)** và không đoán mò vỏ hộp đen.<br>• Gán `state=green` (hoặc `red`), `relevance=ego_relevant`. |
| **Đèn đường chiếu sáng đô thị** | Đốm sáng tròn màu vàng/cam lơ lửng trên cao từ cột kim loại uốn cong | • **IGNORE (Tuyệt đối không vẽ box)**.<br>• Chỉ vẽ khi nhìn thấy cấu trúc hộp đèn tín hiệu chữ nhật có mào che. Đốm sáng cao áp đơn lẻ phải bỏ qua. |
| **Đèn hậu xe hơi phía trước (Tail lights)** | Đốm sáng màu đỏ ở tầm thấp ngang đuôi xe ô tô | • **IGNORE (Tuyệt đối không vẽ box)**.<br>• Đây là đèn xe, không phải đèn điều khiển giao thông đường bộ. |
| **Đèn ở xa bị mờ / nhòe chuyển động** | Đèn ở ngã tư xa kích thước 8 px - 15 px, mờ nhòe không rõ màu | • Kích thước < 8 px: **IGNORE**.<br>• Kích thước 8 px đến dưới 15 px: Vẽ box ôm đầu đèn, gán `state=unknown`, `relevance=unknown` (hoặc `ego_relevant` nếu đúng tim đường), tick `needs_review=true`.<br>• Kích thước >= 15 px: Bắt buộc đọc đúng màu. |
| **Hai đèn: Đèn trước Xanh, Đèn ngã tư sau Đỏ (BDD26)** | Xe tới ngã tư gần thấy đèn xanh, nhìn xuyên qua thấy ngã tư xa cách 150m đang đỏ | • **Đèn ngã tư gần:** `state=green`, `relevance=ego_relevant`.<br>• **Đèn ngã tư sau:** `state=red`, **BẮT BUỘC gán `relevance=other_lane`**.<br>• Tuyệt đối không gán đèn sau là ego_relevant để tránh lỗi xe phanh gấp giữa ngã tư trước! |
| **Mặt đường ướt trời mưa (BDD17)** | Vệt sáng đỏ/xanh phản chiếu rực rỡ dưới mặt đường nhựa ướt hoặc kính xe | • **IGNORE vệt phản chiếu**.<br>• Chỉ vẽ box quanh đầu đèn vật lý thật treo trên cao. |
| **Đầu đèn bị cành cây che (BDD04)** | Cành cây che mất khoảng 50% - 60% thân đầu đèn | • Vẽ box ước lượng toàn bộ đầu đèn (amodal box).<br>• Gán đúng màu đèn đang sáng, `relevance=ego_relevant` và **tick chọn `occluded=true`**. |

---

## 11. DANH MỤC MẪU ĐIỂN HÌNH (REPRESENTATIVE EXAMPLES)

| sample_id | Vị trí / Đối tượng quan sát | Expected Output | Quy tắc áp dụng |
|---|---|---|---|
| `BDD02` | Đầu đèn treo giữa làn đi thẳng | `traffic_light`, box khít đầu đèn, `state=green`, `relevance=ego_relevant`, `occluded=false` | Đèn chuẩn ban ngày làn đi thẳng |
| `BDD04` | Đèn bên phải bị cành cây che một nửa | `traffic_light`, `state=red`, `relevance=ego_relevant`, `occluded=true` | Quy tắc che khuất >= 50% |
| `BDD10` | Cột có 2 đầu đèn: 1 đèn tròn, 1 mũi tên rẽ | Vẽ 2 box riêng: Box tròn (`red`, `ego_relevant`), Box rẽ (`green`, `other_lane`) | Quy tắc Multi-head cluster |
| `LISA01` | Đèn giao lộ đang tiếp cận, frame đầu tiên | Track `traffic_light`, `state=green`, `relevance=ego_relevant` | Quy tắc chuỗi video tiếp cận |
| `BDD14` | Cao tốc ngoại ô, chỉ có biển chỉ dẫn lớn | **Không vẽ box nào** (Negative sample) | Quy tắc loại trừ biển báo cao tốc |
| `BDD18` | Phố đêm: 2 đốm sáng xanh ngọc trên cao | Vẽ 2 box ôm lõi bóng xanh, `state=green`, `relevance=ego_relevant`. Bỏ qua đèn hậu và đèn cao áp | Quy tắc Invisible Housing ban đêm |

---

## 12. QUY TRÌNH THAO TÁC KỸ THUẬT TRÊN CVAT (SOP)

### Bước 1: Mở Job và Sanity Check
- Mở link Job được giao (Job #20 cho Calibration hoặc Job #21 cho Golden/Blind).
- Bấm phím **`N`** để kiểm tra menu: Nhãn `traffic_light` hiển thị với 4 thuộc tính dropdown và checkbox. Giá trị default là `__undefined__`.

### Bước 2: Quy trình 5 bước gán nhãn từng ảnh
1. **Quét tổng quan (Global Scan):** Xác định cấu trúc nút giao, hướng đi của xe (ego lane), vị trí các cột gantry và ngã tư gần / ngã tư xa.
2. **Zoom chi tiết (300% - 400%):** Lăn chuột phóng to vào từng vị trí có đầu đèn.
3. **Vẽ Bounding Box (Phím `N`):**
   - Click góc trên-trái rồi góc dưới-phải của hộp đầu đèn.
   - Nếu là ban đêm không thấy vỏ: ôm sát phần lõi sáng của bóng đèn (Visible Lamp Rule).
4. **Gán thuộc tính:**
   - Chọn `state` (`red`, `yellow`, `green`, `off`, `unknown`).
   - Chọn `relevance` (`ego_relevant`, `other_lane`, `pedestrian`, `unknown`).
   - Đánh dấu `occluded` nếu bị che >= 50%.
   - Đánh dấu `needs_review` nếu có điểm nghi vấn.
5. **Lưu bài (`Ctrl + S`):** Luôn bấm lưu trước khi nhấn phím `F` để chuyển sang frame kế tiếp.

---

## 13. QUY TRÌNH THẨM ĐỊNH (REVIEW SOP) & CHECKLIST NGHIỆM THU CỦA LEAD

Với vai trò **Lead & Reviewer (ĐỖ TUẤN KIÊN)**, quy trình soát xét job thực hiện theo 2 tầng kiểm soát:

### 13.1. Checklist Nghiệm thu Từng Frame (Per-Frame Gate)
- [ ] Không có box nào bao cả cột đèn gantry hoặc thanh giàn kim loại.
- [ ] Bounding box ôm khít mép đầu đèn, sai số dung sai <= 3 px.
- [ ] Ban đêm: Không bị nhầm đốm sáng đèn cao áp đô thị thành traffic light.
- [ ] Ban đêm: Không bị nhầm đèn hậu ô tô (tail lights) thành traffic light.
- [ ] Với ảnh BDD18: Đèn giao thông thật được gán đúng `state=green` (Visible Lamp).
- [ ] Cụm nhiều đầu đèn được tách thành các box độc lập, không gom chung 1 box.
- [ ] Đèn của ngã tư phía sau (far intersection) được gán đúng `relevance=other_lane`.
- [ ] **100% thuộc tính đã được chọn**: Tuyệt đối không còn giá trị `__undefined__`.
- [ ] Các ca nghi vấn lóa mờ đã được gán `needs_review=true`.

### 13.2. Checklist Nghiệm thu Toàn Job (Job-Level Gate)
- [ ] Hoàn thành 100% số frame trong task.
- [ ] Tỷ lệ lỗi Critical (sai đỏ/xanh ego lane, nhầm far intersection): **0.0%**.
- [ ] Bấm Save trên CVAT và xuất file dataset định dạng `CVAT for images 1.1`.

---

## 14. QUY CHẾ VẬN HÀNH: CLARIFICATION LOG & SỔ QUYẾT ĐỊNH

1. **Ghi nhận thắc mắc:** Khi annotator hoặc nhóm peer gặp tình huống chưa rõ trong blind handoff:
   - Ghi nhận ngay vào [clarification_log.csv](project/07_blind_handoff/clarification_log.csv) gồm `time`, `asker`, `question`, `answered_how`, `guideline_change`.
   - Tuyệt đối không giải thích bằng miệng ngoài guideline trong blind window.
2. **Cập nhật quy tắc:**
   - Mọi thắc mắc hợp lý được đưa vào phiên bản guideline tiếp theo.
   - Ghi chi tiết lý do và bằng chứng vào [08_revision_log.md](project/08_revision_log.md).
