# Ví dụ cấu hình Nhóm Agent

---

## Ví dụ 1: Nhóm nghiên cứu (Chế độ nhóm agent)

### Kiến trúc nhóm: Fan-out/Fan-in
### Chế độ thực thi: Nhóm agent

```
[Leader/Orchestrator]
    ├── TeamCreate(research-team)
    ├── TaskCreate(4 tác vụ điều tra)
    ├── Thành viên tự điều phối (SendMessage)
    ├── Thu thập kết quả (Read)
    └── Tạo báo cáo tổng hợp
```

### Cấu thành agent

| Thành viên | Loại agent | Vai trò | Đầu ra |
|-----------|-----------|---------|--------|
| official-researcher | general-purpose | Tài liệu chính thức/Blog | research_official.md |
| media-researcher | general-purpose | Truyền thông/Đầu tư | research_media.md |
| community-researcher | general-purpose | Cộng đồng/MXH | research_community.md |
| background-researcher | general-purpose | Bối cảnh/Cạnh tranh/Học thuật | research_background.md |
| (leader = orchestrator) | — | Báo cáo tổng hợp | bao-cao-tong-hop.md |

> Agent nghiên cứu dùng loại tích hợp sẵn `general-purpose` nhưng phải định nghĩa bằng file `.claude/agents/{name}.md`. File chứa vai trò, phạm vi điều tra, giao thức giao tiếp nhóm để đảm bảo tính tái sử dụng và chất lượng cộng tác.

### Quy trình orchestrator (Nhóm agent)

```
Phase 1: Chuẩn bị
  - Phân tích đầu vào người dùng (chủ đề, xác định chế độ điều tra)
  - Tạo _workspace/

Phase 2: Cấu thành nhóm
  - TeamCreate(team_name: "research-team", members: [
      { name: "official", prompt: "Điều tra kênh chính thức..." },
      { name: "media", prompt: "Điều tra xu hướng truyền thông/đầu tư..." },
      { name: "community", prompt: "Điều tra phản ứng cộng đồng..." },
      { name: "background", prompt: "Điều tra bối cảnh/môi trường cạnh tranh..." }
    ])
  - TaskCreate(tasks: [
      { title: "Điều tra kênh chính thức", assignee: "official" },
      { title: "Điều tra xu hướng truyền thông", assignee: "media" },
      { title: "Điều tra phản ứng cộng đồng", assignee: "community" },
      { title: "Điều tra bối cảnh môi trường", assignee: "background" }
    ])

Phase 3: Thực hiện điều tra
  - 4 thành viên điều tra độc lập
  - Khi có phát hiện thú vị, chia sẻ giữa thành viên qua SendMessage
    (v.d.: media phát hiện tin đầu tư, truyền cho background)
  - Thảo luận trực tiếp giữa thành viên khi tìm thấy thông tin mâu thuẫn
  - Mỗi thành viên lưu file và thông báo cho leader khi hoàn thành

Phase 4: Tích hợp
  - Leader Read 4 đầu ra
  - Tạo báo cáo tổng hợp
  - Thông tin mâu thuẫn ghi rõ nguồn gốc

Phase 5: Dọn dẹp
  - Yêu cầu thành viên kết thúc
  - Giải thể nhóm
  - Bảo toàn _workspace/ (để xác thực hậu kỳ và audit)
```

### Mẫu giao tiếp nhóm

```
official ──SendMessage──→ background  (Chia sẻ thông báo chính thức liên quan)
media ────SendMessage──→ background  (Chia sẻ thông tin đầu tư/mua lại)
community ─SendMessage──→ media      (Thông tin liên quan truyền thông trong phản ứng cộng đồng)
Tất cả thành viên ──TaskUpdate──→ Danh sách tác vụ dùng chung  (Cập nhật tiến độ)
Leader ←───── Thông báo rảnh ──── Thành viên hoàn thành   (Tự động)
```

---

## Ví dụ 2: Nhóm viết tiểu thuyết SF (Chế độ nhóm agent)

### Kiến trúc nhóm: Pipeline + Fan-out
### Chế độ thực thi: Nhóm agent

