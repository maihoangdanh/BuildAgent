---
name: harness
description: "Cấu hình harness. Meta-skill định nghĩa các agent chuyên biệt và tạo skill mà các agent đó sử dụng. Dùng khi: (1) yêu cầu 'xây harness', 'cấu hình harness'; (2) yêu cầu 'thiết kế harness', 'kỹ thuật harness'; (3) xây dựng hệ thống tự động hóa dựa trên harness cho domain/dự án mới; (4) tái cấu hình hoặc mở rộng cấu hình harness hiện có; (5) yêu cầu vận hành/bảo trì harness hiện có như 'kiểm tra harness', 'kiểm toán harness', 'hiện trạng harness', 'đồng bộ agent/skill'."
---

# Harness — Kiến trúc sư Nhóm Agent & Skill

Meta-skill cấu hình harness phù hợp với domain/dự án, định nghĩa vai trò của từng agent và tạo skill mà agent sử dụng.

**Nguyên tắc cốt lõi:**
1. Tạo định nghĩa agent (`.claude/agents/`) và skill (`.claude/skills/`).
2. **Sử dụng nhóm agent làm chế độ thực thi mặc định.**
3. **Đăng ký pointer harness vào CLAUDE.md.** — Chỉ ghi pointer tối thiểu (quy tắc trigger + lịch sử thay đổi) để orchestrator skill được kích hoạt trong phiên mới.
4. **Harness không phải thứ cố định mà là hệ thống tiến hóa.** — Sau mỗi lần chạy, phản ánh phản hồi và liên tục cập nhật agent, skill, CLAUDE.md.

## Quy trình

### Phase 0: Kiểm tra hiện trạng

Khi harness skill được trigger, đầu tiên kiểm tra trạng thái harness hiện có.

1. Đọc `dự-án/.claude/agents/`, `dự-án/.claude/skills/`, `dự-án/CLAUDE.md`
2. Phân nhánh chế độ thực thi dựa trên hiện trạng:
   - **Xây mới**: Thư mục agent/skill không tồn tại hoặc rỗng → Thực thi toàn bộ từ Phase 1
   - **Mở rộng hiện có**: Harness hiện có và yêu cầu thêm agent/skill mới → Chỉ thực thi Phase cần thiết theo ma trận chọn Phase bên dưới
   - **Vận hành/Bảo trì**: Yêu cầu kiểm toán/sửa đổi/đồng bộ harness hiện có → Chuyển sang quy trình vận hành/bảo trì Phase 7-5

   **Ma trận chọn Phase khi mở rộng hiện có:**
   | Loại thay đổi | Phase 1 | Phase 2 | Phase 3 | Phase 4 | Phase 5 | Phase 6 |
   |--------------|---------|---------|---------|---------|---------|---------|
   | Thêm agent | Bỏ qua (dùng kết quả Phase 0) | Chỉ quyết định vị trí | Bắt buộc (gồm 3-0) | Nếu cần skill riêng (gồm 4-0) | Sửa orchestrator | Bắt buộc |
   | Thêm/sửa skill | Bỏ qua | Bỏ qua | Bỏ qua | Bắt buộc (gồm 4-0) | Nếu kết nối thay đổi | Bắt buộc |
   | Thay đổi kiến trúc | Bỏ qua | Bắt buộc | Chỉ agent bị ảnh hưởng (gồm 3-0) | Chỉ skill bị ảnh hưởng (gồm 4-0) | Bắt buộc | Bắt buộc |
3. Đối chiếu danh sách agent/skill hiện có với hồ sơ CLAUDE.md để phát hiện không nhất quán (drift)
4. Báo cáo tóm tắt kết quả kiểm toán cho người dùng và xác nhận kế hoạch thực thi

### Phase 1: Phân tích Domain
1. Xác định domain/dự án từ yêu cầu người dùng
2. Nhận diện các loại tác vụ cốt lõi (tạo, xác thực, chỉnh sửa, phân tích, v.v.)
3. Phân tích xung đột/trùng lặp với agent/skill hiện có dựa trên kết quả kiểm toán Phase 0
4. Khám phá codebase dự án — nắm bắt tech stack, mô hình dữ liệu, module chính
5. **Phát hiện trình độ người dùng** — Dựa vào manh mối ngữ cảnh trong hội thoại (thuật ngữ sử dụng, mức độ câu hỏi) để nắm bắt trình độ kỹ thuật và điều chỉnh tone giao tiếp cho phù hợp. Không dùng thuật ngữ như "assertion", "JSON schema" mà không giải thích cho người dùng ít kinh nghiệm coding.

