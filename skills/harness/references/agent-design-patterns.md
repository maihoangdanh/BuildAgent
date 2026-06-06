# Mẫu Thiết kế Nhóm Agent

## Chế độ thực thi: Nhóm Agent vs Subagent

Hiểu sự khác biệt cốt lõi giữa hai chế độ thực thi và chọn chế độ phù hợp.

### Nhóm Agent (Agent Teams) — Chế độ mặc định

Leader tạo nhóm bằng `TeamCreate`, các thành viên chạy như các instance Claude Code độc lập. Thành viên giao tiếp trực tiếp qua `SendMessage` và tự điều phối bằng danh sách tác vụ dùng chung (`TaskCreate`/`TaskUpdate`).

```
[Leader] ←→ [Thành viên A] ←→ [Thành viên B]
  ↕              ↕                 ↕
  └──── Danh sách tác vụ dùng chung ────┘
```

**Công cụ cốt lõi:**
- `TeamCreate`: Tạo nhóm + spawn thành viên
- `SendMessage({to: name})`: Gửi tin nhắn đến thành viên cụ thể
- `SendMessage({to: "all"})`: Broadcast (chi phí cao, dùng ít thôi)
- `TaskCreate`/`TaskUpdate`: Quản lý danh sách tác vụ dùng chung

**Đặc điểm:**
- Thành viên có thể trò chuyện, thách thức, xác thực lẫn nhau trực tiếp
- Trao đổi thông tin giữa thành viên mà không qua leader
- Tự điều phối bằng danh sách tác vụ dùng chung (có thể tự yêu cầu tác vụ)
- Thành viên rảnh rỗi tự động thông báo cho leader
- Chế độ phê duyệt kế hoạch cho phép review trước tác vụ nguy hiểm

**Hạn chế:**
- Chỉ có thể **hoạt động** một nhóm mỗi phiên (nhưng có thể giải thể và tạo nhóm mới giữa các Phase)
- Không thể lồng nhóm (thành viên không thể tạo nhóm của mình)
- Leader cố định (không chuyển được)
- Chi phí token cao

**Mẫu tái cấu trúc nhóm:**
Khi cần tổ hợp chuyên gia khác nhau theo Phase, hãy lưu đầu ra của nhóm cũ vào file → giải thể nhóm → tạo nhóm mới. Đầu ra của nhóm cũ được bảo toàn trong `_workspace/` nên nhóm mới có thể truy cập bằng Read.

### Subagent — Chế độ nhẹ

Agent chính tạo subagent bằng `Agent` tool. Subagent chỉ trả kết quả tác vụ về cho agent chính, không giao tiếp với nhau.

```
[Main] → [Sub A] → Trả kết quả
      → [Sub B] → Trả kết quả
      → [Sub C] → Trả kết quả
```

**Công cụ cốt lõi:**
- `Agent(prompt, subagent_type, run_in_background)`: Tạo subagent

**Đặc điểm:**
- Nhẹ và nhanh
- Kết quả được trả về tóm tắt cho context chính
- Tiết kiệm token

**Hạn chế:**
- Subagent không thể giao tiếp với nhau
- Main đảm nhận toàn bộ điều phối
- Không thể cộng tác/thách thức thời gian thực

### Cây quyết định chọn chế độ

```
Có 2+ agent không?
├── Có → Cần giao tiếp giữa các agent không?
│         ├── Có → Nhóm agent (mặc định)
│         │         Nâng cao chất lượng qua xác thực chéo, chia sẻ phát hiện, phản hồi thời gian thực.
│         │
│         └── Không → Subagent cũng được
│                  Khi chỉ cần truyền kết quả như producer-reviewer, expert pool.
│
└── Không (1 agent) → Subagent
              Tác vụ agent đơn lẻ không cần cấu thành nhóm.
```

> **Nguyên tắc cốt lõi:** Nhóm agent là mặc định. Khi chọn subagent, hãy tự hỏi "Giao tiếp giữa thành viên có thực sự không cần thiết không?".

---

## Loại kiến trúc nhóm agent

### 1. Pipeline
Luồng tác vụ tuần tự. Đầu ra của agent trước là đầu vào của agent tiếp theo.

```
[Phân tích] → [Thiết kế] → [Triển khai] → [Xác thực]
```

**Phù hợp khi:** Mỗi bước phụ thuộc mạnh vào đầu ra của bước trước
**Ví dụ:** Viết tiểu thuyết — Thế giới quan → Nhân vật → Cốt truyện → Viết → Chỉnh sửa
**Lưu ý:** Điểm nghẽn làm chậm toàn bộ pipeline. Thiết kế mỗi bước độc lập nhất có thể.
**Phù hợp chế độ nhóm:** Phụ thuộc tuần tự mạnh nên lợi ích nhóm hạn chế. Tuy nhiên nhóm hữu ích nếu có đoạn song song trong pipeline.