```
Phase 1 (Song song — Nhóm agent): worldbuilder + character-designer + plot-architect
  → Điều phối nhất quán qua SendMessage với nhau
Phase 2 (Tuần tự): prose-stylist (viết)
Phase 3 (Song song — Nhóm agent): science-consultant + continuity-manager (review)
  → Chia sẻ phát hiện qua SendMessage với nhau
Phase 4 (Tuần tự): prose-stylist (sửa phản hồi review)
```

### Cấu thành agent

| Thành viên | Loại agent | Vai trò | Skill |
|-----------|-----------|---------|-------|
| worldbuilder | Tùy chỉnh | Xây dựng thế giới quan | world-setting |
| character-designer | Tùy chỉnh | Thiết kế nhân vật | character-profile |
| plot-architect | Tùy chỉnh | Cấu trúc cốt truyện | outline |
| prose-stylist | Tùy chỉnh | Chỉnh sửa văn phong + Viết | write-scene, review-chapter |
| science-consultant | Tùy chỉnh | Xác thực khoa học | science-check |
| continuity-manager | Tùy chỉnh | Xác thực nhất quán | consistency-check |

### Ví dụ toàn bộ file agent: `worldbuilder.md`

```markdown
---
name: worldbuilder
description: "Chuyên gia xây dựng thế giới quan tiểu thuyết SF. Thiết kế vật lý học, cấu trúc xã hội, trình độ công nghệ và lịch sử."
---

# Worldbuilder — Chuyên gia thiết kế thế giới quan SF

Bạn là chuyên gia thiết kế thế giới quan tiểu thuyết SF. Dựa trên sự thật khoa học và mở rộng trí tưởng tượng, bạn xây dựng nền tảng vật lý, xã hội và công nghệ cho câu chuyện.

## Vai trò cốt lõi
1. Định nghĩa vật lý học và trình độ công nghệ của thế giới
2. Thiết kế cấu trúc xã hội, hệ thống chính trị, kinh tế
3. Xây dựng bối cảnh lịch sử và cấu trúc xung đột hiện tại
4. Mô tả môi trường và không khí theo từng địa điểm

## Nguyên tắc tác vụ
- Nhất quán nội tại là ưu tiên hàng đầu — Không được mâu thuẫn giữa các thiết định
- Suy luận hiệu ứng lan truyền của thế giới bằng câu hỏi "Nếu có công nghệ này thì sao?"
- Thế giới quan phục vụ câu chuyện — Tránh thiết định quá mức cản trở cốt truyện

## Giao thức đầu vào/đầu ra
- Đầu vào: Concept thế giới quan của người dùng, yêu cầu thể loại
- Đầu ra: `_workspace/01_worldbuilder_setting.md`
- Định dạng: Markdown. Theo mục (Vật lý/Xã hội/Công nghệ/Lịch sử/Địa điểm)

## Giao thức giao tiếp nhóm
- Gửi cho character-designer: Cấu trúc xã hội, hệ thống giai cấp, nghề nghiệp qua SendMessage
- Gửi cho plot-architect: Cấu trúc xung đột chính, yếu tố khủng hoảng qua SendMessage
- Nhận từ science-consultant: Phản hồi lỗi khoa học → Sửa đổi thiết định
- Khi thay đổi thế giới quan, broadcast cho tất cả thành viên liên quan

## Xử lý lỗi
- Khi concept mơ hồ, đề xuất 3 hướng và yêu cầu chọn
- Khi phát hiện lỗi khoa học, cung cấp phương án thay thế cùng lúc

## Cộng tác
- Cung cấp thông tin cấu trúc xã hội cho character-designer
- Cung cấp thông tin cấu trúc xung đột cho plot-architect
- Sửa đổi thiết định theo phản hồi của science-consultant
```

### Chi tiết quy trình nhóm

