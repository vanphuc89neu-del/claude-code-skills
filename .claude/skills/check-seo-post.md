---
name: check-seo-post
description: This skill should be used when the user asks to "kiểm tra bài viết", "audit bài SEO", "chấm điểm bài viết", "check bài chuẩn SEO không", "đánh giá bài theo checklist SEONGON", "/check-seo-post [url] [keyword]", "bài này đạt chuẩn chưa", or wants to score an article against the SEONGON 53-criteria SEO publishing standard. Fetches the target URL, evaluates each criterion, and returns a /100 score with PASS/FAIL/manual-check breakdown and prioritized action list. Use when reviewing any article before or after publishing.
---

# check-seo-post

Kiểm tra bài viết theo 53 tiêu chí chuẩn đăng bài SEONGON. Chấm điểm /100, liệt kê rõ từng tiêu chí PASS/FAIL/cần kiểm tra thủ công, kèm hành động ưu tiên để cải thiện. Dành cho content writer và SEO specialist trước khi publish hoặc sau khi nhận bài từ cộng tác viên.

## Khi nào dùng

User nói một trong các pattern:
- `/check-seo-post [URL] "[từ khóa chính]"`
- "kiểm tra bài viết [URL]"
- "audit bài SEO này"
- "chấm điểm bài này theo checklist SEONGON"
- "bài viết này đạt chuẩn chưa"

KHÔNG dùng skill này khi:
- URL yêu cầu đăng nhập (trả về 403/redirect) → yêu cầu user paste nội dung trực tiếp
- User chỉ muốn kiểm tra 1 tiêu chí đơn lẻ (trả lời thẳng, không cần skill)
- Chưa có từ khóa chính → hỏi user trước khi chạy

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Ngôn ngữ output | Tiếng Việt | User giao tiếp hoàn toàn tiếng Anh |
| Tiêu chí N/A | Liệt kê riêng, không tính vào điểm | User muốn tính điểm kể cả N/A |
| Ưu tiên hành động | 3 mức: Cao / Trung bình / Thấp | User chỉ muốn top 3 việc cần làm ngay |

## Pipeline — 5 bước

Theo thứ tự, không skip.

### Bước 1 — Phân tích đầu vào

Tách argument thành:
- `TARGET_URL`: URL bài viết
- `MAIN_KEYWORD`: từ khóa chính (bỏ dấu ngoặc kép)

Nếu thiếu URL hoặc từ khóa: hỏi lại đúng phần còn thiếu, không hỏi lại toàn bộ.

### Bước 2 — Load checklist

Đọc file `~/.claude/commands/seongon-seo-checklist.md`.
Ghi nhớ đủ 53 tiêu chí, nhóm, và cách kiểm tra từng mục.

### Bước 3 — Thu thập dữ liệu bài viết

Gọi `WebFetch` với `TARGET_URL` và prompt:

```
Trích xuất thông tin SEO từ trang:
1. Meta title — nội dung chính xác + đếm ký tự
2. Meta description — nội dung chính xác + đếm ký tự
3. H1 — nội dung chính xác + đếm ký tự
4. Tất cả H2 và H3 — liệt kê đầy đủ
5. 300 ký tự đầu (đoạn sapo)
6. 200 ký tự cuối (đoạn kết)
7. Từ/cụm từ bôi đậm trong sapo
8. Có blockquote hoặc ô highlight không?
9. 3 URL ảnh đầu tiên (src attribute)
10. Internal links: số lượng + 3 anchortext mẫu
11. External links: số lượng + domain đích
12. Bullet/list có nhất quán không?
13. Độ dài đoạn văn: ngắn hay dài? Cho 2 ví dụ
14. Có câu nối giữa heading và danh sách không?
```

Lỗi 403 hoặc trang yêu cầu đăng nhập: báo user, đề nghị paste nội dung trực tiếp.

### Bước 4 — Đánh giá từng tiêu chí

Với mỗi trong 53 tiêu chí, đánh giá:
- **PASS** — tiêu chí rõ ràng đạt, có dẫn chứng cụ thể
- **FAIL** — tiêu chí rõ ràng không đạt, ghi rõ lý do + cách sửa
- **N/A** — không thể kiểm tra qua crawl, ghi rõ tool/cách kiểm tra thủ công

Luôn so sánh với `MAIN_KEYWORD`. Đếm ký tự chính xác cho H1, meta title, meta description — không ước lượng.

Tính điểm: `(số PASS / (53 - số N/A)) × 100`, làm tròn đến số nguyên.

### Bước 5 — Xuất kết quả

Trình bày đúng cấu trúc:

```
## Kết quả kiểm tra SEO
URL: [url]
Từ khóa chính: [keyword]

---

## Tổng điểm: XX/100
> X PASS | Y FAIL | Z cần kiểm tra thủ công
> (điểm tự động = X/(53-Z) × 100)

---

## Điểm tốt

### [Nhóm]
- [ID] [Tiêu chí] — [dẫn chứng ngắn]

---

## Điểm cần cải thiện

| ID | Tiêu chí | Vấn đề cụ thể | Cách sửa |
|----|----------|---------------|----------|

---

## Cần kiểm tra thủ công

| ID | Tiêu chí | Công cụ / Cách kiểm tra |
|----|----------|------------------------|

---

## Tóm tắt hành động

| Ưu tiên | Việc cần làm |
|---------|-------------|
| Cao     | ...          |
| Trung bình | ...       |
| Thấp    | ...          |
```

## Anti-patterns

- KHÔNG để trống cột "Cách sửa" trong bảng FAIL. BAD: ghi "Cần cải thiện" mà không có hướng dẫn cụ thể. GOOD: "Thêm từ khóa 'X' vào 150 ký tự cuối đoạn kết".
- KHÔNG ước lượng số ký tự. BAD: "H1 khoảng 60-65 ký tự". GOOD: đếm chính xác "H1 = 62 ký tự".
- KHÔNG đánh PASS khi không có dữ liệu. BAD: đánh PASS meta description khi WebFetch không lấy được. GOOD: đánh N/A và ghi "View Source để kiểm tra".
- KHÔNG bỏ sót nhóm 4 (Meta description) khi WebFetch không trả về. GOOD: đánh tất cả 4 tiêu chí nhóm 4 là N/A với hướng dẫn View Source.

## Tiêu chí chất lượng

- [ ] Đếm ký tự chính xác cho H1, meta title, meta description
- [ ] Mỗi FAIL có lý do cụ thể + gợi ý sửa
- [ ] Điểm tổng tính đúng công thức: PASS / (53 - N/A) × 100
- [ ] Phần hành động có ít nhất 1 mục ưu tiên Cao
- [ ] Không bịa dữ liệu — chỉ đánh giá từ nội dung fetch được thực tế

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `seongon-seo-checklist.md` | 53 tiêu chí đầy đủ + cách kiểm tra | Bước 2 — trước khi đánh giá |
| `check-seo-post-evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
