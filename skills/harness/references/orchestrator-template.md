# Template Orchestrator Skill

Orchestrator là skill cấp cao điều phối toàn bộ nhóm. Cung cấp 3 template theo chế độ thực thi:

- **Template A: Chế độ nhóm agent (mặc định)** — Ưu tiên hàng đầu khi 2+ agent cộng tác
- **Template B: Chế độ subagent (phương án thay thế)** — Khi không cần giao tiếp nhóm
- **Template C: Chế độ hybrid** — Kết hợp chế độ theo từng Phase

---

## Template A: Chế độ nhóm agent (Mặc định · Ưu tiên hàng đầu)

**Chế độ cơ bản cần cân nhắc trước tiên** khi 2+ agent cộng tác. Cấu thành nhóm bằng `TeamCreate`, điều phối bằng danh sách tác vụ dùng chung và `SendMessage`.

```markdown
---
name: {domain}-orchestrator
description: "Orchestrator điều phối nhóm agent {domain}. {Từ khóa trigger lần đầu}. Tác vụ tiếp theo: Dùng skill này khi yêu cầu sửa đổi kết quả {domain}, chạy lại một phần, cập nhật, bổ sung, chạy lại, cải thiện kết quả trước."
---

# {Domain} Orchestrator

Skill tích hợp điều phối nhóm agent {domain} để tạo ra {đầu ra cuối cùng}.

## Chế độ thực thi: Nhóm agent

## Cấu thành Agent

| Thành viên | Loại agent | Vai trò | Skill | Đầu ra |
|-----------|-----------|---------|-------|--------|
| {teammate-1} | {Tùy chỉnh hoặc tích hợp sẵn} | {Vai trò} | {skill} | {output-file} |
| {teammate-2} | {Tùy chỉnh hoặc tích hợp sẵn} | {Vai trò} | {skill} | {output-file} |
| ... | | | | |

## Quy trình

### Phase 0: Kiểm tra context (Hỗ trợ tác vụ tiếp theo)

Kiểm tra sự tồn tại của đầu ra cũ để quyết định chế độ thực thi:

1. Kiểm tra sự tồn tại của thư mục `_workspace/`
2. Quyết định chế độ thực thi:
   - **`_workspace/` chưa tồn tại** → Chạy lần đầu. Tiến hành Phase 1
   - **`_workspace/` tồn tại + người dùng yêu cầu sửa một phần** → Chạy lại một phần. Chỉ gọi lại agent đó, chỉ ghi đè đầu ra cần sửa
   - **`_workspace/` tồn tại + người dùng cung cấp đầu vào mới** → Chạy mới. Di chuyển `_workspace/` hiện có sang `_workspace_{YYYYMMDD_HHMMSS}/` rồi tiến hành Phase 1
3. Khi chạy lại một phần: Bao gồm đường dẫn đầu ra trước trong prompt agent để agent đọc kết quả cũ và phản ánh phản hồi

### Phase 1: Chuẩn bị
1. Phân tích đầu vào người dùng — {Xác định gì}
2. Tạo `_workspace/` trong thư mục làm việc
   - **Chạy lần đầu**: Tạo `_workspace/` mới
   - **Chạy mới**: Di chuyển `_workspace/` hiện có sang `_workspace_{YYYYMMDD_HHMMSS}/` rồi tạo lại `_workspace/` mới
3. Lưu dữ liệu đầu vào vào `_workspace/00_input/`

### Phase 2: Cấu thành nhóm

1. Tạo nhóm:
   ```
   TeamCreate(
     team_name: "{domain}-team",
     members: [
       { name: "{teammate-1}", agent_type: "{type}", model: "opus", prompt: "{Mô tả vai trò và chỉ thị tác vụ}" },
       { name: "{teammate-2}", agent_type: "{type}", model: "opus", prompt: "{Mô tả vai trò và chỉ thị tác vụ}" },
       ...
     ]
   )
   ```

2. Đăng ký tác vụ:
   ```
   TaskCreate(tasks: [
     { title: "{Tác vụ 1}", description: "{Chi tiết}", assignee: "{teammate-1}" },
     { title: "{Tác vụ 2}", description: "{Chi tiết}", assignee: "{teammate-2}" },
     { title: "{Tác vụ 3}", description: "{Chi tiết}", depends_on: ["{Tác vụ 1}"] },
     ...
   ])
   ```

   > 5~6 tác vụ mỗi thành viên là phù hợp. Ghi rõ phụ thuộc của tác vụ bằng `depends_on`.

### Phase 3: {Tác vụ chính — v.d.: Điều tra/Tạo/Phân tích}

**Cách thực thi:** Thành viên tự điều phối

Thành viên yêu cầu và thực hiện độc lập tác vụ từ danh sách tác vụ dùng chung.
Leader giám sát tiến độ và can thiệp khi cần.

**Quy tắc giao tiếp giữa thành viên:**
- {teammate-1} truyền {thông tin gì} cho {teammate-2} qua SendMessage
- {teammate-2} lưu kết quả vào file khi hoàn thành và thông báo cho leader
- Khi thành viên cần kết quả của thành viên khác, yêu cầu qua SendMessage

**Lưu đầu ra:**

| Thành viên | Đường dẫn đầu ra |
|-----------|----------------|
| {teammate-1} | `_workspace/{phase}_{teammate-1}_{artifact}.md` |
| {teammate-2} | `_workspace/{phase}_{teammate-2}_{artifact}.md` |

**Giám sát của leader:**
- Tự động nhận thông báo khi thành viên rảnh
- Khi thành viên bị kẹt, hướng dẫn qua SendMessage hoặc phân bổ lại tác vụ
- Kiểm tra tiến độ tổng thể bằng TaskGet

### Phase 4: {Tác vụ tiếp theo — v.d.: Xác thực/Tích hợp}
1. Chờ tất cả thành viên hoàn thành tác vụ (kiểm tra trạng thái bằng TaskGet)
2. Thu thập đầu ra của từng thành viên bằng Read
3. {Logic tích hợp/xác thực}
4. Tạo đầu ra cuối cùng: `{output-path}/{filename}`

### Phase 5: Dọn dẹp
1. Yêu cầu thành viên kết thúc (SendMessage)
2. Giải thể nhóm (TeamDelete)
3. Bảo toàn thư mục `_workspace/` (không xóa đầu ra trung gian — để xác thực hậu kỳ và audit)
4. Báo cáo tóm tắt kết quả cho người dùng

> **Khi cần tái cấu trúc nhóm:** Nếu cần tổ hợp chuyên gia khác nhau theo Phase, giải thể nhóm hiện tại bằng TeamDelete rồi tạo nhóm Phase tiếp theo bằng TeamCreate mới. Đầu ra của nhóm cũ được bảo toàn trong `_workspace/` nên nhóm mới có thể truy cập bằng Read.

## Luồng dữ liệu

```
[Leader] → TeamCreate → [teammate-1] ←SendMessage→ [teammate-2]
                            │                           │
                            ↓                           ↓
                      artifact-1.md              artifact-2.md
                            │                           │
                            └───────── Read ────────────┘
                                       ↓
                                [Leader: Tích hợp]
                                       ↓
                                Đầu ra cuối cùng
