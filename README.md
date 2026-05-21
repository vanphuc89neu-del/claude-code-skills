# Claude Code Skills — SEONGON

Bộ agents và skills tùy chỉnh cho Claude Code, được xây dựng cho workflow SEO tại SEONGON.

## Cài đặt

**Agents** — đặt tại `.claude/agents/`, Claude Code tự nhận diện tự động.

**Skills** — đặt tại `.claude/skills/<name>/SKILL.md`, gọi bằng `/tên-skill`.

---

## Agents (Subagents)

Agents là các chuyên gia tự động — Claude gọi đúng agent dựa trên ngữ cảnh, không cần gọi thủ công.

### `competitor-content-analyzer` — Phân tích đối thủ & Content Gap (Bước 1/2)

Chuyên gia phân tích nội dung đối thủ. Tự động chạy khi bạn yêu cầu viết bài SEO cho một từ khóa.

**Trigger tự động:**
- "Viết bài viết cho từ khóa [X]"
- "viết bài về [topic]"
- "phân tích đối thủ", "tìm content gap"
- "check top 10 rồi phân tích bài viết"

**Skills sử dụng:**
1. `check-top-10` — Lấy SERP top 10 từ Ahrefs, phân tích trùng lặp
2. `phan-tich-content-top-10` — Fetch từng bài đối thủ, xây dựng ma trận topic coverage, tìm content gap

**Đầu ra:**
- Bảng top 10 kèm DR, Traffic, Ref Domains
- Ma trận topic: Must-have / Differentiator / Gap
- Danh sách URL để chuyển sang Bước 2

> Luôn chạy TRƯỚC `seo-article-writer`

---

### `seo-article-writer` — Viết bài SEO hoàn chỉnh (Bước 2/2)

Chuyên gia viết nội dung SEO. Nhận gap analysis từ `competitor-content-analyzer`, tạo outline và viết bài hoàn chỉnh.

**Trigger tự động:**
- "Viết bài viết cho từ khóa [X]"
- "viết bài SEO", "write article for keyword"
- "tạo bài dựa trên gap analysis"

**Skills sử dụng:**
1. `writing-outline` — Chuyển gap analysis thành outline đầy đủ, xử lý từng H2 riêng lẻ
2. `writing-content` — Viết từng section theo thứ tự, tự review, gộp thành bài hoàn chỉnh

**Đầu ra:**
- Outline chi tiết (có xác nhận với user trước khi viết)
- Bài viết SEO hoàn chỉnh, sẵn sàng đăng

> Luôn chạy SAU `competitor-content-analyzer`

---

## Skills

Skills là các công cụ chuyên biệt, gọi thủ công hoặc được agent tự động gọi trong pipeline.

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
- Danh sách URL để đưa vào `phan-tich-content-top-10`

**Yêu cầu:** Kết nối Ahrefs MCP

---

### 2. `/phan-tich-content-top-10` — Phân tích nội dung đối thủ & Content Gap

Nhận danh sách URL từ `check-top-10`, fetch từng bài, phân tích cấu trúc nội dung và tìm khoảng trống.

**Dùng:**
```
/phan-tich-content-top-10 [danh sách URLs] "từ khóa"
```

**Đầu ra:**
- Tổng quan từng bài (cấu trúc H2/H3, số từ, góc tiếp cận)
- Ma trận topic coverage: Must-have (≥70%) / Differentiator (30–70%) / Gap (<30%)
- Phân tích format và góc tiếp cận
- Tóm tắt cơ hội để đưa vào `writing-outline`

**Yêu cầu:** Tối thiểu 3 URL hợp lệ (không 403/timeout)

---

### 3. `/writing-outline` — Tạo outline bài viết SEO

Nhận gap analysis từ `phan-tich-content-top-10`, xác định topic priorities, viết content direction từng heading, gộp thành outline hoàn chỉnh.

**Dùng:**
```
/writing-outline "từ khóa"
```
*(Thường được `seo-article-writer` gọi tự động sau khi có gap analysis)*

**Đầu ra:**
- Outline đầy đủ: H1, sapo brief, các H2 với góc tiếp cận + điểm cần có + từ ước tính, kết bài
- Ghi chú cho content writer: must-have topics, điểm khác biệt, format đề xuất

**Lưu ý:** Xử lý từng H2 riêng lẻ, không dump toàn bộ outline 1 lần

---

### 4. `/writing-content` — Viết bài SEO từ outline

Nhận outline từ `writing-outline`, viết từng H2 section theo thứ tự, tự review sau mỗi section, gộp thành bài hoàn chỉnh.

**Dùng:**
```
/writing-content
```
*(Thường được `seo-article-writer` gọi tự động sau khi outline được xác nhận)*

**Đầu ra:**
- Bài viết SEO hoàn chỉnh, publication-ready
- Mỗi H2 có câu mở + nội dung theo direction + không chỉ là list thuần
- Tổng từ trong ±20% so với target

**Lưu ý:** KHÔNG viết toàn bài 1 shot — đây là cơ chế kiểm soát chất lượng cốt lõi

---

### 5. `/check-seo-post` — Kiểm tra bài viết chuẩn SEO

Đánh giá bài viết theo 53 tiêu chí chuẩn SEO SEONGON, chấm điểm /100.

**Dùng:**
```
/check-seo-post https://example.com/bai-viet "từ khóa chính"
```

**Đầu ra:**
- Tổng điểm /100
- Danh sách tiêu chí PASS ✅ / FAIL ❌ / Cần kiểm tra thủ công ⚠️
- Bảng hành động ưu tiên

> File checklist đầy đủ: `.claude/skills/phan-tich-content-top-10/seongon-seo-checklist.md`

---

### 6. `/tao-lich-hop` — Tạo lịch họp Google Calendar

Tạo sự kiện trên Google Calendar và tự động gửi invite đến người tham dự.

**Dùng:**
```
/tao-lich-hop Tên cuộc họp | DD/MM/YYYY HH:MM | DD/MM/YYYY HH:MM | email1, email2
```

**Yêu cầu:** Kết nối Google Calendar qua `/mcp` trước khi dùng.

---

## Pipeline đầy đủ: "Viết bài viết cho từ khóa X"

Khi gõ **"Viết bài viết cho từ khóa [X]"**, Claude tự động chạy toàn bộ pipeline:

```
competitor-content-analyzer (Bước 1)
  └─ check-top-10          → SERP data + URL list
  └─ phan-tich-content-top-10 → Gap analysis

seo-article-writer (Bước 2)
  └─ writing-outline       → Outline chi tiết (xác nhận với user)
  └─ writing-content       → Bài viết hoàn chỉnh
```

---

## Nguồn gốc

Skills và agents được tạo ra trong quá trình training Claude Code tại SEONGON.  
Lịch sử trò chuyện: xem các file `session-*.txt`
