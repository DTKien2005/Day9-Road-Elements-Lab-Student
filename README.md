# Day 9 Lab — Road Elements Guideline Design Challenge

Dự án gán nhãn chuẩn công nghiệp: **Traffic Light State & Ego Relevance** (Nhận diện trạng thái và độ liên quan của đèn giao thông cho xe tự hành tại giao lộ nhiều đầu đèn và điều kiện ánh sáng phức tạp).

**Nhóm thực hiện:** **TrafficVision-AI**

### 👥 Danh sách thành viên nhóm:

| Tên | MSV | Vị trí |
| :--- | :---: | :---: |
| **Đỗ Trung Kiên** | 2A202602283 | **Lead** |
| Nguyễn Xuân Quang | 2A202602311 | Thành viên |
| Ngô Minh Tuấn | 2A202602287 | Thành viên |
| Trần Đức Thọ | 2A202602324 | Thành viên |
| Lã Việt Quang | 2A202602267 | Thành viên |

### 📖 Tài liệu Hướng dẫn Gán nhãn (Guideline Specs):
- 📘 **[GUIDELINE.md](GUIDELINE.md)**: **Bản Đầy Đủ (Full Specification — 13 Mục Chi Tiết)**
- ⚡ **[GUIDELINE_QUICKSTART.md](GUIDELINE_QUICKSTART.md)**: **Bản Rút Gọn / Cheatsheet 1 Trang (Dành Cho Người Cần Thao Tác Nhanh)**

Toàn bộ hồ sơ dự án nằm trong thư mục [guideline-challenge/](guideline-challenge/).

---

## 1. Trạng thái dự án (6/6 Gates Passed)

| Gate | Tên Gate | Trạng thái | Bằng chứng nghiệm thu |
| :---: | :--- | :---: | :--- |
| **G1** | **Topic Lock** | ✓ ĐẠT | [00_team.md](guideline-challenge/project/00_team.md), [01_problem_statement.md](guideline-challenge/project/01_problem_statement.md) |
| **G2** | **CVAT Ready** | ✓ ĐẠT | [02_guideline.md](guideline-challenge/project/02_guideline.md) (v1), [03_cvat_labels.json](guideline-challenge/project/03_cvat_labels.json), [sample_pack.csv](guideline-challenge/project/sample_pack.csv) |
| **G3** | **Calibration Done** | ✓ ĐẠT | [06_calibration_measure.csv](guideline-challenge/project/06_calibration_measure.csv), [06_calibration_report.csv](guideline-challenge/project/06_calibration_report.csv), [08_revision_log.md](guideline-challenge/project/08_revision_log.md) (v2) |
| **G4** | **Gold Frozen** | ✓ ĐẠT | [gold_decisions.csv](guideline-challenge/project/04_edge_cases/gold_decisions.csv), [edge_case_cards.md](guideline-challenge/project/04_edge_cases/edge_case_cards.md), `FREEZE.txt` (tag `gold-freeze`) |
| **G5** | **Handoff Complete** | ✓ ĐẠT | [peer export](guideline-challenge/project/07_blind_handoff/peer_output/dongtinh_blind.zip), [transfer_score.csv](guideline-challenge/project/07_blind_handoff/transfer_score.csv), [clarification_log.csv](guideline-challenge/project/07_blind_handoff/clarification_log.csv), [peer_feedback.md](guideline-challenge/project/07_blind_handoff/peer_feedback.md) |
| **G6** | **Final Handoff** | ✓ ĐẠT | [02_guideline.md](guideline-challenge/project/02_guideline.md) (v3), [09_cvat_export_or_task_reference.txt](guideline-challenge/project/09_cvat_export_or_task_reference.txt) |

- **Điểm chuyển giao GTS:** **63.3 / 100** (Decision accuracy 55.6%, Critical 50%, Geometry 100%, Independence 100%).
- **Hậu kiểm frozen gold:** D04, D06, D07 và D08 không khớp Task #27/guideline; nhóm giữ nguyên gold theo protocol, chấm `0` và ghi `gold sai:`. Vì vậy 2 critical escapes này được debrief tách khỏi lỗi guideline/peer.

---

## 2. Các Task trên CVAT Local

Hai task đã được tạo sẵn trên CVAT cục bộ (`http://localhost:8080`) với đầy đủ label, thuộc tính và tích hợp guideline:

1. **Task Calibration (6 ảnh):**
   - **Tên task:** `team-traffic-light-calibration` (Task #26)
   - **Tham chiếu cục bộ:** Task #26, Job #20 (chỉ mở được trên máy chạy CVAT của nhóm)
   - **Mục đích:** 6 ảnh `BDD11`, `BDD12`, `BDD13`, `BDD15`, `BDD17`, `LISA05` để đo lường bất đồng kiểm chuẩn nội bộ.

2. **Task Golden / Blind Set (5 ảnh):**
   - **Tên task:** `team-traffic-light-golden-blind` (Task #27)
   - **Tham chiếu cục bộ:** Task #27, Job #21 (chỉ mở được trên máy chạy CVAT của nhóm)
   - **Mục đích:** 5 ảnh `BDD14`, `BDD18`, `BDD24`, `BDD25`, `BDD26` làm đáp án chuẩn Golden Reference của nhóm.

3. **Task Peer:**
   - Calibration: `PEER-WARMUP-traffic-light-calibration` (Task #29 / Job #23).
   - Blind: `PEER-TEST-traffic-light-blind` (Task #28 / Job #22).

---

## 3. Lệnh kiểm tra trong buổi

Di chuyển vào thư mục bài làm:
```bash
cd guideline-challenge
```

Trên Windows PowerShell:
```powershell
# Kiểm tra trạng thái 6 gate
$env:PYTHONUTF8="1"; python lab9.py status

# Xác thực tính toàn vẹn của mã khóa freeze
$env:PYTHONUTF8="1"; python lab9.py verify

# Kiểm tra tổng thể trước khi nộp
$env:PYTHONUTF8="1"; python lab9.py check
```

Nguồn và giấy phép dữ liệu ảnh: [ATTRIBUTION.txt](ATTRIBUTION.txt).
