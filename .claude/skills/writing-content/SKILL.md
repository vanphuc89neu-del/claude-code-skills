---
name: writing-content
description: This skill should be used when the user asks to "viết bài", "viết nội dung", "write article", "tạo bài viết SEO", "/writing-content", "viết bài từ outline", "generate article content", "viết từng section", or when an agent passes a complete outline from writing-outline and needs full article content. Writes each H2 section individually in sequence, then combines into one complete publication-ready article. Use after writing-outline — never write full article in one shot.
---

# writing-content

Nhận outline hoàn chỉnh từ writing-outline — viết từng H2 section một theo thứ tự, tự review sau mỗi section, rồi gộp lại thành 1 bài viết hoàn chỉnh. KHÔNG viết toàn bài 1 lần — mỗi heading xử lý riêng để giữ chất lượng và kiểm soát chiều sâu từng phần.

## Khi nào dùng

Pattern trigger:
- Agent gọi sau khi có output từ `writing-outline`
- `/writing-content`
- "viết bài từ outline này"
- "generate article from outline"
- "viết nội dung cho từng heading"
- "tạo bài viết SEO từ dàn bài"
- "write full article section by section"

KHÔNG dùng skill này khi:
- Chưa có outline từ `writing-outline` → yêu cầu tạo outline trước
- User chỉ muốn viết 1 section cụ thể → viết thẳng section đó, không cần skill
- Outline chỉ có tên H2, không có content direction → yêu cầu bổ sung direction trước khi chạy

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Giọng văn | Informative, chuyên nghiệp, dễ đọc | User cung cấp brand voice / tone guide |
| Độ dài mỗi section | Theo từ ước tính trong outline | User nói "viết ngắn hơn" hoặc "viết chi tiết hơn" |
| Ngôn ngữ | Tiếng Việt | User chỉ định tiếng Anh hoặc song ngữ |

## Pipeline — 4 bước

Theo thứ tự, không skip.

### Bước 1 — Đọc và phân tích outline

| Input | Bắt buộc | Default | Mô tả |
|---|---|---|---|
| `OUTLINE` | Có | — | Output hoàn chỉnh từ writing-outline (H1, H2s với direction, sapo brief, kết bài brief) |
| `BRAND_VOICE` | Không | Informative, chuyên nghiệp | Tone guide hoặc brand voice nếu user cung cấp |
| `LANGUAGE` | Không | Tiếng Việt | Ngôn ngữ bài viết |

Đọc toàn bộ outline. Lập danh sách làm việc:
- H1 + brief sapo
- Danh sách H2 theo thứ tự + content direction từng cái
- H3 trong mỗi H2 (nếu có)
- Brief kết bài
- Ghi chú cho content writer (must-have, điểm khác biệt, format)

Nếu outline thiếu content direction cho H2 nào: ghi chú "section [H2] thiếu direction — viết dựa trên tên heading và keyword", không dừng pipeline.

### Bước 2 — Viết từng section theo thứ tự

Viết lần lượt, xong 1 phần mới sang phần tiếp. Thứ tự cố định:

1. **Sapo** (100–150 từ)
   - Mở bằng pain point hoặc context người đọc đang gặp
   - Preview ngắn gọn giá trị bài mang lại
   - Hook cuối sapo để kéo xuống đọc tiếp

2. **Section H2.1** (bao gồm H3 nếu có)
3. **Section H2.2**
4. ...
5. **Section H2.N**
6. **Kết bài** (80–120 từ)
   - Tóm tắt value chính
   - CTA hoặc next step rõ ràng

**Tiêu chuẩn viết mỗi H2 section:**
- Câu mở section: 1–2 câu dẫn dắt vào topic của H2, không bắt đầu ngay bằng bullet
- Nội dung: follow đúng góc tiếp cận + điểm cần có từ outline
- H3 (nếu có): viết đầy đủ content cho từng H3 trước khi sang H3 tiếp theo
- Keyword chính: xuất hiện tự nhiên trong section, không nhồi
- Câu kết section: bridge nhẹ sang section tiếp (nếu flow tự nhiên)

### Bước 3 — Self-review sau mỗi section

Sau khi viết xong mỗi H2, kiểm tra ngay:
- [ ] Section bao phủ đủ các điểm trong content direction của outline
- [ ] Có câu mở — không bắt đầu ngay bằng bullet hay heading cấp dưới
- [ ] Đọc tự nhiên, không cứng, không nhồi keyword

Nếu fail bất kỳ check nào: sửa ngay trước khi sang section tiếp theo.

### Bước 4 — Gộp thành bài hoàn chỉnh

Sau khi viết xong tất cả sections, gộp theo thứ tự:

```
# [H1]

[Sapo]

## [H2.1]
[Content H2.1]

### [H3.1 nếu có]
[Content H3.1]

### [H3.2 nếu có]
[Content H3.2]

## [H2.2]
[Content H2.2]

...

## [H2.N]
[Content H2.N]

[Kết bài]
```

Sau khi gộp, check nhanh toàn bài:
- Flow đọc liên tục, không bị đứt đoạn giữa các section
- Không có section nào bị thiếu kết hoặc cut-off
- Tổng từ xấp xỉ target trong outline (trong ±20%)

## Anti-patterns

- KHÔNG viết toàn bài 1 shot. BAD: nhận outline → generate 2500 từ 1 lần. GOOD: viết sapo → H2.1 → review → H2.2 → review → ... → combine.
- KHÔNG bỏ qua content direction trong outline. BAD: viết H2 theo ý mình, không theo góc/bullet đã xác định. GOOD: mỗi section reflect đúng direction từ outline.
- KHÔNG nhồi keyword. BAD: mỗi đoạn có keyword ít nhất 1 lần cứng nhắc. GOOD: keyword xuất hiện tự nhiên ở những chỗ hợp lý.
- KHÔNG để section chỉ có heading + bullet list thuần. BAD: H2 → bullet 5 điểm → hết. GOOD: câu dẫn → bullet với explanation → câu kết.

## Tiêu chí chất lượng

- [ ] Mỗi H2 có câu mở + nội dung theo direction + không chỉ là list thuần
- [ ] Sapo ≤150 từ, có hook, không liệt kê chủ đề khô khan
- [ ] Kết bài ≤120 từ, có CTA hoặc next step rõ ràng
- [ ] Tổng từ trong ±20% so với target từ outline
- [ ] Keyword chính xuất hiện ≥3 lần trong bài, không nhồi nhân tạo

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