```
Phase 1: TeamCreate(team_name: "novel-team", members: [worldbuilder, character-designer, plot-architect])
         TaskCreate([Xây dựng thế giới quan, Thiết kế nhân vật, Cấu trúc cốt truyện])
         → Thành viên làm việc song song trong khi tự điều phối
         → worldbuilder gửi SendMessage cho character-designer khi hoàn thành cấu trúc xã hội
         → character-designer gửi SendMessage cho plot-architect khi thiết lập nhân vật chính

Phase 2: Giải thể nhóm Phase 1 → Gọi prose-stylist bằng subagent (không cần nhóm vì viết đơn lẻ)
         prose-stylist Read 3 đầu ra trong _workspace/ và viết
         → Lưu kết quả vào _workspace/02_prose_draft.md

Phase 3: Tạo nhóm mới — TeamCreate(team_name: "review-team", members: [science-consultant, continuity-manager])
         (Một phiên chỉ có một nhóm hoạt động, nhưng đã giải thể nhóm Phase 1 nên có thể tạo nhóm mới)
         → Hai reviewer kiểm tra bản nháp, chia sẻ phát hiện với nhau
         → science-consultant phát hiện lỗi vật lý cũng thông báo cho continuity-manager
         → Giải thể nhóm sau khi review xong

Phase 4: Gọi prose-stylist bằng subagent, phản ánh kết quả review và sửa đổi cuối cùng
```

---

## Ví dụ 3: Nhóm sản xuất Webtoon (Chế độ subagent)

### Kiến trúc nhóm: Producer-Reviewer
### Chế độ thực thi: Subagent

> Trong mẫu producer-reviewer chỉ có 2 agent, và truyền kết quả quan trọng hơn giao tiếp nên subagent phù hợp.

```
Phase 1: Agent(webtoon-artist) → Tạo panel
Phase 2: Agent(webtoon-reviewer) → Kiểm tra
Phase 3: Agent(webtoon-artist) → Tạo lại panel có vấn đề (tối đa 2 lần)
```

### Cấu thành agent

| Agent | subagent_type | Vai trò | Skill |
|-------|--------------|---------|-------|
| webtoon-artist | Tùy chỉnh | Tạo ảnh panel | generate-webtoon |
| webtoon-reviewer | Tùy chỉnh | Kiểm tra chất lượng | review-webtoon, fix-webtoon-panel |

### Ví dụ toàn bộ file agent: `webtoon-reviewer.md`

```markdown
---
name: webtoon-reviewer
description: "Chuyên gia kiểm tra chất lượng panel webtoon. Đánh giá bố cục, nhất quán nhân vật, khả năng đọc văn bản và diễn xuất."
---

# Webtoon Reviewer — Chuyên gia kiểm tra chất lượng Webtoon

Bạn là chuyên gia kiểm tra chất lượng panel webtoon. Bạn đánh giá panel theo tiêu chí hoàn thiện thị giác, khả năng truyền đạt câu chuyện và nhất quán nhân vật.

## Vai trò cốt lõi
1. Đánh giá bố cục và hoàn thiện thị giác của từng panel
2. Xác thực nhất quán ngoại hình nhân vật giữa các panel
3. Đánh giá khả năng đọc và vị trí của văn bản bong bóng thoại
4. Xem xét lưu lượng diễn xuất và nhịp điệu của toàn tập

## Nguyên tắc tác vụ
- Phán định rõ ràng theo 3 mức PASS/FIX/REDO
- FIX khi có thể giải quyết bằng sửa đổi một phần, REDO khi cần tạo lại toàn bộ
- Phán định theo tiêu chí khách quan (nhất quán, khả năng đọc, bố cục) không phải sở thích cá nhân

## Giao thức đầu vào/đầu ra
- Đầu vào: Các ảnh panel trong thư mục `_workspace/panels/`
- Đầu ra: `_workspace/review_report.md`
- Định dạng:
  ```
  ## Panel {N}
  - Phán định: PASS | FIX | REDO
  - Lý do: [Lý do cụ thể]
  - Chỉ thị sửa: [Hướng sửa đổi cụ thể nếu FIX/REDO]
  ```

## Xử lý lỗi
- Khi tải ảnh thất bại, phán định panel đó là REDO
- Panel vẫn REDO sau 2 lần tạo lại, xử lý PASS kèm cảnh báo

## Cộng tác
- Truyền tài liệu chỉ thị sửa đổi cho webtoon-artist (dựa trên file kết quả)
- Kiểm tra lại panel đã tạo lại (tối đa 2 vòng lặp)
```

### Xử lý lỗi

```
Chính sách thử lại:
- Panel REDO → Yêu cầu artist tạo lại (bao gồm chỉ thị sửa đổi cụ thể)
- Tối đa 2 vòng lặp rồi ép buộc PASS
- Nếu trên 50% tổng số panel là REDO, đề xuất người dùng chỉnh sửa prompt
```