### Phase 2: Thiết kế Kiến trúc Nhóm

#### 2-1. Chọn chế độ thực thi

**Nhóm agent là giá trị mặc định ưu tiên hàng đầu.** Khi 2 agent trở lên cộng tác, nhất thiết phải cân nhắc nhóm agent trước. Các thành viên tự điều phối thông qua giao tiếp trực tiếp (SendMessage) và danh sách tác vụ dùng chung (TaskCreate), việc chia sẻ phát hiện, tranh luận về mâu thuẫn và bổ sung thiếu sót giúp nâng cao chất lượng đầu ra.

| Chế độ | Khi nào dùng | Đặc điểm |
|--------|-------------|----------|
| **Nhóm agent** (mặc định) | 2+ agent cộng tác, cần điều phối và trao đổi phản hồi thời gian thực, tham chiếu chéo đầu ra trung gian | Tự điều phối bằng `TeamCreate` + `SendMessage` + `TaskCreate` |
| **Subagent** (phương án thay thế) | Tác vụ agent đơn lẻ, chỉ cần trả kết quả về main, overhead giao tiếp nhóm quá lớn | Gọi trực tiếp `Agent` tool, song song với `run_in_background` |
| **Hybrid** | Khi các Phase có đặc điểm khác nhau — v.d.: thu thập song song (subagent) → tích hợp dựa trên đồng thuận (nhóm) | Kết hợp nhóm/subagent theo Phase |

**Thứ tự ra quyết định:**
1. Trước tiên kiểm tra có thể thiết kế với nhóm agent không — mặc định nếu có 2+ agent
2. Chỉ chọn subagent khi giao tiếp nhóm không cần thiết về mặt cấu trúc (chỉ cần truyền kết quả) và overhead nhóm lớn hơn lợi ích
3. Cân nhắc hybrid nếu đặc điểm từng Phase khác nhau rõ rệt — ghi rõ chế độ thực thi của mỗi Phase trong orchestrator

> Chi tiết bảng so sánh và cây quyết định theo mẫu trong `references/agent-design-patterns.md` phần "Chế độ thực thi".

#### 2-2. Chọn mẫu kiến trúc

1. Phân rã tác vụ theo lĩnh vực chuyên môn
2. Quyết định cấu trúc nhóm agent (tham khảo mẫu kiến trúc trong `references/agent-design-patterns.md`)
   - **Pipeline**: Tác vụ tuần tự phụ thuộc nhau
   - **Fan-out/Fan-in**: Tác vụ song song độc lập
   - **Expert Pool**: Gọi có chọn lọc theo ngữ cảnh
   - **Producer-Reviewer**: Tạo nội dung rồi kiểm tra chất lượng
   - **Supervisor**: Agent trung tâm quản lý trạng thái và phân phối tác vụ động
   - **Hierarchical Delegation**: Agent cấp trên ủy quyền đệ quy cho cấp dưới

#### 2-3. Tiêu chí phân tách agent

Đánh giá theo 4 trục: chuyên môn, song song, context, tái sử dụng. Chi tiết bảng tiêu chí trong `references/agent-design-patterns.md` phần "Tiêu chí phân tách agent". Kiểm tra trùng lặp/tái sử dụng với agent hiện có được xử lý trong Phase 3-0.

### Phase 3: Tạo Định nghĩa Agent

#### 3-0. Kiểm tra trùng lặp agent hiện có

Trước khi tạo agent mới, kiểm tra trùng lặp với agent hiện có trong `dự-án/.claude/agents/`. Khi xây harness lặp đi lặp lại, dễ tích lũy các agent trùng vai trò dưới các tên khác nhau.

> Tiêu chí phân loại trùng lặp và thiết kế tái sử dụng trong `references/agent-design-patterns.md` phần "Thiết kế tái sử dụng agent".

**Tất cả agent phải được định nghĩa trong file `dự-án/.claude/agents/{name}.md`.** Nghiêm cấm đặt vai trò trực tiếp vào prompt của Agent tool mà không có file định nghĩa agent. Lý do:
- File định nghĩa agent phải tồn tại để có thể tái sử dụng trong phiên tiếp theo
- Giao thức giao tiếp nhóm phải được ghi rõ để đảm bảo chất lượng cộng tác liên agent
- Giá trị cốt lõi của harness là sự phân tách giữa agent (ai làm) và skill (làm như thế nào)

Ngay cả khi dùng loại tích hợp sẵn (`general-purpose`, `Explore`, `Plan`), vẫn phải tạo file định nghĩa agent. Loại tích hợp sẵn được chỉ định qua tham số `subagent_type` của Agent tool, còn file định nghĩa agent chứa vai trò, nguyên tắc và giao thức.

