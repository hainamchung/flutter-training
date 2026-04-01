# Hướng Dẫn Vận Hành Nhóm Học Tập Flutter

> Tài liệu hướng dẫn chi tiết cách tổ chức, vận hành và duy trì nhóm học tập Flutter hiệu quả trong suốt chương trình đào tạo 8 tuần (16 buổi).

---

## 1. Cấu Trúc Buổi Sync-Up 90 Phút

Mỗi buổi sync-up được chia thành 4 phần rõ ràng:

```
┌─────────────────────────────────────────────────────────────────┐
│  00:00 - 00:15  │  Review (15 phút)                            │
│  00:15 - 00:45  │  Trình bày chủ đề (30 phút)                  │
│  00:45 - 01:15  │  Thảo luận & Q&A (30 phút)                   │
│  01:15 - 01:30  │  Planning & Action Items (15 phút)            │
└─────────────────────────────────────────────────────────────────┘
```

### 1.1. Review (15 phút)

| Thời gian | Nội dung |
|-----------|----------|
| 5 phút | Điểm danh, kiểm tra hoàn thành action items buổi trước |
| 5 phút | Mỗi member chia sẻ nhanh: đã học gì, gặp khó khăn gì |
| 5 phút | Facilitator tóm tắt tiến độ chung của nhóm |

**Lưu ý:**
- Mỗi member chỉ nói tối đa **1-2 phút** trong phần chia sẻ nhanh
- Sử dụng format: "Tuần qua tôi đã... / Tôi gặp khó ở... / Tuần tới tôi sẽ..."
- Facilitator cắt lịch sự nếu ai nói quá thời gian

### 1.2. Trình Bày Chủ Đề (30 phút)

| Thời gian | Nội dung |
|-----------|----------|
| 5 phút | Giới thiệu chủ đề, mục tiêu buổi học |
| 20 phút | Trình bày nội dung chính + live demo code |
| 5 phút | Tóm tắt key takeaways |

**Yêu cầu cho Presenter:**
- Chuẩn bị slide hoặc tài liệu trước ít nhất **2 ngày**
- Bắt buộc có **live coding demo** (không chỉ lý thuyết)
- Chia sẻ code trước buổi học qua GitHub repo của nhóm
- Chuẩn bị **3 câu hỏi thảo luận** cho phần tiếp theo

### 1.3. Thảo Luận & Q&A (30 phút)

| Thời gian | Nội dung |
|-----------|----------|
| 10 phút | Thảo luận theo câu hỏi của Presenter |
| 10 phút | Q&A tự do, debug cùng nhau |
| 10 phút | Code review hoặc pair programming exercise |

**Nguyên tắc thảo luận:**
- Mọi câu hỏi đều có giá trị, không phán xét
- Ưu tiên "show me the code" hơn nói lý thuyết suông
- Nếu câu hỏi quá phức tạp → ghi nhận vào parking lot, xử lý offline

### 1.4. Planning & Action Items (15 phút)

| Thời gian | Nội dung |
|-----------|----------|
| 5 phút | Xác nhận chủ đề buổi tiếp theo, phân công Presenter |
| 5 phút | Giao action items cụ thể cho từng member |
| 5 phút | Feedback nhanh về buổi học (1-5 sao + 1 dòng comment) |

**Action items phải đảm bảo SMART:**
- **S**pecific: Rõ ràng task là gì
- **M**easurable: Đo lường được kết quả
- **A**chievable: Khả thi trong thời gian cho phép
- **R**elevant: Liên quan đến chủ đề đang học
- **T**ime-bound: Có deadline cụ thể

---

## 2. Vai Trò Luân Phiên

Mỗi buổi sync-up có 3 vai trò chính, luân phiên giữa các member:

### 2.1. Presenter (Người trình bày)

**Trách nhiệm:**
- Chuẩn bị nội dung trình bày theo chủ đề được phân công
- Tạo slide/tài liệu và push lên repo trước **48 giờ**
- Chuẩn bị live demo code chạy được
- Soạn 3 câu hỏi thảo luận
- Tạo mini exercise cho nhóm (nếu có thể)

**Tips cho Presenter:**
- Bắt đầu bằng "vì sao cần biết điều này" trước khi đi vào chi tiết
- Dùng diagram để giải thích kiến trúc, flow
- Kể 1 câu chuyện thực tế (bug đã gặp, bài học kinh nghiệm)
- Chuẩn bị backup plan nếu demo code fail

