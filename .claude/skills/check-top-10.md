---
name: check-top-10
description: Phân tích top 10 SERP cho một từ khóa — kiểm tra trùng lặp tiêu đề, mô tả và nội dung. Dùng: /check-top-10 "từ khóa"
argument-hint: '"từ khóa cần phân tích"'
---

# /check-top-10 — Phân tích SERP Top 10

Nhận `$ARGUMENTS` là từ khóa cần phân tích. Thực hiện **đúng thứ tự** các bước sau.

---

## BƯỚC 1 — Lấy dữ liệu SERP (Ahrefs)

Gọi tool `mcp__claude_ai_Ahrefs__serp-overview` với:
- `keyword`: lấy từ `$ARGUMENTS` (bỏ dấu ngoặc nếu có)
- `country`: `vn`
- `top_positions`: `10`
- `select`: `url,title,position,domain_rating,traffic,backlinks,refdomains,top_keyword,top_keyword_volume,keywords`

Lưu toàn bộ kết quả `positions[]` vào bộ nhớ làm việc.

---

## BƯỚC 2 — Phân tích tiêu đề trùng lặp

Từ danh sách `positions[]`, kiểm tra:

**2a. Trùng hoàn toàn:** So sánh từng cặp title — ghi nhận nếu 2 title giống nhau 100%.

**2b. Trùng cụm từ chính:** Tìm các cụm từ ≥ 3 từ xuất hiện trong ≥ 2 title (ví dụ: "định cư Châu Âu dễ nhất").

**2c. Domain trùng:** Đếm số lần mỗi domain xuất hiện trong top 10 — đánh dấu domain nào có ≥ 2 URL.

---

## BƯỚC 3 — Lấy meta description (load on-demand)

Chỉ fetch khi cần thiết. Với **từng URL** trong top 10, gọi `WebFetch` với prompt:
```
Extract ONLY: meta title tag and meta description tag from HTML head. Return exactly:
TITLE: [content]
META_DESC: [content]
```

> Nếu WebFetch bị từ chối hoặc lỗi cho 1 URL, ghi nhận "Không lấy được" và tiếp tục — không dừng toàn bộ quy trình.

Sau khi có kết quả, kiểm tra:
- Meta description trùng hoàn toàn giữa các trang
- Meta description quá ngắn (< 50 ký tự) hoặc quá dài (> 160 ký tự)
- Meta description không chứa từ khóa chính

---

## BƯỚC 4 — Phân tích nội dung trùng lặp

Dựa vào title + URL slug để nhóm theo ý nghĩa:

| Mức độ | Tiêu chí |
|--------|----------|
| **Cao** | Cùng domain + chủ đề gần nhau |
| **Trung bình** | Khác domain nhưng title cùng góc độ tiếp cận |
| **Thấp** | Cùng chủ đề rộng nhưng góc độ khác nhau |

---

## BƯỚC 5 — Xuất kết quả

Trình bày theo cấu trúc chuẩn sau (không thêm, không bớt):

```
## Top 10 SERP — "[từ khóa]" (VN)

| # | Domain | Title | DR | Traffic | Ref Domains |
|---|--------|-------|----|---------|-------------|
...

---

## Phân tích trùng lặp

### 1. Tiêu đề (Title)
- Trùng hoàn toàn: [liệt kê hoặc "Không có"]
- Trùng cụm từ: [liệt kê nhóm + số lần]
- Domain xuất hiện nhiều lần: [liệt kê hoặc "Không có"]

### 2. Mô tả (Meta Description)
- Trùng hoàn toàn: [liệt kê hoặc "Không có"]
- Vấn đề độ dài: [liệt kê URL bị lỗi]
- Thiếu từ khóa: [liệt kê URL]
(Ghi rõ nếu không lấy được dữ liệu)

### 3. Nội dung (Content)
| Mức độ | Các trang |
|--------|-----------|
| Cao    | ...       |
| Trung bình | ...   |
| Thấp   | ...       |

---

## Kết luận & Cơ hội

- Góc độ còn bỏ trống: ...
- Điểm yếu đối thủ (DR thấp, 0 backlink): ...
- Nên tránh: ...
```

---

## VẾT XE ĐỔ — Lỗi thường gặp

| Lỗi | Cách xử lý |
|-----|-----------|
| `column 'description' not found` | Ahrefs không có trường description — dùng WebFetch thay thế |
| WebFetch bị từ chối | Ghi "Không lấy được" và tiếp tục, không dừng |
| Tool schema chưa load | Gọi `ToolSearch` với `select:mcp__claude_ai_Ahrefs__serp-overview` trước |
| Từ khóa tiếng Việt bị encode sai | Giữ nguyên Unicode, không escape |
| `render-data-table` bị từ chối | Bỏ qua render, trình bày bằng markdown table thay thế |

---

## TIÊU CHÍ CHẤT LƯỢNG — Tự kiểm trước khi trả lời

- [ ] Đủ 10 kết quả trong bảng (không thiếu dòng)
- [ ] Phân tích title có ít nhất 2 trong 3 mục: trùng hoàn toàn / trùng cụm từ / domain trùng
- [ ] Phần meta description có dữ liệu HOẶC ghi rõ lý do không có
- [ ] Bảng nội dung trùng lặp có ít nhất 1 nhóm được phân loại
- [ ] Phần "Kết luận & Cơ hội" có ít nhất 1 cơ hội cụ thể được chỉ ra
- [ ] Không có thông tin bịa đặt — chỉ dựa vào dữ liệu thực từ Ahrefs và WebFetch