```

## Xử lý lỗi

| Tình huống | Chiến lược |
|----------|-----------|
| 1 thành viên thất bại/dừng | Leader phát hiện → Kiểm tra trạng thái qua SendMessage → Khởi động lại hoặc tạo thành viên thay thế |
| Quá nửa thành viên thất bại | Thông báo người dùng và xác nhận có tiếp tục không |
| Timeout | Dùng kết quả một phần thu thập được, kết thúc thành viên chưa hoàn thành |
| Xung đột dữ liệu giữa thành viên | Ghi rõ nguồn gốc và liệt kê cả hai, không xóa |
| Trạng thái tác vụ bị trễ | Leader kiểm tra bằng TaskGet rồi thủ công TaskUpdate |

## Kịch bản kiểm thử

### Luồng bình thường
1. Người dùng cung cấp {đầu vào}
2. Phase 1 tạo ra {kết quả phân tích}
3. Phase 2 cấu thành nhóm ({N} thành viên + {M} tác vụ)
4. Phase 3 thành viên tự điều phối thực hiện tác vụ
5. Phase 4 tích hợp đầu ra tạo kết quả cuối cùng
6. Phase 5 giải thể nhóm
7. Kết quả mong đợi: Tạo được `{output-path}/{filename}`

### Luồng lỗi
1. Phase 3: {teammate-2} dừng do lỗi
2. Leader nhận thông báo rảnh
3. Kiểm tra trạng thái qua SendMessage → Thử khởi động lại
4. Nếu khởi động lại thất bại, phân bổ lại tác vụ của {teammate-2} cho {teammate-1}
5. Phase 4 tiến hành với kết quả còn lại
6. Báo cáo cuối ghi rõ "Một phần lĩnh vực {teammate-2} chưa thu thập được"
```

