---
name: tao-lich-hop
description: This skill should be used when the user asks to "tạo lịch họp", "đặt lịch họp", "tạo meeting", "create a meeting", "schedule a call", "lên lịch gặp", "/tao-lich-hop", or wants to create a Google Calendar event and automatically send invites to attendees. Creates the event in nguyenvanphuc@seongon.com's calendar via Google Calendar MCP and sends email invitations. Use whenever scheduling any meeting or call that requires calendar invites.
tools: mcp__claude_ai_Google_Calendar
---

# tao-lich-hop

Tạo sự kiện họp trên Google Calendar cho nguyenvanphuc@seongon.com và tự động gửi email invite đến tất cả người tham dự. Hỗ trợ thời gian dạng tự nhiên (tiếng Việt), tự chuyển sang UTC+7.

## Khi nào dùng

User nói một trong các pattern:
- `/tao-lich-hop [tên] | [bắt đầu] | [kết thúc] | [email,...]`
- "tạo lịch họp [tên cuộc họp]"
- "đặt meeting lúc [giờ] với [người]"
- "lên lịch gặp [tên] vào [thời gian]"
- "create a meeting with [person] at [time]"

KHÔNG dùng skill này khi:
- Google Calendar chưa xác thực → hướng dẫn auth trước
- User chỉ muốn xem lịch hiện có → dùng tool list_events trực tiếp
- Sự kiện không cần invite (ghi nhớ cá nhân) → gợi ý tạo thủ công

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Organizer | `nguyenvanphuc@seongon.com` | Không override — luôn thêm |
| Thời lượng | 1 tiếng | User chỉ định thời gian kết thúc |
| Timezone | Asia/Ho_Chi_Minh (UTC+7) | User chỉ định timezone khác |
| Send invites | true | Không override — luôn gửi |
| Ngôn ngữ confirm | Tiếng Việt | User giao tiếp tiếng Anh |

## Pipeline — 5 bước

Theo thứ tự, không skip.

### Bước 1 — Phân tích đầu vào

Tách argument phân cách bằng `|`:
- Phần 1 → `MEETING_TITLE`
- Phần 2 → `START_TIME`
- Phần 3 → `END_TIME` (nếu thiếu: START_TIME + 1 tiếng)
- Phần 4 → `ATTENDEES` (danh sách email, phân cách bằng dấu phẩy)

Nếu thiếu phần nào trong 1, 2, 4: hỏi lại đúng phần còn thiếu, không hỏi lại toàn bộ.
Nếu thời gian mơ hồ ("chiều mai", "cuối tuần"): xác nhận ngày giờ cụ thể trước khi tiếp tục.

### Bước 2 — Kiểm tra xác thực

Load schema: `ToolSearch` với `select:mcp__claude_ai_Google_Calendar__create_event`.

Nếu nhận lỗi authentication:
1. Thông báo: "Google Calendar chưa được kết nối. Gõ `/mcp` -> chọn 'claude.ai Google Calendar' để xác thực."
2. Dừng lại, chờ user xác thực xong.

### Bước 3 — Chuyển đổi thời gian

Chuyển `START_TIME` và `END_TIME` sang ISO 8601 timezone Asia/Ho_Chi_Minh:
- `20/05/2026 14:00` -> `2026-05-20T14:00:00+07:00`
- "ngày mai 3 giờ chiều" -> xác nhận ngày cụ thể với user trước

Kiểm tra: END_TIME phải sau START_TIME. Nếu không: báo lỗi + hỏi lại END_TIME.

### Bước 4 — Tạo sự kiện

Gọi `mcp__claude_ai_Google_Calendar__create_event` với:

```
title              : MEETING_TITLE
start_time         : START_TIME (ISO 8601 UTC+7)
end_time           : END_TIME   (ISO 8601 UTC+7)
attendees          : [nguyenvanphuc@seongon.com] + ATTENDEES
description        : "Lịch họp được tạo qua Claude Code"
send_notifications : true
```

### Bước 5 — Xác nhận kết quả

Sau khi tạo thành công, trình bày:

```
Da tao lich hop thanh cong.

Ten:       [MEETING_TITLE]
Bat dau:   [DD/MM/YYYY HH:MM]
Ket thuc:  [DD/MM/YYYY HH:MM]
Nguoi du:  [liet ke tung email]
Invite:    Da gui email moi den [so luong] nguoi tham du
Link:      [event link neu co]
```

Nếu tool không trả về event link: bỏ dòng Link, vẫn xác nhận tạo thành công.

## Anti-patterns

- KHÔNG tạo event khi thời gian chưa được xác nhận. BAD: dùng "ngày mai" mà không hỏi ngày cụ thể. GOOD: "Bạn muốn đặt vào ngày [ngày cụ thể] đúng không?"
- KHÔNG bỏ `nguyenvanphuc@seongon.com` khỏi attendees. BAD: chỉ gửi invite cho người khác. GOOD: luôn thêm organizer vào danh sách.
- KHÔNG dùng `send_notifications: false`. BAD: tạo event mà không gửi invite. GOOD: luôn set true.
- KHÔNG hỏi lại toàn bộ khi thiếu 1 phần. BAD: "Bạn chưa cung cấp đủ thông tin, hãy nhập lại từ đầu". GOOD: "Bạn chưa cung cấp email người tham dự — email là gì?"

## Tiêu chí chất lượng

- [ ] `nguyenvanphuc@seongon.com` luôn có trong attendees
- [ ] Thời gian đã chuyển đúng sang UTC+7 trước khi gọi tool
- [ ] `send_notifications: true` được truyền vào
- [ ] Thời gian mơ hồ đã được xác nhận với user trước khi tạo
- [ ] Output xác nhận hiển thị đủ: tên, giờ, người dự, trạng thái invite

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `tao-lich-hop-evals.md` | 3 test scenarios | Khi cần test skill hoạt động đúng |