### 2.2. Facilitator (Người điều phối)

**Trách nhiệm:**
- Bắt đầu và kết thúc buổi họp đúng giờ
- Quản lý thời gian từng phần (dùng timer)
- Điều phối thảo luận, đảm bảo mọi người đều tham gia
- Nhắc nhở khi lạc đề
- Bắt đầu và kết thúc recording

**Tips cho Facilitator:**
- Set timer trên điện thoại cho từng phần
- Khi thấy im lặng, chỉ định member cụ thể phát biểu
- Dùng kỹ thuật "round-robin" để mọi người đều nói
- Ghi chú các câu hỏi "parking lot" để xử lý sau

### 2.3. Note-taker (Người ghi chú)

**Trách nhiệm:**
- Ghi chép nội dung chính của buổi họp theo template
- Ghi lại action items với người chịu trách nhiệm và deadline
- Ghi lại các câu hỏi chưa được trả lời (parking lot)
- Push meeting notes lên repo **trong vòng 24 giờ** sau buổi họp
- Ghi lại link recording

**Tips cho Note-taker:**
- Dùng template có sẵn (xem phần 7)
- Không cần ghi nguyên văn, tập trung vào key points
- Dùng bullet points, ngắn gọn
- Tag member liên quan khi ghi action items

---

## 3. Lịch Phân Công 16 Buổi

> Giả sử nhóm có **4 member**: A, B, C, D. Điều chỉnh nếu nhóm có số lượng khác.

### Tuần 1-2: Dart Fundamentals & Flutter Basics

| Buổi | Chủ đề | Presenter | Facilitator | Note-taker |
|------|--------|-----------|-------------|------------|
| 1 | Giới thiệu Dart & Flutter | A | B | C |
| 2 | Dart nâng cao | B | C | D |
| 3 | Widget Tree cơ bản | C | D | A |
| 4 | Layout System | D | A | B |

### Tuần 3-4: State Management & Architecture

| Buổi | Chủ đề | Presenter | Facilitator | Note-taker |
|------|--------|-----------|-------------|------------|
| 5 | Navigation & Routing | A | B | C |
| 6 | State Management cơ bản | B | C | D |
| 7 | Riverpod | C | D | A |
| 8 | BLoC Pattern | D | A | B |

### Tuần 5-6: Networking, Storage & Testing

| Buổi | Chủ đề | Presenter | Facilitator | Note-taker |
|------|--------|-----------|-------------|------------|
| 9 | Clean Architecture | A | B | C |
| 10 | DI & Testing | B | C | D |
| 11 | Networking | C | D | A |
| 12 | Local Storage | D | A | B |

### Tuần 7-8: Advanced Topics & Capstone

| Buổi | Chủ đề | Presenter | Facilitator | Note-taker |
|------|--------|-----------|-------------|------------|
| 13 | Performance Optimization | A | B | C |
| 14 | Animation | B | C | D |
| 15 | Platform Integration | C | D | A |
| 16 | CI/CD & Production | D | A | B |

### Quy tắc luân phiên

- Vai trò xoay vòng theo thứ tự cố định: A → B → C → D → A...
- Nếu member vắng, người tiếp theo trong vòng đảm nhận thay
- Member vắng sẽ làm bù vai trò ở buổi tiếp theo mình tham dự
- Trong trường hợp cần swap, phải thông báo **trước 24 giờ** trong group chat

---

## 4. Mẹo Duy Trì Kỷ Luật Nhóm

### 4.1. Thiết lập quy tắc từ buổi đầu tiên

- [ ] Thống nhất thời gian họp cố định (ví dụ: Thứ 3 & Thứ 5, 19h-20h30)
- [ ] Đồng ý về quy tắc vắng mặt (tối đa 2 buổi vắng không phép)
- [ ] Thống nhất channel liên lạc chính (Slack, Zalo, Teams)
- [ ] Mọi member cam kết bằng văn bản (commitment contract)

### 4.2. Accountability System

**Daily check-in (async):**
Mỗi ngày trong tuần, mỗi member post vào group chat:
```
📅 [Ngày]
✅ Hôm nay: [đã làm gì]
📝 Ngày mai: [dự định làm gì]
🚧 Blocker: [nếu có]
```

