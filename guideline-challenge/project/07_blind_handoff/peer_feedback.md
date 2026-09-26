# Peer feedback + owner response

- **Nhóm peer:** PeerTeam-02
- **Người label blind:** Hoàng Minh Đức

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất? Quy tắc cấm lấy cột đèn (tight head only) và quy tắc tách 2 box độc lập cho đầu đèn tròn vs mũi tên rẽ rất dễ hiểu và quyết định nhanh chóng.
2. Rule nào mơ hồ hoặc phải tự suy diễn? Ban đêm ở ảnh BDD18 có nhiều đốm sáng màu vàng/cam, ban đầu hơi lưỡng lự giữa đèn cao áp chiếu sáng đô thị và đèn giao thông màu vàng.
3. Sample nào khiến guideline "vỡ"? Không có sample nào làm vỡ guideline; mẫu BDD26 nhiều đầu đèn ở cả ngã tư gần và ngã tư xa đòi hỏi phải đọc kỹ quy tắc consecutive intersections.
4. Attribute / default nào trong CVAT dễ gây thao tác sai? Giá trị mặc định `__undefined__` rất hiệu quả để bắt lỗi quên chọn, nhưng cần chú ý khi vẽ liên tiếp nhiều box.
5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn? Bổ sung thêm ví dụ mô tả đặc điểm nhận diện đèn đường cao áp đơn độc ban đêm để annotator tự tin IGNORE ngay.

## 2. Owner phân loại

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Lưỡng lự giữa đốm sáng đèn đường cao áp và đèn vàng ban đêm ở BDD18 | guideline_gap | accept + revise | Bổ sung quy tắc Visual Structure Evidence và ví dụ đèn cao áp vào mục 5.3 trong guideline v3 |
| Băn khoăn về mức độ rõ ràng của đèn ngã tư kế tiếp ở BDD26 | execution_error | coaching | Hướng dẫn đối chiếu vị trí nút giao và gán relevance=other_lane cho far intersection |
| Tầm nhìn bị lóa do tuyết trắng ở BDD24 | data_ambiguity | add_escalation | Thêm quy tắc gán state=unknown và tick needs_review=true khi độ tương phản quá thấp |
