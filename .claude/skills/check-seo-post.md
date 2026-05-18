---
name: check-seo-post
description: Kiểm tra bài viết theo 53 tiêu chí chuẩn SEO SEONGON. Chấm điểm /100, liệt kê đạt/không đạt/cần kiểm tra thủ công. Dùng: /check-seo-post <URL> <từ_khóa_chính>
argument-hint: "<URL bài viết> <từ khóa chính>"
tools: WebFetch
---

# /check-seo-post — Kiểm tra bài viết chuẩn SEO

Nhận `$ARGUMENTS` gồm 2 phần: **URL bài viết** và **từ khóa chính**.
Ví dụ: `https://example.com/bai-viet "bài viết chuẩn SEO"`

> Checklist đầy đủ nằm tại: `~/.claude/commands/seongon-seo-checklist.md`
> Load file đó **trước khi bắt đầu đánh giá** để có đủ 53 tiêu chí và hướng dẫn kiểm tra từng mục.

---

## BƯỚC 1 — Phân tích đầu vào

Tách `$ARGUMENTS` thành:
- `TARGET_URL`: phần đầu tiên (URL bài viết)
- `MAIN_KEYWORD`: phần còn lại (từ khóa chính, bỏ dấu ngoặc kép nếu có)

---

## BƯỚC 2 — Load checklist

Đọc file `~/.claude/commands/seongon-seo-checklist.md`.
Ghi nhớ toàn bộ 53 tiêu chí (ID, nội dung, nhóm, cách kiểm tra).

---

## BƯỚC 3 — Thu thập dữ liệu bài viết

Gọi `WebFetch` với `TARGET_URL` và prompt sau:

```
Trích xuất toàn bộ thông tin SEO sau từ trang web:
1. Meta title (thẻ <title>) — nội dung chính xác + số ký tự
2. Meta description — nội dung chính xác + số ký tự
3. H1 — nội dung chính xác + số ký tự
4. Toàn bộ H2 và H3 — liệt kê tất cả
5. 300 ký tự đầu tiên của body (đoạn sapo)
6. 200 ký tự cuối cùng của body (đoạn kết)
7. Các từ/cụm từ được bôi đậm (bold) trong sapo
8. Có blockquote hoặc ô highlight không?
9. Ảnh: có caption không? URL ảnh có chứa từ khóa không?
10. Internal links: số lượng + 3 ví dụ anchortext
11. External links: số lượng + domain đích
12. Bullet/list có nhất quán không?
13. Độ dài đoạn văn: ngắn (2-3 câu) hay dài?
14. Có câu nối giữa heading và danh sách không?
```

---

## BƯỚC 4 — Đánh giá từng tiêu chí

Dựa vào dữ liệu thu thập được, đánh giá **từng tiêu chí** trong checklist:

- **✅ PASS** — Tiêu chí rõ ràng đạt
- **❌ FAIL** — Tiêu chí rõ ràng không đạt, ghi rõ lý do cụ thể
- **⚠️ N/A** — Không thể kiểm tra qua crawl (cần tool hoặc kiểm tra thủ công), ghi rõ cách kiểm tra

Khi đánh giá:
- Luôn so sánh với `MAIN_KEYWORD` được cung cấp
- Đếm ký tự chính xác cho H1, meta title, meta description
- Không đoán mò — nếu không có dữ liệu thì đánh dấu N/A

---

## BƯỚC 5 — Tính điểm

```
Tổng tiêu chí            = 53
Tiêu chí có thể kiểm tra = 53 - (số N/A)
Điểm tự động            = (số PASS / tiêu chí có thể kiểm tra) × 100
```

Làm tròn đến số nguyên. Ghi chú rõ: "X/Y tiêu chí được kiểm tra tự động, Z tiêu chí cần kiểm tra thủ công."

---

## BƯỚC 6 — Xuất kết quả

Trình bày **đúng cấu trúc** sau, không thêm không bớt:

```
## Kết quả kiểm tra SEO
**URL:** [url]
**Từ khóa chính:** [keyword]

---

## Tổng điểm: XX/100
> X tiêu chí PASS | Y tiêu chí FAIL | Z tiêu chí cần kiểm tra thủ công

---

## ✅ Điểm tốt

### [Tên nhóm]
- [ID] [Mô tả tiêu chí] ✓ — [ghi chú ngắn nếu cần]
...

---

## ❌ Điểm cần cải thiện

| ID | Tiêu chí | Vấn đề cụ thể | Cách sửa |
|----|----------|---------------|----------|
| XX | ...      | ...           | ...      |

---

## ⚠️ Cần kiểm tra thủ công

| ID | Tiêu chí | Công cụ / Cách kiểm tra |
|----|----------|------------------------|
| XX | ...      | ...                     |

---

## Tóm tắt hành động

| Ưu tiên | Việc cần làm |
|---------|-------------|
| 🔴 Cao  | ...          |
| 🟡 Trung bình | ...    |
| 🟢 Thấp | ...          |
```

---

## VẾT XE ĐỔ — Lỗi thường gặp

| Lỗi | Cách xử lý |
|-----|-----------|
| WebFetch không lấy được meta title/description | Ghi N/A cho nhóm 3 và 4, tiếp tục các nhóm khác |
| WebFetch trả về nội dung bị cắt bớt | Gọi lần 2 với prompt hỏi cụ thể đoạn cuối bài |
| Không tách được URL và từ khóa từ $ARGUMENTS | Hỏi lại người dùng: "Bạn có thể gõ lại theo format: /check-seo-post URL 'từ khóa chính' không?" |
| Trang yêu cầu đăng nhập / trả về 403 | Thông báo không thể kiểm tra, đề nghị người dùng paste nội dung trực tiếp |
| Checklist file không tìm thấy | Báo lỗi: "Không tìm thấy ~/.claude/commands/seongon-seo-checklist.md — vui lòng tạo lại file này" |

---

## TIÊU CHÍ CHẤT LƯỢNG — Tự kiểm trước khi trả lời

- [ ] Đã đếm ký tự chính xác cho H1, meta title, meta description (không ước lượng)
- [ ] Điểm tổng được tính đúng công thức: PASS / (53 - N/A) × 100
- [ ] Mỗi FAIL phải có lý do cụ thể + gợi ý sửa
- [ ] Không để trống cột "Cách sửa" trong bảng FAIL
- [ ] Phần "Tóm tắt hành động" có ít nhất 1 hành động ưu tiên Cao (🔴)
- [ ] Không bịa đặt dữ liệu — chỉ đánh giá dựa trên nội dung thực tế fetch được
