---
name: check-seo-post-evals
description: File test scenarios cho skill check-seo-post. Không phải skill gọi trực tiếp — dùng để verify check-seo-post hoạt động đúng.
---

# EVALS — check-seo-post

## Eval 1: Golden path

**User input**: `/check-seo-post https://seongon.com/blog/seo/viet-bai-chuan-seo.html "bài viết chuẩn SEO"`

**Expected behavior**:
- Fetch URL thành công, lấy được H1, H2/H3, sapo, kết bài
- Đếm ký tự H1 chính xác (không ước lượng)
- Chấm đủ 53 tiêu chí: PASS / FAIL / N/A
- Tính điểm đúng công thức PASS/(53-N/A)×100
- Bảng FAIL có cột "Cách sửa" không trống
- Output có phần "Tóm tắt hành động" với ít nhất 1 mục ưu tiên Cao

**Pass criteria**: Có điểm /100 + đủ 3 bảng (PASS/FAIL/N/A) + hành động có ưu tiên Cao

---

## Eval 2: Edge case — trang trả về meta description rỗng

**User input**: `/check-seo-post https://example.com/bai-viet "từ khóa"`

**Expected behavior**:
- WebFetch không lấy được meta description
- Skill đánh tất cả 4 tiêu chí nhóm 4 (meta desc) là N/A
- Trong bảng N/A, ghi rõ "View Source -> tìm `<meta name='description'`"
- Không đánh PASS giả cho meta description

**Pass criteria**: Nhóm 4 có 4 dòng N/A, không có dòng PASS giả

---

## Eval 3: Anti-pattern — thiếu từ khóa chính

**User input**: `/check-seo-post https://seongon.com/blog/seo/viet-bai-chuan-seo.html`

**Expected behavior**:
- Skill phát hiện thiếu từ khóa chính
- Hỏi lại đúng 1 câu: "Từ khóa chính của bài này là gì?"
- Không chạy pipeline khi chưa có từ khóa
- Không hỏi lại cả URL

**Pass criteria**: Dừng lại, hỏi đúng 1 câu về từ khóa, không hỏi lại URL
