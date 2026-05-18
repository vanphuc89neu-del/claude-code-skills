---
name: tao-lich-hop
description: Tạo lịch họp trên Google Calendar cho nguyenvanphuc@seongon.com — tự động gửi invite đến người tham dự. Dùng: /tao-lich-hop <tên cuộc họp> | <thời gian> | <email người họp,...>
argument-hint: "<tên cuộc họp> | <thời gian bắt đầu> | <thời gian kết thúc> | <email1,email2,...>"
tools: mcp__claude_ai_Google_Calendar
---

# /tao-lich-hop — Tạo lịch họp Google Calendar

Nhận `$ARGUMENTS` với cấu trúc phân cách bằng `|`:

```
/tao-lich-hop Họp kế hoạch Q3 | 20/05/2026 14:00 | 20/05/2026 15:00 | abc@gmail.com, xyz@seongon.com
```

---

## BƯỚC 1 — Phân tích đầu vào

Tách `$ARGUMENTS` theo dấu `|` thành:
- `MEETING_TITLE` : phần 1 — tên cuộc họp
- `START_TIME`    : phần 2 — thời gian bắt đầu (định dạng DD/MM/YYYY HH:MM hoặc tự nhiên như "ngày mai 3h chiều")
- `END_TIME`      : phần 3 — thời gian kết thúc (nếu không có, mặc định = START_TIME + 1 tiếng)
- `ATTENDEES`     : phần 4 — danh sách email phân cách bằng dấu phẩy

> Nếu người dùng không điền đủ 4 phần, hỏi lại phần còn thiếu trước khi tiếp tục.

**Luôn thêm `nguyenvanphuc@seongon.com` vào danh sách attendees** (organizer).

---

## BƯỚC 2 — Kiểm tra xác thực

Thử gọi bất kỳ Calendar tool nào. Nếu nhận được lỗi authentication:
1. Thông báo: *"Google Calendar chưa được kết nối. Bạn gõ `/mcp` → chọn 'claude.ai Google Calendar' để xác thực nhé!"*
2. Dừng lại, chờ người dùng xác thực xong rồi tiếp tục.

---

## BƯỚC 3 — Chuyển đổi thời gian

Chuyển `START_TIME` và `END_TIME` sang định dạng ISO 8601:
- Múi giờ: **Asia/Ho_Chi_Minh (UTC+7)**
- Ví dụ: `20/05/2026 14:00` → `2026-05-20T14:00:00+07:00`
- Nếu chỉ có ngày, không có giờ: hỏi lại giờ bắt đầu

---

## BƯỚC 4 — Tạo sự kiện Calendar

Gọi tool tạo sự kiện Google Calendar với các tham số:

```
title       : MEETING_TITLE
start       : START_TIME (ISO 8601, timezone Asia/Ho_Chi_Minh)
end         : END_TIME   (ISO 8601, timezone Asia/Ho_Chi_Minh)
attendees   : [nguyenvanphuc@seongon.com] + ATTENDEES (tách bởi dấu phẩy)
description : "Lịch họp được tạo tự động qua Claude Code"
send_notifications : true   ← BẮT BUỘC để gửi invite email
```

> Google Calendar tự động gửi email invite đến tất cả attendees khi `send_notifications = true`.

---

## BƯỚC 5 — Xuất kết quả

Sau khi tạo thành công, trình bày:

```
✅ Đã tạo lịch họp thành công!

📅 Tên:        [MEETING_TITLE]
🕐 Bắt đầu:   [START_TIME dạng DD/MM/YYYY HH:MM]
🕑 Kết thúc:  [END_TIME dạng DD/MM/YYYY HH:MM]
👥 Người dự:  [liệt kê từng email]
📧 Invite:    Đã gửi email mời đến tất cả người tham dự
🔗 Link:      [event link nếu có]
```

---

## VẾT XE ĐỔ — Lỗi thường gặp

| Lỗi | Cách xử lý |
|-----|-----------|
| Chưa xác thực Google Calendar | Hướng dẫn: `/mcp` → chọn "claude.ai Google Calendar" |
| `$ARGUMENTS` thiếu phần | Hỏi lại đúng phần còn thiếu, không hỏi lại toàn bộ |
| Email sai định dạng | Báo cụ thể email nào sai, đề nghị sửa |
| Thời gian kết thúc trước thời gian bắt đầu | Báo lỗi + hỏi lại thời gian kết thúc |
| Thời gian không rõ (vd: "chiều mai") | Xác nhận lại ngày giờ cụ thể với người dùng trước khi tạo |
| Tool không trả về event link | Bỏ qua dòng "Link", vẫn xác nhận tạo thành công |

---

## TIÊU CHÍ CHẤT LƯỢNG — Tự kiểm trước khi trả lời

- [ ] `nguyenvanphuc@seongon.com` luôn có trong danh sách attendees
- [ ] Thời gian đã chuyển đúng sang UTC+7
- [ ] `send_notifications = true` được truyền vào (để gửi invite)
- [ ] Đã xác nhận lại với người dùng nếu thời gian mơ hồ
- [ ] Output hiển thị đầy đủ: tên, giờ, người dự, trạng thái invite
