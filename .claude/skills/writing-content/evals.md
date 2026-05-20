---
name: writing-content-evals
description: File test scenarios cho skill writing-content. Không phải skill gọi trực tiếp — dùng để verify writing-content hoạt động đúng.
---

# EVALS — writing-content

## Eval 1: Golden path — nhận outline đầy đủ direction

**User input**: (Agent gọi) writing-content với outline hoàn chỉnh: H1, sapo brief, 6 H2 (mỗi H2 có góc + bullet + từ ước tính), 2 H2 có H3, kết bài brief. Keyword "bảo hiểm sức khỏe", target 2000 từ.

**Expected behavior**:
- Viết sapo (100–150 từ) — có pain point, preview, hook
- Viết H2.1 → review → H2.2 → review → ... → H2.6 → kết bài
- Mỗi H2 có câu mở trước bullet, follow đúng content direction
- Gộp thành bài hoàn chỉnh ≈1600–2400 từ

**Pass criteria**: Bài hoàn chỉnh có đủ tất cả sections, sapo không khô khan, không có H2 nào chỉ là bullet thuần, tổng từ trong ±20% target

---

## Eval 2: Edge case — outline thiếu content direction cho 1 H2

**User input**: writing-content với outline có 5 H2, nhưng H2.3 chỉ có tên heading ("## So sánh các loại bảo hiểm") không có góc, bullet, từ ước tính

**Expected behavior**:
- Ghi chú trước khi viết: "H2.3 (So sánh các loại bảo hiểm) thiếu direction — sẽ viết dựa trên tên heading và keyword"
- Vẫn viết section đó dựa trên tên heading + context từ keyword
- Không dừng pipeline vì 1 section thiếu direction

**Pass criteria**: Ghi chú rõ section thiếu direction, bài vẫn được hoàn thành, section H2.3 có nội dung hợp lý dựa trên tên heading

---

## Eval 3: Anti-pattern — không có outline, yêu cầu viết bài thẳng

**User input**: /writing-content viết bài về "bảo hiểm sức khỏe" cho tôi

**Expected behavior**:
- Nhận ra không có outline được cung cấp
- Báo rõ "cần outline từ writing-outline trước khi viết nội dung"
- Gợi ý: chạy `seo-article-writer "bảo hiểm sức khỏe"` để thực hiện toàn bộ pipeline, hoặc cung cấp outline thủ công
- Không tự viết bài từ đầu khi chưa có outline

**Pass criteria**: Không viết bài khi thiếu outline, giải thích rõ tại sao cần outline, đề xuất action tiếp theo cụ thể