**Cài đặt model:** Tất cả agent dùng `model: "opus"`. Khi gọi Agent tool phải ghi rõ tham số `model: "opus"`. Chất lượng harness gắn trực tiếp với khả năng suy luận của agent, và opus đảm bảo chất lượng cao nhất.

**Tái cấu trúc nhóm:** Chỉ có thể có một nhóm hoạt động mỗi phiên, nhưng có thể giải thể và tạo nhóm mới giữa các Phase. Nếu mẫu pipeline cần tổ hợp chuyên gia khác nhau theo Phase, hãy lưu đầu ra của nhóm cũ vào file, giải thể nhóm, rồi tạo nhóm mới.

Định nghĩa mỗi agent trong `dự-án/.claude/agents/{name}.md`. Phần bắt buộc: vai trò cốt lõi, nguyên tắc tác vụ, giao thức đầu vào/đầu ra, xử lý lỗi, cộng tác. Trong chế độ nhóm agent, thêm mục `## Giao thức giao tiếp nhóm` để ghi rõ nguồn/đích nhận/gửi tin nhắn và phạm vi yêu cầu tác vụ.

> Template định nghĩa và toàn bộ file thực tế trong `references/agent-design-patterns.md` phần "Cấu trúc định nghĩa agent" + `references/team-examples.md`.

**Yêu cầu bắt buộc khi bao gồm QA agent:**
- QA agent dùng loại `general-purpose` (`Explore` chỉ đọc nên không thể chạy script xác thực)
- Cốt lõi của QA là **"so sánh chéo ranh giới"** không phải "kiểm tra sự tồn tại" — đọc API response và frontend hook đồng thời, so sánh shape
- QA không chỉ chạy một lần sau khi hoàn tất toàn bộ mà **chạy tăng dần ngay sau khi mỗi module hoàn thành** (incremental QA)
- Hướng dẫn chi tiết: `references/qa-agent-guide.md`

### Phase 4: Tạo Skill

Tạo skill mà mỗi agent sử dụng tại `dự-án/.claude/skills/{name}/SKILL.md`. Hướng dẫn viết chi tiết trong `references/skill-writing-guide.md`.

#### 4-0. Kiểm tra trùng lặp skill hiện có

Trước khi tạo skill mới, kiểm tra trùng lặp với skill hiện có trong `dự-án/.claude/skills/`. Khi xây harness lặp đi lặp lại, dễ tích lũy các skill chức năng trùng nhau dưới các tên khác nhau.

> Tiêu chí phân loại trùng lặp và mẫu tổng quát hóa trong `references/skill-writing-guide.md` phần "Thiết kế tái sử dụng skill".

#### 4-1. Cấu trúc Skill

```
skill-name/
├── SKILL.md (bắt buộc)
│   ├── YAML frontmatter (bắt buộc name, description)
│   └── Nội dung Markdown
└── Tài nguyên đóng gói (tùy chọn)
    ├── scripts/    - Code thực thi cho tác vụ lặp lại/xác định
    ├── references/ - Tài liệu tham khảo load có điều kiện
    └── assets/     - File dùng trong đầu ra (template, ảnh, v.v.)
```

#### 4-2. Viết Description — Khuyến khích trigger tích cực

Description là cơ chế trigger duy nhất của skill. Claude có xu hướng đánh giá trigger một cách bảo thủ, vì vậy hãy viết description một cách **tích cực ("pushy")**.

**Ví dụ xấu:** `"Skill xử lý tài liệu PDF"`
**Ví dụ tốt:** `"Thực hiện tất cả tác vụ PDF bao gồm đọc, trích xuất văn bản/bảng, hợp nhất, tách, xoay, watermark, mã hóa, OCR. Khi nhắc đến file .pdf hoặc yêu cầu đầu ra PDF, phải dùng skill này."`

Cốt lõi: Mô tả cả những gì skill làm + tình huống trigger cụ thể, và phân biệt với các trường hợp tương tự nhưng không nên trigger.

#### 4-3. Nguyên tắc viết nội dung

