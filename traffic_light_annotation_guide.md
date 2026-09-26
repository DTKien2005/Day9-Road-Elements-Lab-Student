# 03 Traffic Light - State & Relevance

Bounding box là phần dễ. Điểm khó là state, relevance và consistency qua thời gian.

## 3.1 Guideline mẫu - ưu tiên DTLD

**Nguồn dự án thật: DriveU Traffic Light Dataset (DTLD).** DTLD có hơn 230k traffic-light annotations, bbox + attributes như relevance, state, orientation, pictogram, occlusion và track identity. Relevance được định nghĩa theo planned route của ego vehicle; traffic light của intersection kế tiếp không được đánh relevant cho intersection hiện tại.

| Attribute | Rule |
| :--- | :--- |
| **bbox** | one box per traffic-light head; bao signal head, không lấy cả pole/gantry nếu scope không yêu cầu |
| **state** | khi so GT DTLD: dùng đúng value trong JSON v2; state có thể unknown và DTLD có transition state như red-yellow |
| **relevance** | relevant nếu light valid cho planned ego route; otherwise not relevant; unknown nếu project cho phép và evidence thiếu |
| **orientation/pictogram** | giữ nếu muốn bài nâng cao; arrow/pedestrian/bike có thể quyết định relevance |
| **occlusion** | flag khi partially hidden/truncated theo source/project rule |
| **track identity** | giữ cùng physical head qua một approach sequence |

> **Đừng remap state trước khi so GT**
> Slide Day 9 dùng schema đơn giản red/yellow/green/off/unknown. DTLD có vocabulary riêng (và state transition). Khi luyện với DTLD, hãy so theo raw GT trước; chỉ map sang schema của project sau, với bảng mapping được version rõ.

**Decision rule cho relevance**
1. Xác định ego lane/hướng đi hoặc planned route trong sequence.
2. Xác định light head thuộc lane group/branch nào qua vị trí, gantry, arrow/pictogram và road geometry.
3. Loại pedestrian/bicycle/cross-street light nếu không điều khiển ego path.
4. Dùng frame trước/sau để lấy context; không dùng temporal context để "bịa" state bị occluded.
5. Nếu evidence không đủ, flag uncertain/needs_review theo schema thay vì đoán.

## 3.2 Tải sample có ground truth

1. **DTLD official:** đăng ký tại Ulm University. Terms hiện tại cho phép dùng miễn phí cho research/teaching nhưng cấm commercial use và cấm chia sẻ dataset cho bên thứ ba. Vì vậy học viên/đơn vị cần tải theo quyền của mình; không re-host raw DTLD trong class repo.
2. **Chọn sequence:** lấy 1 approach sequence tới intersection, khoảng 20-30 frame liên tiếp có ít nhất 2-4 light heads và một state transition nếu có.
3. **Parser/visualizer:** dùng repo dtld_parsing để đọc JSON v2, visualize bbox/attributes và giữ GT tách khỏi task practice.
4. **Upload vào CVAT:** nếu source image là TIFF và môi trường browser/CVAT của lớp có vấn đề, có thể convert sang PNG/JPEG giữ nguyên kích thước pixel; không resize vì bbox GT sẽ lệch.

*   **DTLD official:** https://www.uni-ulm.de/en/in/institute-of-measurement-control-and-microtechnology/research/data-sets/driveu-traffic-light-dataset/
*   **DTLD registration / terms:** https://www.uni-ulm.de/en/in/institute-of-measurement-control-and-microtechnology/research/data-sets/driveu-traffic-light-dataset/registration-form-dtld/
*   **DTLD parser:** https://github.com/julimueller/dtld_parsing

## 3.3 Làm trên CVAT

1. **Project schema:** Label traffic_light. Tạo select attribute state, relevance; optional pictogram/orientation; checkbox occluded. State phải mutable trong video.
2. **Shape vs Track:** ảnh độc lập -> Rectangle/Shape. Sequence/video -> Rectangle/Track để giữ object identity và dùng keyframes.
3. **Draw box:** tight box quanh signal head; tạo annotation riêng cho từng head có meaning riêng.
4. **Track sequence:** đặt keyframe khi bbox thay đổi rõ hoặc state thay đổi; CVAT nội suy vị trí giữa keyframes, nhưng attributes vẫn phải được kiểm tại transition.
5. **Attribute pass:** sau khi geometry ổn, chuyển Attribute Annotation Mode để gán state/relevance nhanh, đặc biệt khi nhiều light heads.
6. **Review:** xem theo sequence, tìm state jump vô lý, relevance drift, missing small/far lights và reflection false positives.

*   **CVAT Docs - Track mode:** https://docs.cvat.ai/docs/manual/basics/track-mode-basics/
*   **CVAT Academy - Track Mode:** https://www.cvat.ai/academy/track-mode
*   **CVAT Academy - Bounding boxes:** https://www.cvat.ai/academy/bounding-box-annotation
*   **CVAT Docs - Attribute mode:** https://docs.cvat.ai/docs/manual/basics/attribute-annotation-mode-basics/
*   **CVAT Academy - Attributes:** https://www.cvat.ai/academy/label-attributes