### 2. Fan-out/Fan-in
Xử lý song song rồi tích hợp kết quả. Thực hiện đồng thời các tác vụ độc lập.

```
           ┌→ [Chuyên gia A] ─┐
[Phân phối] → ├→ [Chuyên gia B] ─┼→ [Tích hợp]
           └→ [Chuyên gia C] ─┘
```

**Phù hợp khi:** Cần phân tích từ nhiều góc độ/lĩnh vực khác nhau cho cùng một đầu vào
**Ví dụ:** Nghiên cứu tổng hợp — Điều tra đồng thời nguồn chính thức/truyền thông/cộng đồng/bối cảnh → Báo cáo tích hợp
**Lưu ý:** Chất lượng giai đoạn tích hợp quyết định chất lượng tổng thể.
**Phù hợp chế độ nhóm:** Mẫu tự nhiên nhất của nhóm agent. **Phải cấu thành bằng nhóm agent.** Thành viên chia sẻ phát hiện và thách thức nhau, phát hiện của một agent có thể điều chỉnh hướng điều tra của agent khác theo thời gian thực, giúp nâng cao chất lượng đáng kể so với điều tra độc lập.

### 3. Expert Pool
Gọi chuyên gia phù hợp có chọn lọc tùy ngữ cảnh.

```
[Router] → { Chuyên gia A | Chuyên gia B | Chuyên gia C }
```

**Phù hợp khi:** Cần xử lý khác nhau tùy loại đầu vào
**Ví dụ:** Đánh giá code — Chỉ gọi chuyên gia bảo mật/hiệu suất/kiến trúc cho lĩnh vực liên quan
**Lưu ý:** Độ chính xác phân loại của router là yếu tố then chốt.
**Phù hợp chế độ nhóm:** Subagent phù hợp hơn. Không cần nhóm thường trực vì chỉ gọi chuyên gia cần thiết.

### 4. Producer-Reviewer
Agent tạo và agent xác thực hoạt động theo cặp.

```
[Tạo] → [Xác thực] → (nếu có vấn đề) → [Tạo] chạy lại
```

**Phù hợp khi:** Đảm bảo chất lượng đầu ra quan trọng và có tiêu chí xác thực khách quan
**Ví dụ:** Webtoon — artist tạo → reviewer kiểm tra → tạo lại panel có vấn đề
**Lưu ý:** Phải đặt số lần thử lại tối đa (2~3 lần) để tránh vòng lặp vô tận.
**Phù hợp chế độ nhóm:** Nhóm agent hữu ích. Trao đổi phản hồi thời gian thực giữa người tạo↔người xác thực qua SendMessage.

### 5. Supervisor
Agent trung tâm quản lý trạng thái tác vụ và phân phối động cho agent cấp dưới.

```
              ┌→ [Worker A]
[Supervisor] ─┼→ [Worker B]    ← Supervisor phân phối động dựa trên trạng thái
              └→ [Worker C]
```

**Phù hợp khi:** Khối lượng tác vụ có thể thay đổi hoặc cần quyết định phân phối tác vụ tại runtime
**Ví dụ:** Migration code quy mô lớn — supervisor phân tích danh sách file và phân bổ batch cho worker
**Khác Fan-out:** Fan-out phân phối tác vụ cố định trước, supervisor điều chỉnh động theo tiến độ
**Lưu ý:** Đặt đơn vị ủy quyền đủ lớn để supervisor không trở thành điểm nghẽn.
**Phù hợp chế độ nhóm:** Danh sách tác vụ dùng chung của nhóm agent phù hợp tự nhiên với mẫu supervisor. Đăng ký tác vụ bằng TaskCreate, thành viên tự yêu cầu.

### 6. Hierarchical Delegation
Agent cấp trên ủy quyền đệ quy cho cấp dưới. Phân rã vấn đề phức tạp theo từng bước.

```
[Tổng quản] → [Trưởng nhóm A] → [Nhân viên A1]
                              → [Nhân viên A2]
            → [Trưởng nhóm B] → [Nhân viên B1]
```

**Phù hợp khi:** Vấn đề phân rã tự nhiên theo cấu trúc phân cấp
**Ví dụ:** Phát triển ứng dụng full-stack — tổng quản → trưởng nhóm frontend → (UI/logic/test) + trưởng nhóm backend → (API/DB/test)
**Lưu ý:** Độ sâu trên 3 tầng dẫn đến trễ và mất context lớn. Khuyến nghị không quá 2 tầng.
**Phù hợp chế độ nhóm:** Nhóm agent không thể lồng nhau (thành viên không thể tạo nhóm). Triển khai tầng 1 bằng nhóm, tầng 2 bằng subagent, hoặc san phẳng thành một nhóm duy nhất.