| Nguyên tắc | Giải thích |
|-----------|-----------|
| **Giải thích tại sao** | Thay vì chỉ thị cưỡng bức như "ALWAYS/NEVER", truyền đạt lý do tại sao phải làm như vậy. LLM hiểu lý do sẽ phán đoán đúng ngay cả trong trường hợp edge case. |
| **Giữ gọn nhẹ** | Context window là tài nguyên chung. Hướng đến dưới 500 dòng cho nội dung SKILL.md, xóa hoặc chuyển sang references/ những gì không tạo thêm giá trị. |
| **Tổng quát hóa** | Thay vì quy tắc hẹp chỉ khớp với ví dụ cụ thể, giải thích nguyên lý để đáp ứng được đa dạng đầu vào. Cấm overfitting. |
| **Đóng gói code lặp lại** | Khi các agent thường xuyên viết cùng một script trong quá trình kiểm thử, hãy đóng gói trước vào `scripts/`. |
| **Viết theo lệnh** | Dùng dạng mệnh lệnh/chỉ thị "~làm điều này", "~hãy làm". |

#### 4-4. Progressive Disclosure (Tiết lộ thông tin dần dần)

Skill quản lý context bằng hệ thống load 3 tầng:

| Tầng | Thời điểm load | Mục tiêu kích thước |
|------|---------------|-------------------|
| **Metadata** (name + description) | Luôn có trong context | ~100 từ |
| **Nội dung SKILL.md** | Khi skill được trigger | <500 dòng |
| **references/** | Chỉ khi cần | Không giới hạn (script có thể chạy mà không cần load) |

**Quy tắc quản lý kích thước:**
- Khi SKILL.md tiếp cận 500 dòng, tách chi tiết sang references/ và để pointer "khi nào đọc file này" trong nội dung
- File reference trên 300 dòng phải bao gồm **mục lục (ToC)** ở đầu
- Nếu có biến thể theo domain/framework, tách theo domain trong references/ để chỉ load file liên quan

```
cloud-deploy/
├── SKILL.md (quy trình + hướng dẫn chọn)
└── references/
    ├── aws.md    ← Chỉ load khi chọn AWS
    ├── gcp.md
    └── azure.md
```

#### 4-5. Nguyên tắc kết nối Skill-Agent

- 1 agent ↔ 1~N skill (1:1 hoặc 1:nhiều)
- Skill dùng chung giữa nhiều agent cũng được
- Skill chứa "làm như thế nào", agent chứa "ai làm"

> Chi tiết mẫu viết, ví dụ và chuẩn data schema trong `references/skill-writing-guide.md`.

### Phase 5: Tích hợp & Điều phối

Orchestrator là dạng đặc biệt của skill, liên kết các agent và skill riêng lẻ thành một quy trình duy nhất để điều phối toàn bộ nhóm. Nếu skill riêng lẻ tạo trong Phase 4 định nghĩa "mỗi agent làm gì và làm thế nào", thì orchestrator định nghĩa "ai, khi nào, theo thứ tự nào để cộng tác". Template cụ thể trong `references/orchestrator-template.md`.

**Sửa đổi orchestrator khi mở rộng hiện có:** Khi mở rộng hiện có (không phải xây mới), không tạo orchestrator mới mà sửa đổi orchestrator hiện có. Khi thêm agent, phản ánh agent mới vào cấu thành nhóm, phân bổ tác vụ, luồng dữ liệu, và thêm từ khóa trigger liên quan đến agent mới vào description.

Mẫu orchestrator thay đổi theo chế độ thực thi được chọn trong Phase 2-1:

#### 5-0. Mẫu Orchestrator (theo chế độ)

**Mẫu nhóm agent (mặc định):**
Orchestrator cấu thành nhóm bằng `TeamCreate` và phân bổ tác vụ bằng `TaskCreate`. Các thành viên tự điều phối trực tiếp bằng `SendMessage`. Leader (orchestrator) giám sát tiến độ và tổng hợp kết quả.

```
[Orchestrator/Leader]
    ├── TeamCreate(team_name, members)
    ├── TaskCreate(tasks with dependencies)
    ├── Thành viên tự điều phối (SendMessage)
    ├── Thu thập & tổng hợp kết quả
    └── Giải thể nhóm
```

**Mẫu subagent (phương án thay thế):**
Orchestrator gọi trực tiếp subagent bằng `Agent` tool. Thực thi song song với `run_in_background: true`, kết quả chỉ trả về cho main. Dùng khi không cần giao tiếp nhóm và muốn giảm overhead.

```
[Orchestrator]
    ├── Agent(agent-1, run_in_background=true)
    ├── Agent(agent-2, run_in_background=true)
    ├── Chờ & thu thập kết quả
    └── Tạo đầu ra tích hợp
```

**Mẫu hybrid:**
Kết hợp các chế độ khác nhau theo Phase. Tổ hợp thường dùng:
- **Thu thập song song (subagent) → Tích hợp đồng thuận (nhóm)**: Phase 2 thu thập tài liệu độc lập song song bằng subagent → Phase 3 tạo nhóm để thảo luận và tích hợp dựa trên đồng thuận
- **Tạo nhóm (nhóm) → Xác thực (subagent)**: Phase 2 nhóm tạo bản nháp → Phase 3 subagent đơn lẻ xác thực độc lập
- **Tái cấu trúc nhóm giữa Phase**: `TeamDelete` rồi `TeamCreate` mới mỗi Phase, thêm subagent call ở giữa

Khi chọn hybrid, ghi rõ chế độ thực thi của Phase đó ở đầu mỗi mục Phase trong orchestrator (v.d.: `**Chế độ thực thi:** Nhóm agent`).

#### 5-1. Giao thức truyền dữ liệu

Ghi rõ trong orchestrator cách truyền dữ liệu giữa các agent:

| Chiến lược | Cách thực hiện | Chế độ áp dụng | Khi nào phù hợp |
|-----------|---------------|----------------|----------------|
| **Dựa trên message** | Giao tiếp trực tiếp giữa thành viên qua `SendMessage` | Nhóm | Điều phối thời gian thực, trao đổi phản hồi, truyền trạng thái nhẹ |
| **Dựa trên task** | Chia sẻ trạng thái tác vụ qua `TaskCreate`/`TaskUpdate` | Nhóm | Theo dõi tiến độ, quản lý phụ thuộc, yêu cầu tác vụ |
| **Dựa trên file** | Ghi và đọc file tại đường dẫn đã thỏa thuận | Nhóm + Subagent | Dữ liệu lớn, đầu ra có cấu trúc, cần audit trail |
| **Dựa trên return value** | Giá trị trả về của `Agent` tool | Subagent | Main thu thập trực tiếp kết quả subagent |

**Tổ hợp khuyến nghị (chế độ nhóm):** Dựa trên task (điều phối) + dựa trên file (đầu ra) + dựa trên message (giao tiếp thời gian thực)
**Tổ hợp khuyến nghị (chế độ subagent):** Dựa trên return value (thu thập kết quả) + dựa trên file (đầu ra lớn)
**Hybrid:** Áp dụng tổ hợp tương ứng cho chế độ thực thi của từng Phase

Quy tắc khi truyền qua file:
- Tạo thư mục `_workspace/` dưới thư mục làm việc để lưu đầu ra trung gian
- Quy ước đặt tên file: `{phase}_{agent}_{artifact}.{ext}` (v.d.: `01_analyst_requirements.md`)
- Chỉ xuất đầu ra cuối cùng đến đường dẫn người dùng chỉ định, giữ lại file trung gian (`_workspace/`) (để xác thực hậu kỳ và audit)

#### 5-2. Xử lý lỗi

Ghi chính sách xử lý lỗi trong orchestrator. Nguyên tắc cốt lõi: Thử lại 1 lần, nếu thất bại tiếp thì tiến hành không có kết quả đó (ghi rõ thiếu sót trong báo cáo), không xóa dữ liệu mâu thuẫn mà ghi rõ nguồn gốc.

> Bảng chiến lược theo loại lỗi và chi tiết triển khai trong `references/orchestrator-template.md` phần "Xử lý lỗi".

#### 5-3. Hướng dẫn quy mô nhóm

| Quy mô tác vụ | Số thành viên khuyến nghị | Số tác vụ/thành viên |
|--------------|--------------------------|---------------------|
| Nhỏ (5~10 tác vụ) | 2~3 người | 3~5 cái |
| Vừa (10~20 tác vụ) | 3~5 người | 4~6 cái |
| Lớn (20+ tác vụ) | 5~7 người | 4~5 cái |

> Nhóm càng nhiều thành viên, overhead điều phối càng lớn. 3 thành viên tập trung tốt hơn 5 thành viên phân tán.

#### 5-4. Đăng ký pointer harness vào CLAUDE.md

Sau khi hoàn tất cấu hình harness, đăng ký pointer tối thiểu vào `CLAUDE.md` của dự án. CLAUDE.md được load mỗi phiên mới, vì vậy chỉ cần ghi sự tồn tại của harness và quy tắc trigger, orchestrator skill sẽ xử lý phần còn lại.

**Template CLAUDE.md:**

````markdown
## Harness: {Tên domain}

**Mục tiêu:** {Mục tiêu cốt lõi của harness trong một dòng}

**Trigger:** Khi có yêu cầu liên quan đến {domain}, dùng skill `{orchestrator-skill-name}`. Câu hỏi đơn giản có thể trả lời trực tiếp.

**Lịch sử thay đổi:**
| Ngày | Nội dung thay đổi | Mục tiêu | Lý do |
|------|-----------------|----------|-------|
| {YYYY-MM-DD} | Cấu hình ban đầu | Toàn bộ | - |
````

**Không đưa vào CLAUDE.md:** Danh sách agent, danh sách skill, cấu trúc thư mục, chi tiết quy tắc thực thi. Lý do: Danh sách agent/skill được quản lý trong orchestrator skill và `.claude/agents/`, `.claude/skills/` nên là trùng lặp. Cấu trúc thư mục có thể kiểm tra trực tiếp từ file system. CLAUDE.md chỉ chứa **pointer (quy tắc trigger) + lịch sử thay đổi**.

#### 5-5. Hỗ trợ tác vụ tiếp theo

Orchestrator phải xử lý không chỉ lần chạy đầu mà cả các tác vụ tiếp theo. Đảm bảo ba điều sau:

**1. Bao gồm từ khóa tác vụ tiếp theo trong description của orchestrator:**
Chỉ có từ khóa tạo lần đầu không đủ để trigger yêu cầu tiếp theo. Bắt buộc bao gồm trong description:
- "chạy lại", "thực thi lại", "cập nhật", "sửa đổi", "bổ sung"
- "{domain} của {phần tác vụ} làm lại"
- "dựa trên kết quả trước", "cải thiện kết quả"

**2. Thêm bước kiểm tra context vào Phase 1 của orchestrator:**
Khi bắt đầu quy trình, kiểm tra sự tồn tại của đầu ra cũ để quyết định chế độ thực thi:
- `_workspace/` tồn tại + người dùng yêu cầu sửa một phần → **Chạy lại một phần** (chỉ gọi lại agent đó)
- `_workspace/` tồn tại + người dùng cung cấp đầu vào mới → **Chạy mới** (di chuyển `_workspace/` hiện có sang `_workspace_prev/`)
- `_workspace/` không tồn tại → **Chạy lần đầu**

**3. Bao gồm hướng dẫn gọi lại trong định nghĩa agent:**
Ghi rõ "hành động khi có đầu ra cũ" trong mỗi file `.md` của agent:
- Nếu file kết quả cũ tồn tại, đọc và phản ánh điểm cần cải thiện
- Nếu có phản hồi từ người dùng, chỉ sửa phần đó

> Tham khảo mục "Phase 0: Kiểm tra context" trong template orchestrator: `references/orchestrator-template.md`

### Phase 6: Xác thực & Kiểm thử

Xác thực harness đã tạo. Phương pháp kiểm thử chi tiết trong `references/skill-testing-guide.md`.

#### 6-1. Xác thực cấu trúc

- Kiểm tra tất cả file agent ở đúng vị trí
- Xác thực frontmatter của skill (name, description)
- Kiểm tra tính nhất quán của tham chiếu giữa các agent
- Xác nhận không có command nào được tạo

#### 6-2. Xác thực theo chế độ thực thi

- **Nhóm agent**: Kiểm tra đường dẫn giao tiếp giữa thành viên, phụ thuộc tác vụ, tính phù hợp của quy mô nhóm
- **Subagent**: Kiểm tra kết nối đầu vào/đầu ra của từng agent, cài đặt `run_in_background`, logic thu thập return value
- **Hybrid**: Kiểm tra chế độ thực thi của từng Phase có được ghi rõ trong orchestrator không, dữ liệu có liên tục qua ranh giới Phase không (khi chuyển nhóm → subagent, đầu ra của nhóm có được kết nối làm đầu vào của subagent không)

#### 6-3. Kiểm thử thực thi Skill

Thực hiện kiểm thử thực thi thực sự cho mỗi skill đã tạo:

1. **Viết prompt kiểm thử** — Viết 2~3 prompt kiểm thử thực tế cho mỗi skill. Viết bằng câu cụ thể, tự nhiên giống như người dùng thực sự sẽ nhập.

2. **Chạy so sánh With-skill vs Without-skill** — Nếu có thể, chạy song song với skill và không có skill để xác nhận giá trị gia tăng của skill. Spawn hai agent:
   - **With-skill**: Đọc skill rồi thực hiện tác vụ
   - **Without-skill (baseline)**: Thực hiện cùng prompt mà không có skill

3. **Đánh giá kết quả** — Đánh giá chất lượng đầu ra bằng cả định tính (người dùng review) và định lượng (dựa trên assertion). Nếu đầu ra có thể xác thực khách quan (tạo file, trích xuất dữ liệu, v.v.) hãy định nghĩa assertion; nếu chủ quan (phong cách, thiết kế) thì dựa vào phản hồi người dùng.

4. **Vòng cải thiện lặp lại** — Khi phát hiện vấn đề từ kết quả kiểm thử:
   - **Tổng quát hóa** phản hồi để sửa đổi skill (cấm sửa hẹp chỉ khớp với ví dụ cụ thể)
   - Kiểm thử lại sau khi sửa đổi
   - Lặp lại cho đến khi người dùng hài lòng hoặc không còn cải thiện đáng kể

5. **Đóng gói mẫu lặp lại** — Nếu phát hiện code mà các agent thường xuyên viết chung (v.d.: mọi kiểm thử đều tạo cùng helper script), đóng gói code đó vào `scripts/` trước.

#### 6-4. Xác thực trigger

Xác thực description của mỗi skill được trigger đúng cách:

1. **Should-trigger queries** (8~10 cái) — Các cách diễn đạt khác nhau với cùng ý định (trang trọng/thông thường, rõ ràng/ngụ ý)
2. **Should-NOT-trigger queries** (8~10 cái) — Query "near-miss" có từ khóa tương tự nhưng phù hợp với công cụ/skill khác hơn

**Cốt lõi viết near-miss:** Query hoàn toàn không liên quan như "viết hàm Fibonacci" không có giá trị kiểm thử. Query **ranh giới mơ hồ** như "trích xuất biểu đồ trong file Excel này thành PNG" (skill xlsx vs chuyển đổi ảnh) mới là test case tốt.

Cũng kiểm tra xung đột trigger với skill hiện có trong bước này.

#### 6-5. Dry-run test

- Kiểm tra thứ tự Phase của orchestrator skill có logic không
- Xác nhận không có liên kết chết (dead link) trong đường dẫn truyền dữ liệu
- Kiểm tra đầu vào của tất cả agent có khớp với đầu ra của Phase trước không
- Xác nhận đường dẫn fallback cho mỗi kịch bản lỗi có thể thực thi được không

#### 6-6. Viết kịch bản kiểm thử

- Thêm mục `## Kịch bản kiểm thử` vào orchestrator skill
- Mô tả ít nhất 1 luồng bình thường + 1 luồng lỗi

### Phase 7: Tiến hóa Harness

Harness không phải đầu ra tĩnh làm một lần rồi xong. Đây là hệ thống tiến hóa liên tục theo phản hồi người dùng.

#### 7-1. Thu thập phản hồi sau mỗi lần chạy

Sau mỗi lần thực thi harness hoàn thành, yêu cầu phản hồi từ người dùng:
- "Kết quả có phần nào cần cải thiện không?"
- "Bạn có muốn thay đổi gì về cấu thành nhóm agent hay quy trình không?"

Nếu không có phản hồi, bỏ qua. Không ép buộc, nhưng phải cung cấp cơ hội.

#### 7-2. Đường dẫn phản ánh phản hồi

Mục tiêu sửa đổi khác nhau tùy loại phản hồi:

| Loại phản hồi | Mục tiêu sửa đổi | Ví dụ |
|--------------|-----------------|-------|
| Chất lượng đầu ra | Skill của agent đó | "Phân tích quá nông" → Thêm tiêu chí độ sâu vào skill |
| Vai trò agent | File định nghĩa agent `.md` | "Cần review bảo mật nữa" → Thêm agent mới |
| Thứ tự quy trình | Orchestrator skill | "Nên xác thực trước" → Đổi thứ tự Phase |
| Cấu thành nhóm | Orchestrator + agent | "Hai cái này có thể gộp được" → Gộp agent |
| Thiếu trigger | Skill description | "Dùng cách diễn đạt này không chạy" → Mở rộng description |

#### 7-3. Lịch sử thay đổi

Mọi thay đổi đều được ghi vào bảng **Lịch sử thay đổi** trong CLAUDE.md (cùng bảng với mục "Lịch sử thay đổi" trong template Phase 5-4):

```markdown
**Lịch sử thay đổi:**
| Ngày | Nội dung thay đổi | Mục tiêu | Lý do |
|------|-----------------|----------|-------|
| 2026-04-05 | Cấu hình ban đầu | Toàn bộ | - |
| 2026-04-07 | Thêm QA agent | agents/qa.md | Phản hồi thiếu xác thực chất lượng đầu ra |
| 2026-04-10 | Thêm hướng dẫn tone | skills/content-creator | Phản hồi "quá khô cứng" |
```

Lịch sử này giúp theo dõi harness đã tiến hóa theo hướng nào và ngăn ngừa hồi quy.

#### 7-4. Trigger tiến hóa

Không chỉ khi người dùng nói rõ "sửa harness", mà còn trong các tình huống sau cũng đề xuất tiến hóa:
- Khi cùng loại phản hồi lặp lại 2 lần trở lên
- Khi phát hiện mẫu agent thất bại lặp lại
- Khi quan sát thấy người dùng làm thủ công thay vì dùng orchestrator

#### 7-5. Quy trình vận hành/bảo trì

Thực hiện kiểm tra/sửa đổi/đồng bộ harness hiện có một cách có hệ thống. Tuân theo quy trình này khi vào nhánh "vận hành/bảo trì" từ Phase 0.

**Bước 1: Kiểm tra hiện trạng**
- So sánh danh sách file trong `.claude/agents/` với cấu thành agent của orchestrator skill → Tạo danh sách không nhất quán
- So sánh danh sách thư mục `.claude/skills/` với cấu thành skill của orchestrator skill → Tạo danh sách không nhất quán
- Báo cáo kết quả kiểm toán cho người dùng

**Bước 2: Thêm/sửa đổi dần dần**
- Thêm/sửa/xóa agent, thêm/sửa/xóa skill theo yêu cầu người dùng
- Thay đổi từng cái một, thực thi Bước 3 (đồng bộ) ngay sau mỗi thay đổi

**Bước 3: Cập nhật lịch sử thay đổi CLAUDE.md**
- Ghi ngày, nội dung thay đổi, mục tiêu, lý do vào bảng lịch sử thay đổi

**Bước 4: Xác thực thay đổi**
- Xác thực cấu trúc agent/skill đã sửa (theo tiêu chí Phase 6-1)
- Nếu phạm vi sửa đổi ảnh hưởng đến trigger, xác thực trigger (theo tiêu chí Phase 6-4)
- Khi thay đổi lớn (thay đổi kiến trúc, thêm/xóa 3+ agent) thực hiện cả Phase 6-3 (kiểm thử thực thi), 6-5 (dry-run)
- Xác nhận cuối cùng CLAUDE.md và file thực tế nhất quán

## Checklist đầu ra

Kiểm tra sau khi hoàn tất tạo:

- [ ] `dự-án/.claude/agents/` — **Bắt buộc tạo file định nghĩa agent** (ngay cả loại tích hợp sẵn cũng phải tạo file)
- [ ] `dự-án/.claude/skills/` — Các file skill (SKILL.md + references/)
- [ ] 1 orchestrator skill (bao gồm luồng dữ liệu + xử lý lỗi + kịch bản kiểm thử)
- [ ] Ghi rõ chế độ thực thi (nhóm agent / subagent / hybrid, nếu hybrid ghi chế độ theo Phase)
- [ ] Tất cả lời gọi Agent đều ghi rõ tham số `model: "opus"`
- [ ] Hoàn tất kiểm tra trùng lặp agent hiện có trước khi tạo agent mới (Phase 3-0)
- [ ] Hoàn tất kiểm tra trùng lặp skill hiện có trước khi tạo skill mới (Phase 4-0)
- [ ] `.claude/commands/` — Không tạo bất kỳ thứ gì
- [ ] Không xung đột với agent/skill hiện có
- [ ] Skill description được viết tích cực ("pushy") — **Bao gồm từ khóa tác vụ tiếp theo**
- [ ] Nội dung SKILL.md dưới 500 dòng, nếu vượt thì tách sang references/
- [ ] Hoàn tất xác thực thực thi với 2~3 prompt kiểm thử
- [ ] Hoàn tất xác thực trigger (should-trigger + should-NOT-trigger)
- [ ] **Đăng ký pointer harness vào CLAUDE.md** (quy tắc trigger + lịch sử thay đổi)
- [ ] **Ghi thêm/xóa/sửa agent/skill vào lịch sử thay đổi CLAUDE.md**
- [ ] **Bước kiểm tra context trong Phase 1 của orchestrator** (phân biệt lần chạy đầu/tiếp theo/một phần)

## Tham khảo

- Mẫu harness: `references/agent-design-patterns.md`
- Ví dụ harness hiện có (bao gồm toàn bộ file thực tế): `references/team-examples.md`
- Template orchestrator: `references/orchestrator-template.md`
- **Hướng dẫn viết skill**: `references/skill-writing-guide.md` — Mẫu viết, ví dụ và chuẩn data schema
- **Hướng dẫn kiểm thử skill**: `references/skill-testing-guide.md` — Phương pháp kiểm thử/đánh giá/cải thiện lặp lại
- **Hướng dẫn QA agent**: `references/qa-agent-guide.md` — Tham khảo khi bao gồm QA agent trong build harness. Phương pháp xác thực tích hợp, mẫu bug ranh giới, template định nghĩa QA agent. Dựa trên 7 trường hợp bug thực tế từ dự án thực.
