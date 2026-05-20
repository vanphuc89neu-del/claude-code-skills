---
name: writing-outline
description: This skill should be used when the user asks to "viết outline", "tạo dàn bài", "lên cấu trúc bài viết", "create article outline", "outline cho từ khóa", "/writing-outline [keyword]", "tạo outline từ phân tích đối thủ", or when an agent passes phan-tich-content-top-10 gap analysis and needs a structured SEO outline. Analyzes topic priorities from the gap report, writes content direction for each heading individually, then combines into a complete writing-ready outline. Use after phan-tich-content-top-10 to convert competitor insights into article structure.
---

# writing-outline

Nhận kết quả phân tích đối thủ từ phan-tich-content-top-10 — xác định topic priorities, viết content direction từng heading một, rồi gộp lại thành outline hoàn chỉnh sẵn sàng đưa sang writing-content.

## Khi nào dùng

Pattern trigger:
- Agent gọi sau khi có output từ `phan-tich-content-top-10`
- `/writing-outline "[từ khóa]"`
- "viết outline cho bài về [từ khóa]"
- "tạo dàn bài từ gap analysis"
- "lên cấu trúc bài viết từ phân tích đối thủ"
- "create SEO outline from competitor research"

KHÔNG dùng skill này khi:
- Chưa có gap analysis từ phan-tich-content-top-10 → gợi ý chạy `competitor-content-analyzer` trước
- User chỉ muốn list heading nhanh (không cần direction chi tiết) → trả lời thẳng, không cần skill

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Số H2 | 5–7 heading chính | User chỉ định "3 heading" hoặc "bài dài 10 heading" |
| Ngôn ngữ | Tiếng Việt | User giao tiếp hoàn toàn tiếng Anh |
| Phong cách | Evergreen / informational | User chỉ định "comparison", "how-to step-by-step"... |
| Từ mục tiêu | Không bắt buộc | User cung cấp → phân bổ từ theo tỷ lệ |

## Pipeline — 4 bước

Theo thứ tự, không skip.

### Bước 1 — Phân tích đầu vào

| Input | Bắt buộc | Default | Mô tả |
|---|---|---|---|
| `GAP_ANALYSIS` | Có | — | Output từ phan-tich-content-top-10 (ma trận topic coverage + tóm tắt cơ hội) |
| `MAIN_KEYWORD` | Có | — | Từ khóa chính của bài viết |
| `TARGET_WORDS` | Không | Không giới hạn | Tổng từ mục tiêu — nếu có, phân bổ từ theo tỷ lệ cho từng H2 |

Nếu thiếu `GAP_ANALYSIS`: hỏi user muốn cung cấp trực tiếp không, hoặc gợi ý chạy `phan-tich-content-top-10` trước.
Nếu thiếu `MAIN_KEYWORD`: hỏi ngay trước khi tiếp tục.

### Bước 2 — Xác định cấu trúc H2 sơ bộ

Từ gap analysis, phân loại:
- **Must-have topics** → bắt buộc thành H2
- **Differentiator topics** → chọn 1–2 cái nổi bật nhất, làm H2 hoặc H3
- **Gap topics** → ưu tiên làm H2 riêng nếu có tiềm năng cao và phù hợp bài

Sắp xếp H2 theo logic tuyến tính: tổng quan → giải thích / lợi ích → hướng dẫn / chi tiết → so sánh / lựa chọn → kết luận / action.

Ghi ra danh sách H2 sơ bộ trước khi sang bước 3.

### Bước 3 — Viết content direction từng heading một

Với từng H2 trong danh sách, xử lý riêng lẻ theo template:

```
H2: [Tiêu đề heading]
Góc tiếp cận: [how-to | definition | list | comparison | example | Q&A | data-driven]
Nội dung cần có:
  - [điểm chính 1]
  - [điểm chính 2]
  - [điểm chính 3]
H3 (nếu cần):
  - [H3.1 — brief 1 câu]
  - [H3.2 — brief 1 câu]
Khác biệt so với đối thủ: [điểm này làm sâu hơn hoặc khác góc như thế nào]
Từ ước tính: X–Y từ
```

Xử lý xong 1 H2 rồi mới sang H2 tiếp theo. KHÔNG viết tất cả heading cùng lúc.

### Bước 4 — Gộp thành outline hoàn chỉnh

Sau khi xử lý hết tất cả H2, gộp thành 1 document:

```
## OUTLINE — "[từ khóa]"
Tổng từ ước tính: X–Y từ

---

# [H1 — chứa keyword chính, ≤65 ký tự]

**Sapo** (100–150 từ): [Brief — pain point của user, preview giá trị bài, hook cuối sapo]

---

## [H2.1]
Góc: [one word]
Cần có: [bullet points]
[H3 nếu có]
Từ: X–Y

---

## [H2.2 → H2.N]
[Tương tự cấu trúc trên]

---

**Kết bài** (80–120 từ): [Brief — tóm tắt value, CTA hoặc next step rõ ràng]

---

## Ghi chú cho content writer
- Must-have bắt buộc: [liệt kê topics]
- Điểm khác biệt chính: [1–2 điểm nổi bật so với đối thủ]
- Format nên thêm: [bảng / checklist / ví dụ thực tế / infographic]
```

## Anti-patterns

- KHÔNG viết tất cả heading cùng 1 lúc ở bước 3. BAD: dump toàn bộ outline 1 shot. GOOD: xử lý từng H2 → combine ở bước 4.
- KHÔNG bỏ qua gap topics. BAD: chỉ copy must-have topics giống đối thủ. GOOD: ít nhất 1 gap topic thành H2 riêng.
- KHÔNG để H2 chỉ có tên. BAD: "## Lợi ích của X" không có bullet. GOOD: mỗi H2 có góc + điểm cần có + từ ước tính.
- KHÔNG đặt H1 quá dài hoặc thiếu keyword. BAD: H1 trừu tượng không có keyword chính. GOOD: H1 ≤65 ký tự, chứa keyword tự nhiên.

## Tiêu chí chất lượng

- [ ] Ít nhất 1 gap topic từ phan-tich-content-top-10 xuất hiện trong outline
- [ ] Mỗi H2 có đủ: góc + nội dung cần có + từ ước tính
- [ ] H1 ≤65 ký tự, chứa keyword chính
- [ ] Sapo và kết bài có brief direction
- [ ] Tổng từ ước tính hợp lý (1500–3000 từ cho bài SEO standard)

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