## Mẫu kết hợp

Trong thực tế, mẫu kết hợp phổ biến hơn mẫu đơn lẻ:

| Mẫu kết hợp | Cấu trúc | Ví dụ |
|------------|---------|-------|
| **Fan-out + Producer-Reviewer** | Tạo song song rồi xác thực từng cái | Dịch đa ngôn ngữ — dịch 4 ngôn ngữ song song → mỗi cái được native reviewer kiểm tra |
| **Pipeline + Fan-out** | Song song hóa một số bước tuần tự | Phân tích (tuần tự) → Triển khai (song song) → Test tích hợp (tuần tự) |
| **Supervisor + Expert Pool** | Supervisor gọi chuyên gia động | Xử lý câu hỏi khách hàng — supervisor phân loại rồi phân công chuyên gia phù hợp |

### Chế độ thực thi trong mẫu kết hợp

**Mặc định dùng nhóm agent cho tất cả mẫu kết hợp.** Giao tiếp tích cực giữa thành viên là động lực cốt lõi của chất lượng kết quả.

| Kịch bản | Chế độ khuyến nghị | Lý do |
|---------|-------------------|-------|
| **Nghiên cứu + Phân tích** | Nhóm agent | Chia sẻ phát hiện giữa điều tra viên, thảo luận thời gian thực về thông tin mâu thuẫn |
| **Thiết kế + Triển khai + Xác thực** | Nhóm agent | Vòng phản hồi giữa thiết kế sư↔lập trình viên↔người xác thực |
| **Supervisor + Worker** | Nhóm agent | Phân bổ động bằng danh sách tác vụ dùng chung, chia sẻ tiến độ giữa worker |
| **Tạo + Xác thực** | Nhóm agent | Tối thiểu hóa làm lại nhờ phản hồi thời gian thực giữa người tạo↔người xác thực |

> Chỉ cân nhắc kết hợp subagent khi một agent đơn lẻ thực hiện tác vụ đơn lẻ hoàn toàn bị cô lập.

## Chọn loại Agent

Khi gọi agent, chỉ định loại qua tham số `subagent_type` của Agent tool. Thành viên nhóm agent cũng có thể dùng định nghĩa agent tùy chỉnh.

### Loại tích hợp sẵn

| Loại | Quyền truy cập công cụ | Phù hợp cho |
|------|----------------------|------------|
| `general-purpose` | Đầy đủ (gồm WebSearch, WebFetch) | Nghiên cứu web, tác vụ đa năng |
| `Explore` | Chỉ đọc (không có Edit/Write) | Khám phá codebase, phân tích |
| `Plan` | Chỉ đọc (không có Edit/Write) | Thiết kế kiến trúc, lập kế hoạch |

### Loại tùy chỉnh

Định nghĩa agent trong `.claude/agents/{name}.md` để gọi bằng `subagent_type: "{name}"`. Agent tùy chỉnh có quyền truy cập đầy đủ công cụ.

### Tiêu chí chọn

| Tình huống | Khuyến nghị | Lý do |
|-----------|------------|-------|
| Vai trò phức tạp, tái sử dụng nhiều phiên | **Loại tùy chỉnh** (`.claude/agents/`) | Quản lý persona và nguyên tắc tác vụ qua file |
| Điều tra/thu thập đơn giản, prompt đủ | **`general-purpose`** + prompt chi tiết | Không cần file agent, bao gồm chỉ thị trong prompt |
| Chỉ cần đọc code (phân tích/review) | **`Explore`** | Ngăn vô tình sửa file |
| Chỉ cần thiết kế/lập kế hoạch | **`Plan`** | Tập trung phân tích, ngăn thay đổi code |
| Tác vụ triển khai cần sửa file | **Loại tùy chỉnh** | Quyền truy cập đầy đủ công cụ + chỉ thị chuyên biệt |

**Nguyên tắc:** Tất cả agent phải được định nghĩa bằng file `.claude/agents/{name}.md`. Ngay cả loại tích hợp sẵn cũng phải tạo file định nghĩa agent để ghi rõ vai trò, nguyên tắc, giao thức. File phải tồn tại để tái sử dụng trong phiên tiếp theo, và giao thức giao tiếp nhóm phải ghi rõ để đảm bảo chất lượng cộng tác.

**Model:** Tất cả agent dùng `model: "opus"`. Khi gọi Agent tool phải ghi rõ tham số `model: "opus"`.

## Cấu trúc định nghĩa Agent