### Checklist tự chấm

| Check | Pass khi... |
| :--- | :--- |
| **Detection** | không miss light head nhỏ/xa có GT; box không lấy cả pole |
| **State** | state đúng trên từng frame, đặc biệt quanh transition/glare |
| **Relevance** | decision có evidence lane/route/pictogram, không dựa vào vị trí "trông có vẻ" |
| **Temporal** | same physical light giữ track/identity; relevance không drift vô lý |

---

# Traffic light annotation = box + state + relevance

Một đèn màu đúng nhưng không áp cho ego lane vẫn không nên điều khiển ego vehicle.

*   **Box:** Bao quanh light head hoặc visible lamp theo guideline; tránh lấy cả pole/sign gantry nếu không yêu cầu.
*   **State:** red / yellow / green / off / unknown / none. State có thể đổi theo frame.
*   **Relevance:** Đèn này có áp cho ego lane/hướng đi hay chỉ áp cho lane khác, pedestrian, bus, left-turn?
*   **Evidence:** Relevance cần dựa vào lane geometry, arrow, position, sign, temporal context; không đoán tùy cảm tính.

---

# Traffic light state • rule đơn giản nhưng nhiều bẫy

Đèn nhỏ, xa, bị lóa hoặc đèn mũi tên thường gây lỗi.

### State labels
*   **red / yellow / green:** visible active color.
*   **off:** light head visible but no active light.
*   **unknown:** không đủ evidence vì blur/glare/occlusion/far.
*   **none:** dùng cẩn thận; có thể không phải traffic light hoặc label không áp dụng.

### Common traps
*   Đèn phản chiếu trên kính/biển quảng cáo.
*   Arrow light khác với round light.
*   Multiple heads on one pole: box từng head hay cả cluster?
*   Đèn pedestrian/bus/bike không áp cho ego lane.
*   Motion blur làm green/yellow khó phân biệt.

---

# Traffic light relevance decision tree

Đây là phần cần dạy kỹ vì detector thường không biết relevance.

1. **Step 1:** Đèn có nằm trong field of view và visible enough không?
2. **Step 2:** Đèn thuộc road branch/lane group nào? Dựa vào position, arrows, gantry, lane geometry.
3. **Step 3:** Ego lane/hướng đi của frame là gì? Straight, left, right, merge, service lane?
4. **Step 4:** Đèn đó điều khiển ego lane hay object khác: pedestrian, bus, bicycle, cross street?
5. **Step 5:** Nếu thiếu evidence, dùng not_relevant hoặc unknown_relevance theo schema; không đoán.

---

# Temporal consistency • state thay đổi, relevance thường ổn định hơn

Qua video, traffic light state có thể đổi; relevance không nên nhảy lung tung frame-to-frame.

*   **State transition:** red -> yellow -> green hoặc green -> yellow -> red có logic thời gian; jump bất thường là QC signal.
*   **Occlusion handling:** Khi xe tải che đèn 3 frame, không tự suy luận màu nếu guideline không cho phép interpolation.
*   **Relevance drift:** Nếu đèn cùng vị trí lúc thì relevant lúc thì not_relevant, cần xem lại rule hoặc tracking.
*   **Reviewer tactic:** Review traffic light theo sequence, không chỉ từng frame độc lập.

---

# CVAT Sprint 3 • Traffic Light State & Relevance

CVAT in-class exercise • 10 phút annotate + 5 phút debate

**Mục tiêu:** Annotate 5-8 traffic lights trên 3 frame liên tiếp và gán state + relevance.

**Các bước:**
1. Tạo rectangle quanh visible traffic light heads.
2. Gán trafficLightColor cho từng object.
3. Gán relevance: ego_relevant / other_lane / pedestrian / unknown.
4. Dùng frame trước/sau để kiểm consistency nhưng không suy luận quá mức.
5. Mỗi nhóm chọn 1 object để bảo vệ relevance decision.

**Nộp nhanh:**
Traffic light table: object_id, state, relevance, evidence, uncertainty.

**Debrief prompt:**
Hai nhóm đổi task cho nhau: tìm 1 lỗi geometry, 1 lỗi attribute và 1 case cần đưa vào decision log.

---

# Traffic light QC • score theo attribute nhiều hơn geometry

Box IoU cao chưa đủ để sử dụng trong driving policy.

*   **Box quality:** IoU/coverage đủ để chứa light head; small object tolerance riêng.
*   **State accuracy:** red/yellow/green/off/unknown; báo cáo theo slice: small/far/night/glare.
*   **Relevance accuracy:** Sai relevance có thể nghiêm trọng hơn box lệch nhẹ.
*   **Temporal logic:** State transition hợp lý; relevance không drift vô cớ.
*   **False positives:** Tail light, sign reflection, pedestrian light, advertisement.
*   **Escalation:** Unknown phải có lý do: occluded/far/glare/no evidence.