---

## Ví dụ 4: Nhóm đánh giá code (Chế độ nhóm agent)

### Kiến trúc nhóm: Fan-out/Fan-in + Thảo luận
### Chế độ thực thi: Nhóm agent

> Đánh giá code là trường hợp điển hình mà nhóm agent tỏa sáng. Các reviewer với góc nhìn khác nhau chia sẻ phát hiện và thách thức nhau để có review sâu hơn.

```
[Leader] → TeamCreate(review-team)
    ├── security-reviewer: Kiểm tra lỗ hổng bảo mật
    ├── performance-reviewer: Phân tích tác động hiệu suất
    └── test-reviewer: Xác thực test coverage
    → Reviewer chia sẻ phát hiện với nhau (SendMessage)
    → Leader tổng hợp kết quả
```

### Mẫu giao tiếp nhóm

```
security ──SendMessage──→ performance  ("SQL injection có thể xảy ra, cần kiểm tra cả góc hiệu suất")
performance ──SendMessage──→ test      ("Phát hiện N+1 query, kiểm tra xem có test liên quan không")
test ────SendMessage──→ security      ("Không có test cho module xác thực, ưu tiên về bảo mật?")
```

Cốt lõi: Reviewer giao tiếp **trực tiếp mà không qua leader** để nhanh chóng nắm bắt vấn đề vùng giao nhau.

---

## Ví dụ 5: Mẫu Supervisor — Nhóm migration code (Chế độ nhóm agent)

### Kiến trúc nhóm: Supervisor
### Chế độ thực thi: Nhóm agent

```
[supervisor/leader] → Phân tích danh sách file → Phân bổ batch
    ├→ [migrator-1] (batch A)
    ├→ [migrator-2] (batch B)
    └→ [migrator-3] (batch C)
    ← Nhận TaskUpdate → Phân bổ batch tiếp theo hoặc phân bổ lại
```

### Cấu thành agent

| Thành viên | Vai trò |
|-----------|---------|
| (leader = migration-supervisor) | Phân tích file, phân phối batch, quản lý tiến độ |
| migrator-1~3 | Migration batch file được phân công |

### Logic phân phối động của supervisor (Sử dụng nhóm agent)

```
1. Thu thập danh sách file đích toàn bộ
2. Ước tính độ phức tạp (kích thước file, số import, dependency)
3. Đăng ký batch file làm tác vụ bằng TaskCreate (bao gồm dependency)
4. Thành viên tự yêu cầu (claim) tác vụ
5. Khi thành viên báo cáo hoàn thành qua TaskUpdate:
   - Thành công → Tự động yêu cầu tác vụ tiếp theo
   - Thất bại → Leader xác nhận nguyên nhân qua SendMessage → Phân bổ lại hoặc giao cho thành viên khác
6. Tất cả tác vụ hoàn thành → Leader chạy integration test
```

Khác Fan-out: Tác vụ không cố định trước mà được **phân bổ động tại runtime**. Tính năng tự yêu cầu (claim) của danh sách tác vụ dùng chung phù hợp tự nhiên với mẫu supervisor.

---

## Tóm tắt mẫu đầu ra

### File định nghĩa agent
Vị trí: `dự-án/.claude/agents/{agent-name}.md`
Phần bắt buộc: Vai trò cốt lõi, nguyên tắc tác vụ, giao thức đầu vào/đầu ra, xử lý lỗi, cộng tác
Phần bổ sung chế độ nhóm: **Giao thức giao tiếp nhóm** (nguồn/đích nhận/gửi tin nhắn, phạm vi yêu cầu tác vụ)

### Cấu trúc file skill
Vị trí: `dự-án/.claude/skills/{skill-name}/SKILL.md` (cấp độ dự án)
Hoặc: `~/.claude/skills/{skill-name}/SKILL.md` (cấp độ toàn cục)

### Skill tích hợp (Orchestrator)
Skill cấp cao điều phối toàn bộ nhóm. Định nghĩa cấu thành agent và quy trình theo từng kịch bản.
Template: Tham khảo `references/orchestrator-template.md`.
**Ghi rõ chế độ thực thi** — Nhóm agent (mặc định) hoặc subagent.
