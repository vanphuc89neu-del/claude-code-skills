---
name: phan-tich-content-top-10-evals
description: File test scenarios cho skill phan-tich-content-top-10 (phân tích nội dung đối thủ + tìm content gap). Không phải skill gọi trực tiếp — dùng để verify phan-tich-content-top-10 hoạt động đúng.
---

# EVALS — phan-tich-content-top-10

## Eval 1: Golden path — agent truyền URL list từ check-top-10

**User input**: (Agent gọi) phan-tich-content-top-10 với 10 URLs từ SERP "bảo hiểm sức khỏe" + keyword "bảo hiểm sức khỏe"

**Expected behavior**:
- Fetch từng URL, trích xuất H1/H2/H3, góc tiếp cận, format, độ dài
- Tạo bảng tổng quan đủ số dòng tương ứng bài fetch thành công
- Phân loại topics vào 3 tier: must-have / differentiator / gap
- Phần "Tóm tắt cơ hội" có ít nhất 1 gap cụ thể dựa trên dữ liệu thực

**Pass criteria**: Có đủ 4 section chính (tổng quan, ma trận, góc tiếp cận, format), không có topic bịa đặt, header ghi rõ số bài phân tích thành công

---

## Eval 2: Edge case — một số URL bị lỗi fetch

**User input**: /phan-tich-content-top-10 [url1, url2 trả về 403, url3, url4, url5 timeout, url6, url7, url8, url9, url10] "từ khóa test"

**Expected behavior**:
- Ghi chú 2 URL lỗi: "SKIP — 403" và "SKIP — timeout"
- Tiếp tục fetch 8 URL còn lại bình thường
- Header output ghi rõ "8/10 bài fetch thành công, skip: [domain2 — 403], [domain5 — timeout]"
- Phân tích và kết luận chỉ dựa trên 8 bài hợp lệ

**Pass criteria**: Pipeline không dừng khi có URL lỗi, số bài thực sự phân tích ghi rõ, section tóm tắt không tính URL lỗi vào phân tích

---

## Eval 3: Anti-pattern — ít hơn 3 URL

**User input**: /phan-tich-content-top-10 https://example.com/bai-viet "keyword"

**Expected behavior**:
- Nhận ra chỉ có 1 URL, không đủ để phân tích pattern
- Báo rõ "cần ít nhất 3 URL để phân tích pattern — hiện chỉ có 1 URL"
- Gợi ý chạy `check-top-10 "[keyword]"` trước để lấy danh sách URL
- Không chạy bước fetch hay xuất kết quả phân tích

**Pass criteria**: Từ chối chạy pipeline với < 3 URL, không bịa output, đề xuất action tiếp theo rõ ràng
