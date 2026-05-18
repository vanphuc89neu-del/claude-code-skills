---
name: check-top-10-evals
description: File test scenarios cho skill check-top-10. Không phải skill gọi trực tiếp — dùng để verify check-top-10 hoạt động đúng.
---

# EVALS — check-top-10

## Eval 1: Golden path

**User input**: `/check-top-10 "dịch vụ seo"`

**Expected behavior**:
- Fetch đủ 10 kết quả từ Ahrefs (country=vn)
- Hiển thị bảng 10 dòng với DR, Traffic, Ref Domains
- Phát hiện cụm từ "Dịch vụ SEO tổng thể" xuất hiện nhiều lần (≥5/10)
- Phát hiện domain seotop.com.vn xuất hiện ≥2 lần
- Kết luận nêu ít nhất 1 cơ hội cụ thể

**Pass criteria**: Bảng đủ 10 dòng + cả 3 mục phân tích title có dữ liệu + có kết luận cơ hội

---

## Eval 2: Edge case — từ khóa không dấu / không ngoặc kép

**User input**: `/check-top-10 dinh cu chau au`

**Expected behavior**:
- Xử lý được argument không có dấu ngoặc kép
- Dùng "dinh cu chau au" làm keyword hoặc hỏi lại 1 lần nếu không chắc
- Không crash, không báo lỗi syntax

**Pass criteria**: Fetch thành công hoặc hỏi lại user đúng 1 lần, không dừng hẳn

---

## Eval 3: Anti-pattern — Ahrefs trả về ít kết quả

**User input**: `/check-top-10 "định cư sao hỏa 2099"`

**Expected behavior**:
- Ahrefs trả về < 5 kết quả có URL thực
- Skill kích hoạt data sufficiency gate
- Báo rõ: "Ahrefs chỉ trả về X kết quả — không đủ để phân tích pattern trùng lặp"
- Không output bảng giả hoặc kết luận từ dữ liệu thiếu

**Pass criteria**: Không có output kết luận, có warning rõ ràng về data không đủ