---

## Template B: Chế độ subagent (Phương án thay thế)

Khi không cần overhead giao tiếp nhóm. Gọi trực tiếp bằng `Agent` tool và thu thập kết quả qua return value.

```markdown
---
name: {domain}-orchestrator
description: "Orchestrator điều phối agent {domain}. {Từ khóa trigger lần đầu}. Bao gồm từ khóa tác vụ tiếp theo."
---

## Chế độ thực thi: Subagent

## Cấu thành Agent

| Agent | subagent_type | Vai trò | Skill | Đầu ra |
|-------|--------------|---------|-------|--------|
| {agent-1} | {Tích hợp sẵn hoặc tùy chỉnh} | {Vai trò} | {skill} | {output-file} |
| {agent-2} | ... | ... | ... | ... |

## Quy trình

### Phase 0: Kiểm tra context
(Giống Template A — Phân nhánh theo sự tồn tại của `_workspace/`)

### Phase 1: Chuẩn bị
1. Phân tích đầu vào
2. Tạo `_workspace/` (khi chạy lần đầu, hoặc sau khi di chuyển `_workspace/` hiện có sang thư mục lưu trữ trong lần chạy mới)

### Phase 2: Thực thi song song
Gọi N Agent tool đồng thời trong một tin nhắn:

| Agent | Đầu vào | Đầu ra | model | run_in_background |
|-------|--------|--------|-------|-------------------|
| {agent-1} | {Nguồn} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |
| {agent-2} | {Nguồn} | `_workspace/{phase}_{agent}_{artifact}.md` | opus | true |

### Phase 3: Tích hợp
1. Thu thập return value của từng agent
2. Thu thập đầu ra dựa trên file bằng Read
3. Áp dụng logic tích hợp → Đầu ra cuối cùng

### Phase 4: Dọn dẹp
1. Bảo toàn `_workspace/`
2. Báo cáo tóm tắt kết quả

## Xử lý lỗi
- 1 agent thất bại: Thử lại 1 lần. Nếu thất bại tiếp, ghi rõ thiếu sót và tiến hành
- Quá nửa thất bại: Thông báo người dùng và xác nhận có tiếp tục không
- Timeout: Dùng kết quả một phần thu thập được
```

---

## Template C: Chế độ hybrid

Dùng chế độ thực thi khác nhau theo từng Phase. Ghi rõ `**Chế độ thực thi:** {nhóm | subagent}` ở đầu mỗi Phase.

