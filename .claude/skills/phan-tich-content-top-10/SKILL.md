---
name: phan-tich-content-top-10
description: This skill should be used when the user asks to "phân tích nội dung đối thủ", "tìm khoảng trống nội dung", "đánh giá bài viết đối thủ", "content gap analysis", "competitor content analysis", "/phan-tich-content-top-10 [URLs] [keyword]", "kiểm tra 10 bài đối thủ", or when an agent passes a list of competitor URLs from check-top-10 and wants to evaluate content quality and identify gaps. Fetches each URL, analyzes content structure and topic coverage, and returns a comparative gap report. Use when preparing content briefs or researching content opportunities for a keyword.
---

# phan-tich-content-top-10

Nhận danh sách URL đối thủ từ top 10 SERP — fetch từng bài, trích xuất cấu trúc heading + topic, phát hiện khoảng trống nội dung và cơ hội khác biệt. Đầu ra là ma trận topic coverage và content gap report để làm cơ sở viết brief.

## Khi nào dùng

Pattern trigger:
- Agent gọi sau khi có output từ `check-top-10`
- `/phan-tich-content-top-10 [url1,url2,...] "[từ khóa]"`
- "phân tích nội dung đối thủ cho từ khóa [X]"
- "tìm khoảng trống nội dung"
- "đánh giá 10 bài đối thủ top Google"
- "competitor content analysis for [keyword]"

KHÔNG dùng skill này khi:
- Chỉ có 1 URL duy nhất → WebFetch trực tiếp nhanh hơn
- URL list trống hoặc < 3 URL hợp lệ → yêu cầu chạy `check-top-10` trước
- User muốn audit bài của mình theo checklist SEONGON → đó là workflow khác, không dùng skill này

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Số URL phân tích | Tất cả URL nhận được (tối đa 10) | User nói "chỉ top 5" |
| URL lỗi | Skip + ghi chú, tiếp tục | User muốn dừng khi có lỗi |
| Ngôn ngữ output | Tiếng Việt | User giao tiếp hoàn toàn tiếng Anh |

## Pipeline — 4 bước

Theo thứ tự, không skip.

### Bước 1 — Phân tích đầu vào

Tách argument thành:
- `URL_LIST`: danh sách URL đối thủ (từ check-top-10 hoặc user cung cấp trực tiếp)
- `MAIN_KEYWORD`: từ khóa chính

Nếu thiếu `MAIN_KEYWORD`: hỏi ngay trước khi tiếp tục.
Nếu `URL_LIST` < 3 URL: báo "cần ít nhất 3 URL để phân tích pattern" và gợi ý chạy `check-top-10` trước.

### Bước 2 — Fetch và trích xuất nội dung từng URL

Với từng URL trong `URL_LIST`, gọi WebFetch với prompt:

```
Trích xuất từ bài viết này:
1. H1 — nội dung chính xác
2. Tất cả H2 — liệt kê đầy đủ theo thứ tự xuất hiện
3. H3 dưới mỗi H2 (nếu có) — liệt kê theo nhóm
4. Góc tiếp cận bài (1 từ: how-to / list / guide / comparison / definition / case-study / news)
5. Đoạn sapo — 200 ký tự đầu tiên
6. Có bảng, checklist, video, infographic không? (liệt kê loại nào có)
7. Ước tính độ dài: ngắn (<800 từ), trung bình (800-2000), dài (>2000) — dựa trên số heading và nội dung
```

URL lỗi (403 / timeout / redirect về trang chủ): ghi "SKIP — [lý do]", tiếp tục các URL còn lại. Không dừng toàn bộ pipeline.

Tối thiểu 3 URL fetch thành công mới tiếp tục bước 3. Dưới 3 URL hợp lệ: báo rõ và dừng.

### Bước 3 — Phân tích cross-article

**3a. Ma trận topic coverage**

Từ tất cả H2/H3 đã trích xuất, tổng hợp danh sách unique topics. Phân loại theo tần suất:

| Tier | Tiêu chí |
|---|---|
| Must-have | Xuất hiện trong ≥ 70% bài |
| Differentiator | Xuất hiện trong 30–70% bài |
| Gap / Blue ocean | Xuất hiện trong < 30% bài hoặc không bài nào đề cập |

Khi gộp topics: coi H2/H3 cùng chủ đề nhưng tên khác nhau là 1 topic (vd: "Cách chọn..." và "Hướng dẫn chọn..." = cùng 1 topic).

**3b. Phân tích góc tiếp cận**

Đếm phân bổ: bao nhiêu bài dùng góc how-to / list / guide / comparison / definition / case-study. Góc nào chưa ai khai thác.

**3c. Phân tích format**

Bao nhiêu bài có bảng, checklist, video, infographic. Format nào còn thiếu trong top 10.

### Bước 4 — Xuất kết quả

Trình bày đúng cấu trúc sau:

```
## Phân tích nội dung đối thủ — "[từ khóa]"
Số bài phân tích thành công: X/[tổng]
URL skip: [domain1 — lý do], [domain2 — lý do] (hoặc "Không có")

---

## Tổng quan từng bài

| # | Domain | H1 | Góc tiếp cận | Format đặc biệt | Độ dài |
|---|--------|----|--------------|-----------------|--------|
[1 dòng mỗi bài fetch thành công]

---

## Ma trận Topic Coverage

### Must-have — Bắt buộc có (≥70% bài đề cập)
- [Topic A] — X/Y bài
- [Topic B] — X/Y bài

### Differentiator — Có thể khác biệt (30–70%)
- [Topic C] — X/Y bài

### Gap — Chưa ai khai thác hoặc ít (<30%)
- [Topic D] — X/Y bài
- [Topic E] — chưa bài nào đề cập

---

## Phân tích góc tiếp cận

| Góc | Số bài | Ví dụ domain |
|-----|--------|--------------|

Góc chưa ai dùng: [liệt kê hoặc "Tất cả góc đã được khai thác"]

---

## Phân tích format

| Format | Số bài có |
|--------|-----------|
| Bảng   | X         |
| Checklist | X      |
| Video  | X         |
| Infographic | X   |

Format chưa ai dùng: [liệt kê]

---

## Tóm tắt cơ hội

| Loại | Chi tiết |
|------|----------|
| Topic gap quan trọng nhất | [topic bỏ trống có tiềm năng cao] |
| Góc khác biệt | [góc tiếp cận chưa ai dùng] |
| Format nổi bật | [format nên thêm để khác biệt] |
| Đối thủ yếu nhất | [domain coverage mỏng hoặc bài ngắn] |
```

## Anti-patterns

- KHÔNG bịa heading. BAD: thêm topic "mà chắc bài có đề cập". GOOD: chỉ liệt kê H2/H3 đã fetch được thực tế.
- KHÔNG bỏ qua lỗi fetch im lặng. BAD: output 10 dòng nhưng 3 URL thực ra failed. GOOD: ghi rõ "7/10 bài fetch thành công, skip: [domain]".
- KHÔNG force phân tích khi < 3 URL hợp lệ. BAD: output gap analysis từ 2 bài. GOOD: báo "insufficient data" và yêu cầu thêm URL.
- KHÔNG đánh giá chất lượng writing hay dữ liệu backlink — chỉ phân tích structure + topic + format từ nội dung fetch được.

## Tiêu chí chất lượng

- [ ] Bảng tổng quan có đủ dòng cho mọi URL fetch thành công
- [ ] Ma trận topic có đủ 3 tier: must-have / differentiator / gap
- [ ] Phần cơ hội có ít nhất 1 gap cụ thể dựa trên dữ liệu thực
- [ ] URL lỗi được ghi chú rõ, không ảnh hưởng logic phân tích
- [ ] Không có topic bịa đặt

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