```markdown
---
name: agent-name
description: "Mô tả vai trò 1-2 câu. Liệt kê từ khóa trigger."
---

# Agent Name — Tóm tắt vai trò trong một dòng

Bạn là chuyên gia [vai trò] trong lĩnh vực [domain].

## Vai trò cốt lõi
1. Vai trò 1
2. Vai trò 2

## Nguyên tắc tác vụ
- Nguyên tắc 1
- Nguyên tắc 2

## Giao thức đầu vào/đầu ra
- Đầu vào: [Nhận gì từ đâu]
- Đầu ra: [Ghi gì ở đâu]
- Định dạng: [Định dạng file, cấu trúc]

## Giao thức giao tiếp nhóm (chế độ nhóm agent)
- Nhận tin nhắn: [Nhận tin nhắn gì từ ai]
- Gửi tin nhắn: [Gửi tin nhắn gì cho ai]
- Yêu cầu tác vụ: [Loại tác vụ nào yêu cầu từ danh sách tác vụ dùng chung]

## Xử lý lỗi
- [Hành động khi thất bại]
- [Hành động khi timeout]

## Cộng tác
- Quan hệ với các agent khác
```

## Tiêu chí phân tách Agent

| Tiêu chí | Phân tách | Hợp nhất |
|---------|----------|---------|
| Chuyên môn | Phân tách nếu lĩnh vực khác nhau | Hợp nhất nếu lĩnh vực chồng chéo |
| Song song | Phân tách nếu có thể chạy độc lập | Cân nhắc hợp nhất nếu phụ thuộc tuần tự |
| Context | Phân tách nếu context nặng | Hợp nhất nếu nhẹ và nhanh |
| Tái sử dụng | Phân tách nếu dùng trong nhóm khác | Cân nhắc hợp nhất nếu chỉ dùng trong nhóm này |

## Thiết kế tái sử dụng Agent

Trước khi tạo agent mới, kiểm tra trùng lặp với agent hiện có. Khi xây harness lặp đi lặp lại, dễ tích lũy các agent trùng vai trò dưới các tên khác nhau.

| Tình huống | Hành động |
|----------|----------|
| Agent hiện có bao gồm hoàn toàn vai trò mới | Cấm tạo mới — Tái sử dụng agent hiện có |
| Agent hiện có bao gồm một phần và có thể tổng quát hóa | Tổng quát hóa và mở rộng agent hiện có |
| Bao gồm một phần là domain đặc hóa có chủ ý | Tiến hành tạo mới — Giữ làm agent riêng biệt |
| Phạm vi vai trò hoàn toàn khác | Tiến hành tạo mới |

**Nguyên tắc:** Agent càng tập trung vào một vai trò, khả năng tái sử dụng càng cao và trùng lặp càng giảm. Nếu có hai vai trò trở lên, trước tiên cân nhắc xem có thể phân tách không.

**Khi tổng quát hóa agent hiện có:** Hành vi của orchestrator/cấu thành nhóm phụ thuộc vào agent đó có thể thay đổi. Kiểm tra dependency trước khi mở rộng, và xác nhận hành vi hiện có được duy trì bằng dry-run sau khi tổng quát hóa.

## Phân biệt Skill vs Agent

| Phân loại | Skill | Agent |
|----------|-------|-------|
| Định nghĩa | Kiến thức quy trình + bộ công cụ | Persona chuyên gia + nguyên tắc hành vi |
| Vị trí | `.claude/skills/` | `.claude/agents/` |
| Trigger | Khớp từ khóa yêu cầu người dùng | Gọi rõ ràng bằng Agent tool |
| Kích thước | Nhỏ~Lớn (quy trình) | Nhỏ (định nghĩa vai trò) |
| Mục đích | "Làm như thế nào" | "Ai làm" |

Skill là **hướng dẫn quy trình** mà agent tham chiếu khi thực hiện tác vụ.
Agent là **định nghĩa vai trò chuyên gia** sử dụng skill.

## Cách kết nối Skill ↔ Agent

3 cách agent sử dụng skill:

| Cách | Triển khai | Phù hợp khi |
|-----|-----------|------------|
| **Gọi Skill tool** | Ghi rõ `Gọi /skill-name bằng Skill tool` trong prompt agent | Skill là quy trình độc lập và người dùng có thể gọi |
| **Inline trong prompt** | Bao gồm trực tiếp nội dung skill trong định nghĩa agent | Skill ngắn (dưới 50 dòng) và dành riêng cho agent này |
| **Load reference** | Load file `references/` của skill bằng `Read` khi cần | Nội dung skill lớn và chỉ cần có điều kiện |

Khuyến nghị: Tái sử dụng cao dùng Skill tool, dành riêng dùng inline, dung lượng lớn dùng load reference.