**Weekly tracking:**
- Dùng spreadsheet chung để track tiến độ mỗi member
- Mỗi member tự update trước buổi sync-up
- Facilitator review và nhắc nhở nếu cần

### 4.3. Motivation & Engagement

| Phương pháp | Mô tả |
|-------------|--------|
| **Streak board** | Track số buổi tham dự liên tiếp, ai streak dài nhất được khen |
| **Buddy system** | Chia cặp 2 người, nhắc nhở và hỗ trợ nhau hàng ngày |
| **Mini challenges** | Tuần nào cũng có 1 coding challenge nhỏ, ai giải nhanh nhất thắng |
| **Show & Tell** | 5 phút đầu buổi, 1 member chia sẻ điều thú vị phát hiện được |
| **Celebration** | Celebrate mỗi milestone: hoàn thành tuần, hoàn thành module |

### 4.4. Xử Lý Tình Huống Khó

**Member không hoàn thành task:**
1. Nhắc nhở riêng lần 1 (qua DM)
2. Nhắc nhở trong nhóm lần 2
3. Họp riêng tìm hiểu nguyên nhân, hỗ trợ nếu cần
4. Nếu tiếp diễn → điều chỉnh workload hoặc thảo luận nghiêm túc

**Member thường xuyên vắng:**
1. Chủ động hỏi thăm tình hình
2. Đề xuất thay đổi lịch nếu cả nhóm đồng ý
3. Cung cấp recording + notes để catch up
4. Nếu vắng > 3 buổi liên tiếp → trao đổi về commitment

**Xung đột trong nhóm:**
1. Facilitator can thiệp ngay, giữ cuộc thảo luận chuyên nghiệp
2. Focus vào code/vấn đề kỹ thuật, không cá nhân hóa
3. Dùng nguyên tắc: "Disagree and commit"
4. Nếu cần → escalate lên người hướng dẫn/mentor

---

## 5. Onboarding Checklist Cho Member Mới

> Dành cho member join nhóm học giữa chừng (ví dụ: từ buổi 5 trở đi).

### Trước buổi sync-up đầu tiên

- [ ] Được thêm vào group chat (Slack/Zalo/Teams)
- [ ] Được thêm vào GitHub repo của nhóm (quyền write)
- [ ] Được cấp quyền truy cập shared drive (Google Drive/Notion)
- [ ] Nhận bản copy lịch sync-up (Google Calendar invite)
- [ ] Đọc README của repo nhóm
- [ ] Đọc quy tắc nhóm (Group Rules)

### Setup môi trường

- [ ] Cài đặt Flutter SDK (phiên bản thống nhất của nhóm)
- [ ] Cài đặt IDE: VS Code hoặc Android Studio với Flutter plugin
- [ ] Cài đặt Git, tạo SSH key, clone repo nhóm
- [ ] Chạy thử `flutter doctor` → tất cả check pass
- [ ] Cài đặt các extension/plugin cần thiết:
  - Flutter & Dart plugin
  - GitHub Copilot (nếu có license)
  - GitLens
  - Error Lens

### Catch up kiến thức

- [ ] Xem recording các buổi đã bỏ lỡ (link trong meeting notes)
- [ ] Đọc meeting notes tất cả buổi trước
- [ ] Hoàn thành exercises của các buổi đã qua
- [ ] Tự đánh giá theo self-assessment form (xem tài liệu rubric)
- [ ] Lên lịch 1-on-1 với 1 member cũ để hỏi đáp

### Buổi sync-up đầu tiên

- [ ] Facilitator giới thiệu member mới với nhóm (2 phút)
- [ ] Member mới tự giới thiệu: background, kinh nghiệm, mục tiêu (3 phút)
- [ ] Phân công buddy (member cũ) hỗ trợ trong 2 tuần đầu
- [ ] Xác nhận vai trò luân phiên: member mới bắt đầu từ Note-taker
- [ ] Member mới quan sát format buổi đầu, bắt đầu vai trò từ buổi sau

### Tuần đầu tiên

- [ ] Buddy check-in hàng ngày (15 phút)
- [ ] Member mới hoàn thành catch-up exercises
- [ ] Tham gia daily check-in async trên group chat
- [ ] Feedback sau tuần đầu: nhóm có cần điều chỉnh gì không

---

## 6. Hướng Dẫn Record Buổi Sync

### 6.1. Sử dụng Zoom

