---
name: writing-outline-evals
description: File test scenarios cho skill writing-outline. Không phải skill gọi trực tiếp — dùng để verify writing-outline hoạt động đúng.
---

# EVALS — writing-outline

## Eval 1: Golden path — nhận đầy đủ gap analysis + keyword

**User input**: (Agent gọi) writing-outline với gap analysis từ phan-tich-content-top-10 cho keyword "bảo hiểm sức khỏe" — gap analysis có đủ 3 tier (must-have: 4 topics, differentiator: 3 topics, gap: 2 topics)

**Expected behavior**:
- Bước 2: lập danh sách H2 sơ bộ gồm 5–7 heading, ít nhất 1 gap topic được đưa vào
- Bước 3: xử lý từng H2 một — mỗi H2 có góc tiếp cận + bullet points + từ ước tính
- Bước 4: gộp thành outline hoàn chỉnh với H1 (≤65 ký tự, có keyword), sapo brief, kết bài brief, ghi chú cho content writer

**Pass criteria**: Outline có đủ cấu trúc (H1, sapo, N×H2 với direction, kết bài, ghi chú), ít nhất 1 gap topic xuất hiện, không có H2 nào thiếu content direction

---

## Eval 2: Edge case — gap analysis chỉ có must-have topics, không có gap

**User input**: writing-outline "từ khóa test" với gap analysis chỉ có must-have topics (tất cả 10 bài đối thủ đều cover giống nhau, không có gap rõ ràng)

**Expected behavior**:
- Ghi nhận "không có gap topic rõ ràng từ phân tích đối thủ"
- Vẫn tạo outline dựa trên must-have + chọn 1–2 differentiator
- Trong phần "Ghi chú cho content writer", ghi rõ "cạnh tranh cao — khác biệt bằng depth, ví dụ thực tế, hoặc format (checklist/bảng)"

**Pass criteria**: Outline vẫn được tạo (không dừng), ghi chú rõ tình huống không có gap, đề xuất cách khác biệt khác

---

## Eval 3: Anti-pattern — thiếu gap analysis

**User input**: /writing-outline "từ khóa SEO"

**Expected behavior**:
- Nhận ra không có gap analysis được cung cấp
- Hỏi user muốn cung cấp gap analysis trực tiếp không
- Gợi ý chạy `competitor-content-analyzer "từ khóa SEO"` để lấy gap analysis trước
- Không tự viết outline từ trí tưởng tượng

**Pass criteria**: Dừng lại, hỏi đúng 1 câu hoặc gợi ý action tiếp theo, không bịa outline khi thiếu input