```markdown
---
name: {domain}-orchestrator
description: "Orchestrator {domain} (hybrid). {Từ khóa}. Bao gồm từ khóa tác vụ tiếp theo."
---

## Chế độ thực thi: Hybrid

| Phase | Chế độ | Lý do |
|-------|-------|-------|
| Phase 2 (Thu thập song song) | Subagent | Thu thập tài liệu độc lập, không cần giao tiếp nhóm |
| Phase 3 (Tích hợp đồng thuận) | Nhóm agent | Cần thảo luận và đạt đồng thuận về dữ liệu mâu thuẫn |
| Phase 4 (Xác thực độc lập) | Subagent | 1 QA agent xác thực khách quan |

## Quy trình

### Phase 2: Thu thập tài liệu song song
**Chế độ thực thi:** Subagent

Gọi song song N agent bằng Agent tool trong một tin nhắn (`run_in_background: true`).
Mỗi kết quả lưu vào `_workspace/02_{agent}_raw.md`.

### Phase 3: Tích hợp dựa trên đồng thuận
**Chế độ thực thi:** Nhóm agent

1. Cấu thành nhóm tích hợp bằng `TeamCreate` (editor + fact-checker + synthesizer)
2. Phân bổ tác vụ bằng `TaskCreate` — tất cả đều Read file `_workspace/02_*` từ Phase 2
3. Thành viên thảo luận về dữ liệu mâu thuẫn qua `SendMessage`, đạt đồng thuận dựa trên file
4. Tạo bản tích hợp cuối cùng `_workspace/03_integrated.md`
5. Giải thể nhóm bằng `TeamDelete`

### Phase 4: Xác thực độc lập
**Chế độ thực thi:** Subagent

1 subagent QA nhận `_workspace/03_integrated.md` làm đầu vào và tạo báo cáo xác thực.
```

**Quy tắc chuyển tiếp hybrid:**
- Nhóm → Subagent: Phải giải thể nhóm bằng `TeamDelete` trước khi gọi Agent tool
- Subagent → Nhóm: Truyền đầu ra file của subagent cho thành viên nhóm qua đường dẫn Read
- Nhóm → Nhóm: Giải thể nhóm cũ rồi tạo `TeamCreate` mới (chỉ có thể có 1 nhóm hoạt động mỗi phiên)

---

## Nguyên tắc viết

1. **Ghi rõ chế độ thực thi trước tiên** — Ghi một trong "nhóm agent" / "subagent" / "hybrid" ở đầu orchestrator. Nếu hybrid phải có bảng chế độ theo Phase
2. **Mô tả cụ thể cách dùng TeamCreate/SendMessage/TaskCreate cho chế độ nhóm** — Cấu thành nhóm, đăng ký tác vụ, quy tắc giao tiếp
3. **Ghi đầy đủ tham số Agent tool cho chế độ subagent** — name, subagent_type, prompt, run_in_background, model
4. **Đường dẫn file tuyệt đối** — Cấm đường dẫn tương đối, đường dẫn rõ ràng dựa trên `_workspace/`
5. **Ghi rõ phụ thuộc giữa Phase** — Phase nào phụ thuộc kết quả của Phase nào. Hybrid đặc biệt nhấn mạnh điểm chuyển chế độ
6. **Xử lý lỗi thực tế** — Không giả định "mọi thứ đều thành công"
7. **Kịch bản kiểm thử bắt buộc** — Ít nhất 1 bình thường + 1 lỗi

## Từ khóa tác vụ tiếp theo trong description

Description của orchestrator không đủ chỉ với từ khóa lần đầu. Phải bao gồm các cách diễn đạt tác vụ tiếp theo sau:

- Chạy lại/thực thi lại/cập nhật/sửa đổi/bổ sung
- "{domain} của {phần} làm lại"
- "Dựa trên kết quả trước", "Cải thiện kết quả"
- Yêu cầu thường ngày liên quan đến domain (v.d.: nếu là harness chiến lược launch thì "launch", "quảng bá", "trending", v.v.)

Nếu thiếu từ khóa tiếp theo, harness về cơ bản trở thành code chết sau lần chạy đầu.

## Tham khảo orchestrator thực tế

Cấu trúc cơ bản của orchestrator mẫu fan-out/fan-in:
Chuẩn bị → Phase 0 (kiểm tra context) → TeamCreate + TaskCreate → N thành viên thực thi song song → Read + tích hợp → Dọn dẹp.
Tham khảo ví dụ nhóm nghiên cứu trong `references/team-examples.md`.