**Chuẩn bị:**
1. Host (Facilitator) đảm bảo tài khoản có quyền recording
2. Kiểm tra dung lượng cloud storage còn đủ
3. Thông báo cho nhóm biết buổi sẽ được record

**Thao tác:**
1. Bắt đầu meeting → Click **"Record"** → Chọn **"Record to the Cloud"**
2. Khi bắt đầu mỗi phần mới, Facilitator nói rõ: "Bắt đầu phần Review" (giúp tìm kiếm sau)
3. Khi kết thúc → Click **"Stop Recording"** → **"End Meeting"**
4. Zoom sẽ xử lý và gửi link recording qua email

**Sau buổi học:**
1. Download recording từ Zoom Cloud
2. Upload lên shared drive của nhóm (Google Drive/OneDrive)
3. Đặt tên file: `[YYYY-MM-DD]_Buoi-XX_[Chu-de].mp4`
4. Cập nhật link vào meeting notes

### 6.2. Sử dụng Google Meet

**Chuẩn bị:**
1. Host cần tài khoản Google Workspace có quyền recording
2. Tạo meeting link trước và chia sẻ trong calendar invite

**Thao tác:**
1. Vào meeting → Click **"Activities"** (biểu tượng tam giác) → **"Recording"** → **"Start recording"**
2. Xác nhận consent notification cho tất cả participants
3. Khi kết thúc → Click **"Stop recording"**
4. Recording tự động lưu vào Google Drive của host

**Sau buổi học:**
1. Di chuyển file recording vào folder chung của nhóm
2. Đặt tên file: `[YYYY-MM-DD]_Buoi-XX_[Chu-de].mp4`
3. Set quyền truy cập: "Anyone with the link can view"
4. Cập nhật link vào meeting notes

### 6.3. Quy tắc recording chung

- **Luôn thông báo** trước khi bắt đầu record
- Tắt camera/mic nếu cần nói chuyện riêng → tránh record nội dung không liên quan
- Không record phần chit-chat trước/sau buổi học (trừ khi cả nhóm đồng ý)
- Đặt tên file/folder có hệ thống để dễ tìm kiếm
- Xóa recording cũ hơn 3 tháng nếu dung lượng không đủ (sau khi hỏi ý kiến nhóm)

### 6.4. Cấu trúc folder recording

```
📁 Flutter-Training-Recordings/
├── 📁 Tuan-01-02_Basics/
│   ├── 2026-04-01_Buoi-01_Gioi-thieu-Dart-Flutter.mp4
│   ├── 2026-04-03_Buoi-02_Dart-nang-cao.mp4
│   ├── 2026-04-08_Buoi-03_Widget-Tree-co-ban.mp4
│   └── 2026-04-10_Buoi-04_Layout-System.mp4
├── 📁 Tuan-03-04_State-Architecture/
│   └── ...
├── 📁 Tuan-05-06_Network-Testing/
│   └── ...
└── 📁 Tuan-07-08_Advanced-Capstone/
    └── ...
```

---

## 7. Template Meeting Notes

Sao chép template dưới đây cho mỗi buổi sync-up:

