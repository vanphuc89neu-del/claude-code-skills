---
name: tao-lich-hop-evals
description: File test scenarios cho skill tao-lich-hop. Không phải skill gọi trực tiếp — dùng để verify tao-lich-hop hoạt động đúng.
---

# EVALS — tao-lich-hop

## Eval 1: Golden path

**User input**: `/tao-lich-hop Hop ke hoach Q3 | 25/05/2026 14:00 | 25/05/2026 15:00 | abc@gmail.com, xyz@seongon.com`

**Expected behavior**:
- Tách đúng 4 phần từ argument
- Chuyển sang ISO 8601 UTC+7: `2026-05-25T14:00:00+07:00`
- Thêm `nguyenvanphuc@seongon.com` vào attendees (tổng 3 người)
- Gọi `create_event` với `send_notifications: true`
- Output xác nhận có: tên, giờ bắt đầu/kết thúc, danh sách email, trạng thái invite

**Pass criteria**: Event được tạo, output đủ 5 trường, organizer luôn có trong danh sách

---

## Eval 2: Edge case — thiếu thời gian kết thúc

**User input**: `/tao-lich-hop Review sprint | 26/05/2026 09:00 | | dev@seongon.com`

**Expected behavior**:
- Phát hiện END_TIME rỗng
- Tự động set END_TIME = START_TIME + 1 tiếng = `2026-05-26T10:00:00+07:00`
- Không hỏi lại user về END_TIME
- Tạo event thành công

**Pass criteria**: Tạo thành công với end_time = start_time + 1h, không hỏi lại

---

## Eval 3: Anti-pattern — thời gian mơ hồ

**User input**: `/tao-lich-hop 1-1 meeting | chiều mai | | boss@company.com`

**Expected behavior**:
- Phát hiện "chiều mai" là thời gian mơ hồ
- Hỏi đúng 1 câu: "Chiều mai là [ngày cụ thể]? Và bắt đầu lúc mấy giờ?"
- Không tự suy đoán ngày rồi tạo event
- Không hỏi lại tên cuộc họp hoặc email

**Pass criteria**: Dừng lại, hỏi đúng 1 câu về ngày/giờ cụ thể, không tạo event khi chưa confirm
