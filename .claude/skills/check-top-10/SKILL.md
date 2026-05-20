---
name: check-top-10
description: This skill should be used when the user asks to "check top 10", "kiểm tra top 10", "phân tích SERP", "analyze competitors", "xem đối thủ xếp hạng gì", "soi top 10 Google", "/check-top-10 [keyword]", "top 10 cho từ khóa này là ai", or wants to identify duplicate titles, descriptions, and content patterns in Google top 10 results. Queries Ahrefs SERP data and surfaces competitor metrics, duplication risks, and content gaps. Use for any keyword research or competitive analysis task before writing content.
---

# check-top-10

Phân tích top 10 kết quả Google cho 1 từ khóa — phát hiện tiêu đề trùng lặp, nội dung trùng lặp, và cơ hội còn bỏ trống. Dành cho SEO specialist cần hiểu nhanh bức tranh cạnh tranh trước khi lên brief nội dung.

## Khi nào dùng

User nói một trong các pattern:
- `/check-top-10 "từ khóa"`
- "kiểm tra top 10 [từ khóa]"
- "phân tích SERP [từ khóa]"
- "đối thủ đang xếp hạng gì cho [từ khóa]"
- "check competitors Google [keyword]"

KHÔNG dùng skill này khi:
- User muốn audit 1 bài viết cụ thể → dùng `/phan-tich-content-top-10`
- User muốn research keyword cluster rộng (>1 từ khóa)
- Ahrefs MCP chưa kết nối

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Country | `vn` | User chỉ định market khác: "us", "sg", "au"... |
| Top positions | `10` | User nói "top 5" hoặc "top 20" |
| Ngôn ngữ output | Tiếng Việt | User giao tiếp hoàn toàn tiếng Anh |

## Pipeline — 4 bước

Theo thứ tự, không skip.

### Bước 1 — Lấy dữ liệu SERP

Load schema nếu chưa có: gọi `ToolSearch` với `select:mcp__claude_ai_Ahrefs__serp-overview`.

Gọi `mcp__claude_ai_Ahrefs__serp-overview` với:
- `keyword`: từ argument, bỏ dấu ngoặc kép nếu có
- `country`: `vn` (hoặc override)
- `top_positions`: `10`
- `select`: `url,title,position,domain_rating,traffic,backlinks,refdomains,keywords`

Lỗi `column not found`: xóa trường lỗi khỏi `select`, gọi lại ngay — không hỏi user.

Data sufficiency gate: nếu Ahrefs trả về < 5 kết quả có URL — báo rõ "không đủ dữ liệu để phân tích pattern" và dừng. Không force output.

### Bước 2 — Phân tích tiêu đề trùng lặp

**2a. Trùng hoàn toàn**: so sánh từng cặp title — ghi nhận nếu 2 title giống 100%.

**2b. Trùng cụm từ**: tìm cụm ≥ 3 từ xuất hiện trong ≥ 2 title. Tính tần suất và tỷ lệ %.

**2c. Domain trùng**: đếm số lần mỗi domain xuất hiện trong top 10 — đánh dấu domain có ≥ 2 URL.

### Bước 3 — Phân tích nội dung trùng lặp

Nhóm theo mức độ dựa vào title + URL slug:

| Mức | Tiêu chí |
|---|---|
| Cao | Cùng domain + cùng chủ đề |
| Trung bình | Khác domain + cùng góc tiếp cận |
| Thấp | Cùng chủ đề rộng + góc khác nhau |

### Bước 4 — Xuất kết quả

Trình bày đúng cấu trúc sau, không thêm không bớt:

```
## Top 10 SERP — "[từ khóa]" ([country])

| # | Domain | Title | DR | Traffic | Ref Domains |
|---|--------|-------|----|---------|-------------|
[10 dòng dữ liệu]

---

## Phân tích trùng lặp

### 1. Tiêu đề
- Trùng hoàn toàn: [liệt kê hoặc "Không có"]
- Trùng cụm từ: [bảng cụm từ + số lần xuất hiện + tỷ lệ]
- Domain xuất hiện nhiều lần: [liệt kê hoặc "Không có"]

### 2. Nội dung
| Mức | Các trang |
|---|---|
| Cao | ... |
| Trung bình | ... |
| Thấp | ... |

---

## Kết luận & Cơ hội
- Góc độ còn bỏ trống: ...
- Điểm yếu đối thủ (DR thấp, 0 backlink): ...
- Nên tránh (đã bão hòa): ...

---

## Danh sách URL để phân tích
1. [url vị trí 1]
2. [url vị trí 2]
3. [url vị trí 3]
4. [url vị trí 4]
5. [url vị trí 5]
6. [url vị trí 6]
7. [url vị trí 7]
8. [url vị trí 8]
9. [url vị trí 9]
10. [url vị trí 10]
```

Nếu `render-data-table` bị từ chối: fallback sang markdown table, không báo lỗi cho user.

## Anti-patterns

- KHÔNG bịa dữ liệu. BAD: output "Domain X có DR 45" khi chưa fetch. GOOD: ghi "Không lấy được — tool lỗi".
- KHÔNG bỏ qua mục 2b (cụm từ). BAD: chỉ báo "không có title trùng hoàn toàn" rồi dừng. GOOD: kiểm tra đủ 2a/2b/2c.
- KHÔNG output khi data < 5. BAD: kết luận từ 2 kết quả Ahrefs trả về. GOOD: báo data sufficiency gate fail.
- KHÔNG nhầm lẫn meta description với title. BAD: phân tích description khi Ahrefs không trả về trường đó. GOOD: ghi rõ "meta description không có trong SERP data — cần WebFetch riêng".

## Tiêu chí chất lượng

- [ ] Bảng top 10 có đủ 10 dòng (hoặc ghi rõ lý do thiếu)
- [ ] Phân tích title có đủ cả 3 mục: trùng hoàn toàn / cụm từ / domain
- [ ] Kết luận có ít nhất 1 cơ hội cụ thể dựa trên dữ liệu thực
- [ ] Không có số liệu bịa đặt

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