```markdown
# Meeting Notes - Buổi [XX]: [Tên chủ đề]

📅 **Ngày:** [YYYY-MM-DD]
⏰ **Thời gian:** [HH:MM] - [HH:MM]
📍 **Hình thức:** [Online - Zoom/Meet] / [Offline - Phòng XXX]
🎥 **Recording:** [Link recording]

## Tham dự

| Member | Vai trò | Có mặt |
|--------|---------|--------|
| [Tên A] | Presenter | ✅ / ❌ |
| [Tên B] | Facilitator | ✅ / ❌ |
| [Tên C] | Note-taker | ✅ / ❌ |
| [Tên D] | Participant | ✅ / ❌ |

## 1. Review (15 phút)

### Tiến độ action items buổi trước

| Action Item | Người thực hiện | Trạng thái |
|-------------|----------------|------------|
| [Task 1] | [Tên] | ✅ Done / 🔄 In Progress / ❌ Not Started |
| [Task 2] | [Tên] | ✅ Done / 🔄 In Progress / ❌ Not Started |

### Chia sẻ nhanh từng member

- **[Tên A]:** [Nội dung chia sẻ]
- **[Tên B]:** [Nội dung chia sẻ]
- **[Tên C]:** [Nội dung chia sẻ]
- **[Tên D]:** [Nội dung chia sẻ]

## 2. Trình bày chủ đề (30 phút)

### Nội dung chính

- [Điểm chính 1]
- [Điểm chính 2]
- [Điểm chính 3]

### Key Takeaways

1. [Takeaway 1]
2. [Takeaway 2]
3. [Takeaway 3]

### Demo Code

- **Repo/Branch:** [Link đến code demo]
- **Mô tả:** [Mô tả ngắn về demo]

## 3. Thảo luận & Q&A (30 phút)

### Câu hỏi thảo luận

1. **Q:** [Câu hỏi 1]
   **A:** [Tóm tắt thảo luận]

2. **Q:** [Câu hỏi 2]
   **A:** [Tóm tắt thảo luận]

### Parking Lot (câu hỏi chưa trả lời)

- [ ] [Câu hỏi cần research thêm] → Assign: [Tên]
- [ ] [Câu hỏi cần research thêm] → Assign: [Tên]

## 4. Planning & Action Items (15 phút)

### Buổi tiếp theo

- **Chủ đề:** [Tên chủ đề]
- **Presenter:** [Tên]
- **Ngày:** [YYYY-MM-DD]

### Action Items

| # | Task | Người thực hiện | Deadline | Ghi chú |
|---|------|----------------|----------|---------|
| 1 | [Task description] | [Tên] | [YYYY-MM-DD] | |
| 2 | [Task description] | [Tên] | [YYYY-MM-DD] | |
| 3 | [Task description] | [Tên] | [YYYY-MM-DD] | |

## Feedback buổi học

| Member | Điểm (1-5) | Comment |
|--------|-----------|---------|
| [Tên A] | ⭐⭐⭐⭐ | [Comment] |
| [Tên B] | ⭐⭐⭐⭐⭐ | [Comment] |
| [Tên C] | ⭐⭐⭐ | [Comment] |
| [Tên D] | ⭐⭐⭐⭐ | [Comment] |

---
*Ghi chú bởi: [Tên Note-taker] | Cập nhật lần cuối: [YYYY-MM-DD HH:MM]*
```

---

## Phụ Lục

### A. Commitment Contract Template

```
COMMITMENT CONTRACT - FLUTTER TRAINING GROUP

Tôi, [Họ tên], cam kết:

1. Tham dự đầy đủ 16 buổi sync-up (tối đa vắng 2 buổi có phép)
2. Hoàn thành action items đúng deadline
3. Chuẩn bị kỹ khi được phân công vai trò Presenter
4. Tham gia daily check-in async trên group chat
5. Tôn trọng thời gian và ý kiến của các member khác
6. Chủ động hỗ trợ member khác khi có thể

Nếu không thể tiếp tục, tôi sẽ thông báo trước ít nhất 1 tuần
để nhóm có thời gian điều chỉnh.

Ký tên: _______________
Ngày: _______________
```

### B. Danh Sách Công Cụ Hỗ Trợ

| Mục đích | Công cụ đề xuất |
|----------|----------------|
| Video call | Zoom / Google Meet / Microsoft Teams |
| Group chat | Slack / Zalo Group / Teams Chat |
| Quản lý task | Notion / Trello / GitHub Projects |
| Chia sẻ code | GitHub repo (private) |
| Chia sẻ file | Google Drive / OneDrive |
| Ghi chú chung | Notion / HackMD / Google Docs |
| Timer | [Cuckoo Timer](https://cuckoo.team/) (online, chia sẻ được) |
| Whiteboard | Excalidraw / Miro / FigJam |

---

## 📚 Tài liệu liên quan

| Tài liệu | Mô tả |
|---|---|
| [README — Tổng quan chương trình](../README.md) | Cài đặt môi trường, lộ trình 16 buổi, hướng dẫn sử dụng |
| [Tiêu chuẩn Middle Developer](../tieu-chuan/middle-level-rubric.md) | Rubric đánh giá năng lực Middle Flutter Developer |
| [AI-Driven Development](../ai-toolkit/ai-driven-development.md) | Hướng dẫn sử dụng AI tools trong phát triển Flutter |
| [Reference Architecture](../project-mau/reference-architecture.md) | Kiến trúc tham chiếu cho dự án Flutter thực tế |

---

*Tài liệu thuộc chương trình Flutter Training. Cập nhật lần cuối: 2026-03-31.*
