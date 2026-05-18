# Claude Code Skills — SEONGON

Bộ skills tùy chỉnh cho Claude Code, được xây dựng cho workflow SEO tại SEONGON.

## Cài đặt

Copy toàn bộ thư mục `.claude/skills/` vào `~/.claude/commands/`:

```bash
cp .claude/skills/*.md ~/.claude/commands/
```

Khởi động lại session Claude Code để skills có hiệu lực.

---

## Danh sách Skills

### 1. `/check-top-10` — Phân tích SERP Top 10
Lấy top 10 kết quả Google cho một từ khóa và phân tích trùng lặp về tiêu đề, mô tả, nội dung.

**Dùng:**
```
/check-top-10 "từ khóa cần phân tích"
```

**Đầu ra:**
- Bảng top 10 kèm DR, Traffic, Ref Domains
- Phân tích title trùng lặp (hoàn toàn / cụm từ / domain)
- Nhóm nội dung trùng lặp theo mức độ
- Cơ hội & điểm yếu đối thủ

---

### 2. `/check-seo-post` — Kiểm tra bài viết chuẩn SEO
Đánh giá bài viết theo 53 tiêu chí chuẩn SEO SEONGON, chấm điểm /100.

**Dùng:**
```
/check-seo-post https://example.com/bai-viet "từ khóa chính"
```

**Đầu ra:**
- Tổng điểm /100
- Danh sách tiêu chí PASS ✅ / FAIL ❌ / Cần kiểm tra thủ công ⚠️
- Bảng hành động ưu tiên

> File checklist đầy đủ: `.claude/skills/seongon-seo-checklist.md`

---

### 3. `/tao-lich-hop` — Tạo lịch họp Google Calendar
Tạo sự kiện trên Google Calendar và tự động gửi invite đến người tham dự.

**Dùng:**
```
/tao-lich-hop Tên cuộc họp | DD/MM/YYYY HH:MM | DD/MM/YYYY HH:MM | email1, email2
```

**Yêu cầu:** Kết nối Google Calendar qua `/mcp` trước khi dùng.

---

## Nguồn gốc

Skills được tạo ra trong quá trình training Claude Code tại SEONGON.
Lịch sử trò chuyện: xem file `conversation-history.txt`
