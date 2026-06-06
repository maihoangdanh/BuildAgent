<p align="center">
  <img src="harness_banner.png" alt="Harness Banner" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Phiên_bản-1.2.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/Giấy_phép-Apache_2.0-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/Claude_Code-Plugin-purple.svg" alt="Claude Code Plugin">
  <img src="https://img.shields.io/badge/Kiến_trúc-6_Mẫu-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Chế_độ-Nhóm_Agent-green.svg" alt="Agent Teams">
  <a href="https://github.com/revfactory/harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#danh-mục--vị-trí-của-harness"><img src="https://img.shields.io/badge/Tầng-L3%20Meta--Factory-orange" alt="Layer"></a>
  <a href="#danh-mục--vị-trí-của-harness"><img src="https://img.shields.io/badge/Phụ--tầng-Team--Architecture%20Factory-teal" alt="Sub-layer"></a>
  <a href="#"><img src="https://img.shields.io/badge/README-Tiếng_Việt-lightgrey" alt="i18n"></a>
</p>

# Harness — Nhà Máy Kiến Trúc Nhóm Agent cho Claude Code

**Tiếng Việt**

> **Harness là nhà máy kiến trúc nhóm agent cho Claude Code.** Nói **"xây harness cho dự án này"** và plugin sẽ biến mô tả domain của bạn thành một nhóm agent cùng các skill tương ứng — được chọn từ sáu mẫu kiến trúc nhóm định sẵn.

## Tổng quan

Harness tận dụng hệ thống nhóm agent của Claude Code để phân rã các tác vụ phức tạp thành các nhóm agent chuyên biệt phối hợp với nhau. Nói "xây harness cho dự án này" và nó tự động tạo ra định nghĩa agent (`.claude/agents/`) và skill (`.claude/skills/`) phù hợp với domain của bạn.

## Danh mục — Vị trí của Harness

Harness nằm ở tầng **L3 Meta-Factory** trong hệ sinh thái Claude Code — tầng tạo ra các harness khác thay vì là một harness thông thường. Trong L3, chúng ta chọn phụ-tầng cụ thể: **Nhà máy Kiến trúc Nhóm (Team-Architecture Factory)**.

| Tầng | Chức năng | Dự án lân cận |
|------|-----------|---------------|
| **L3 — Meta-Factory / Team-Architecture Factory** (chúng ta) | Câu mô tả domain → nhóm agent + skill, qua 6 mẫu nhóm định sẵn | — |
| L3 — Meta-Factory / Runtime-Configuration Factory | Cấu hình runtime xác định và lặp lại được | [coleam00/Archon](https://github.com/coleam00/Archon) |
| L3 — Meta-Factory / Codex Runtime Port | Cùng khái niệm, runtime Codex | [SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness) |
| L2 — Cross-Harness Workflow | Chuẩn hóa skill/rule/hook trên nhiều harness | [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) |

> Archon tạo cấu hình runtime xác định. Harness tạo kiến trúc nhóm agent (pipeline, fan-out/fan-in, expert pool, producer-reviewer, supervisor, hierarchical delegation) cùng các skill mà agent sử dụng. Hai phụ-tầng khác nhau trong cùng L3. Chọn Archon cho tính xác định runtime, Harness cho kiến trúc nhóm, hoặc kết hợp cả hai.

## Lịch sử Star

<a href="https://www.star-history.com/?repos=revfactory%2Fharness&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=revfactory/harness&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=revfactory/harness&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=revfactory/harness&type=date&legend=top-left" />
 </picture>
</a>


## Tính năng chính

- **Thiết kế Nhóm Agent** — 6 mẫu kiến trúc: Pipeline, Fan-out/Fan-in, Expert Pool, Producer-Reviewer, Supervisor và Hierarchical Delegation
- **Tạo Skill** — Tự động tạo skill với Progressive Disclosure để quản lý context hiệu quả
- **Điều phối** — Truyền dữ liệu liên agent, xử lý lỗi và giao thức phối hợp nhóm
- **Xác thực** — Kiểm tra trigger, dry-run testing và so sánh with-skill vs without-skill


## Quy trình

```
Phase 1: Phân tích Domain
    ↓
Phase 2: Thiết kế Kiến trúc Nhóm (Agent Teams vs Subagents)
    ↓
Phase 3: Tạo Định nghĩa Agent (.claude/agents/)
    ↓
Phase 4: Tạo Skill (.claude/skills/)
    ↓
Phase 5: Tích hợp & Điều phối
    ↓
Phase 6: Xác thực & Kiểm thử
```

## Cài đặt & Sử dụng

> **Chỉ cài một lần.** Sau đó dùng được trong mọi project khác — không cần quay lại thư mục này.

---

### 🪟 Windows (PowerShell)

**Bước 1 — Clone repo về máy**
```powershell
git clone https://github.com/maihoangdanh/BuildAgent.git
cd BuildAgent
```

**Bước 2 — Cài skill vào Claude Code**
```powershell
Copy-Item -Recurse skills\harness "$env:USERPROFILE\.claude\skills\harness"
```

**Bước 3 — Bật Agent Teams (vĩnh viễn)**
```powershell
[System.Environment]::SetEnvironmentVariable("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS", "1", "User")
```
> Sau bước này cần **mở lại PowerShell** để biến môi trường có hiệu lực.

**Bước 4 — Dùng ở bất kỳ project nào**
```powershell
# Ví dụ: mở Claude Code trong project của bạn
cd D:\Projects\TenProjectCuaBan
claude
```
Rồi gõ trong Claude Code:
```
Xây harness cho dự án này
```

---

### 🍎 macOS / 🐧 Linux

**Bước 1 — Clone repo về máy**
```bash
git clone https://github.com/maihoangdanh/BuildAgent.git
cd BuildAgent
```

**Bước 2 — Cài skill vào Claude Code**
```bash
cp -r skills/harness ~/.claude/skills/harness
```

**Bước 3 — Bật Agent Teams (vĩnh viễn)**
```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc
```
> Nếu dùng bash thay zsh: thay `~/.zshrc` bằng `~/.bashrc`

**Bước 4 — Dùng ở bất kỳ project nào**
```bash
# Ví dụ: mở Claude Code trong project của bạn
cd ~/Projects/TenProjectCuaBan
claude
```
Rồi gõ trong Claude Code:
```
Xây harness cho dự án này
```

---

### Kết quả sau khi chạy

Harness phân tích codebase của project và tự sinh ra:
```
project-của-bạn/
└── .claude/
    ├── agents/      ← Các agent chuyên biệt cho domain của bạn
    └── skills/      ← Skill hướng dẫn từng agent làm việc
```

Từ lần sau trong project đó, Claude Code sẽ tự dùng đội agent này mỗi khi bạn làm việc.

## Cấu trúc Plugin

```
harness/
├── .claude-plugin/
│   └── plugin.json                 # Plugin manifest
├── skills/
│   └── harness/
│       ├── SKILL.md                # Định nghĩa skill chính (quy trình 6 phase)
│       └── references/
│           ├── agent-design-patterns.md   # 6 mẫu kiến trúc
│           ├── orchestrator-template.md   # Template orchestrator nhóm/subagent
│           ├── team-examples.md           # 5 cấu hình nhóm thực tế
│           ├── skill-writing-guide.md     # Hướng dẫn viết skill
│           ├── skill-testing-guide.md     # Phương pháp kiểm thử & đánh giá
│           └── qa-agent-guide.md          # Hướng dẫn tích hợp QA agent
└── README.md
```

## Cách dùng

Kích hoạt trong Claude Code bằng các prompt như:

```
Xây harness cho dự án này
Thiết kế nhóm agent cho domain này
Cài đặt harness
```

### Chế độ thực thi

| Chế độ | Mô tả | Phù hợp khi |
|--------|-------|-------------|
| **Nhóm Agent** (mặc định) | TeamCreate + SendMessage + TaskCreate | 2+ agent cần cộng tác |
| **Subagent** | Gọi trực tiếp Agent tool | Tác vụ đơn lẻ, không cần giao tiếp liên agent |

<p align="center">
  <img src="harness_team.png" alt="Harness Agent Team" width="500">
</p>

### Mẫu kiến trúc

| Mẫu | Mô tả |
|-----|-------|
| Pipeline | Tác vụ tuần tự phụ thuộc nhau |
| Fan-out/Fan-in | Tác vụ song song độc lập |
| Expert Pool | Gọi có chọn lọc theo ngữ cảnh |
| Producer-Reviewer | Tạo nội dung rồi kiểm tra chất lượng |
| Supervisor | Agent trung tâm phân phối tác vụ động |
| Hierarchical Delegation | Ủy quyền đệ quy từ trên xuống |

## Kết quả đầu ra

Các file được Harness tạo ra:

```
dự-án-của-bạn/
├── .claude/
│   ├── agents/          # File định nghĩa agent
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa.md
│   └── skills/          # File skill
│       ├── analyze/
│       │   └── SKILL.md
│       └── build/
│           ├── SKILL.md
│           └── references/
```

## Trường hợp sử dụng — Thử các prompt này

Sao chép bất kỳ prompt nào dưới đây vào Claude Code sau khi cài Harness:

**Nghiên cứu chuyên sâu**
```
Xây harness cho nghiên cứu chuyên sâu. Tôi cần nhóm agent có thể điều tra
bất kỳ chủ đề nào từ nhiều góc độ — tìm kiếm web, nguồn học thuật,
phản ứng cộng đồng — sau đó xác nhận chéo kết quả và tạo báo cáo toàn diện.
```

**Phát triển website**
```
Xây harness cho phát triển website full-stack. Nhóm cần xử lý
thiết kế, frontend (React/Next.js), backend (API) và kiểm thử QA trong
một pipeline phối hợp từ wireframe đến triển khai.
```

**Sản xuất Webtoon / Truyện tranh**
```
Xây harness cho sản xuất tập webtoon. Tôi cần agent cho viết truyện,
prompt thiết kế nhân vật, lập kế hoạch bố cục bảng và chỉnh sửa hội thoại.
Chúng nên kiểm tra lẫn nhau để đảm bảo nhất quán về phong cách.
```

**Lập kế hoạch nội dung YouTube**
```
Xây harness cho tạo nội dung YouTube. Nhóm cần nghiên cứu
chủ đề trending, viết kịch bản, tối ưu tiêu đề/tag cho SEO và lên kế hoạch
ý tưởng thumbnail — tất cả được điều phối bởi supervisor agent.
```

**Đánh giá & Tái cấu trúc code**
```
Xây harness cho đánh giá code toàn diện. Tôi muốn các agent song song
kiểm tra kiến trúc, lỗ hổng bảo mật, điểm nghẽn hiệu suất
và style code — sau đó gộp tất cả phát hiện vào một báo cáo duy nhất.
```

**Tài liệu kỹ thuật**
```
Xây harness tạo tài liệu API từ codebase này.
Agent cần phân tích endpoint, viết mô tả, tạo ví dụ sử dụng
và đánh giá tính đầy đủ.
```

**Thiết kế Data Pipeline**
```
Xây harness cho thiết kế data pipeline. Tôi cần agent cho thiết kế schema,
logic ETL, quy tắc xác thực dữ liệu và thiết lập giám sát theo
phân cấp ủy quyền.
```

**Chiến dịch Marketing**
```
Xây harness cho tạo chiến dịch marketing. Nhóm cần nghiên cứu
thị trường mục tiêu, viết nội dung quảng cáo, thiết kế concept hình ảnh và thiết lập
kế hoạch A/B test với vòng đánh giá chất lượng lặp lại.
```

## Cùng tồn tại — Harness và các dự án lân cận

Harness không đơn độc trong hệ sinh thái Claude Code / agent-framework. Các repo sau tồn tại ở các tầng liền kề; mỗi repo được mô tả theo cặp "X là..., Harness là..." để bạn có thể chọn cái phù hợp hoặc kết hợp nhiều cái.

| Repo | Vị trí của họ | Mối quan hệ với Harness |
|------|---------------|-------------------------|
| [coleam00/Archon](https://github.com/coleam00/Archon) | "harness builder" — cấu hình runtime xác định, lặp lại được | **Cùng L3, phụ-tầng lân cận.** Archon là Runtime-Configuration Factory, Harness là Team-Architecture Factory. Chọn Archon cho tính xác định runtime, Harness cho kiến trúc nhóm, hoặc kết hợp cả hai. |
| [SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness) | Port Codex của cùng khái niệm | **Cùng L3, runtime khác.** Dùng Harness trên Claude Code, meta-harness trên Codex. |
| [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) | "Tầng hiệu suất & workflow agent harness" (nằm trên các harness hiện có) | **Tầng khác.** ECC là tầng chuẩn hóa trên các harness; Harness là nhà máy tạo ra harness. Có thể kết hợp nối tiếp. |
| [wshobson/agents](https://github.com/wshobson/agents) | Danh mục subagent/skill (182 agent, 149 skill) | **Nhà máy ↔ nguồn cung cấp linh kiện.** wshobson là danh mục để tham khảo; Harness thiết kế nhóm. Hấp thụ các entry của wshobson làm linh kiện trong nhóm do Harness tạo ra. |
| [LangGraph](https://langchain-ai.github.io/langgraph/) | Điều phối state-graph, không phụ thuộc LLM | **Hướng khác.** LangGraph dành cho điều phối dài hạn, phục hồi được trạng thái; Harness dành cho thiết kế nhóm nhanh, native Claude Code. |

## Được xây bằng Harness

### Harness 100

**[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 100 harness nhóm agent sẵn sàng cho production trên 10 domain, có cả tiếng Anh lẫn tiếng Hàn (tổng cộng 200 gói). Mỗi harness đi kèm 4-5 agent chuyên biệt, một orchestrator skill và các skill domain — tất cả được tạo bởi plugin này. 1.808 file markdown bao gồm tạo nội dung, phát triển phần mềm, data/AI, chiến lược kinh doanh, giáo dục, pháp lý, sức khỏe và nhiều hơn nữa.

### Nghiên cứu: A/B Testing hiệu quả của Harness

**[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)** — Thí nghiệm có kiểm soát trên 15 tác vụ kỹ thuật phần mềm đo lường tác động của cấu hình trước có cấu trúc đến chất lượng đầu ra của LLM code agent.

| Chỉ số | Không có Harness | Có Harness | Cải thiện |
|--------|:----------------:|:----------:|:---------:|
| Điểm chất lượng trung bình | 49,5 | 79,3 | **+60%** |
| Tỷ lệ thắng | — | — | **100%** (15/15) |
| Phương sai đầu ra | — | — | **-32%** |

Phát hiện chính: hiệu quả tăng theo độ phức tạp tác vụ — tác vụ càng khó, cải thiện càng lớn (+23,8 cơ bản, +29,6 nâng cao, +36,2 chuyên gia).

**Số liệu chính xác để trích dẫn:** +60% chất lượng trung bình (49,5 → 79,3), tỷ lệ thắng 15/15, phương sai giảm −32% (n=15, A/B do tác giả đo, chờ xác nhận từ bên thứ ba).

> Bài báo đầy đủ: *Hwang, M. (2026). Harness: Cấu hình trước có cấu trúc để nâng cao chất lượng đầu ra của LLM Code Agent.*

## Yêu cầu

- [Agent Teams được bật](https://code.claude.com/docs/en/agent-teams): `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## Câu hỏi thường gặp

<details>
<summary><b>Q1. "+60%" có phải là quảng cáo thổi phồng không?</b></summary>

**A.** Con số +60% đến từ **A/B do tác giả đo (n=15, 15 tác vụ, đo trên repo chị em `claude-code-harness`)**. Mọi trích dẫn trong repo này đều đi kèm thông tin tiết lộ "n=15, do tác giả đo, chờ xác nhận từ bên thứ ba" trong cùng câu. Để quyết định áp dụng, chúng tôi khuyến nghị chạy thử nghiệm nội bộ 2–4 tuần và đo số liệu của riêng bạn.

**Bằng chứng:**
- A/B của tác giả: [revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)
- Bài báo: *Hwang, M. (2026). Harness: Cấu hình trước có cấu trúc để nâng cao chất lượng đầu ra của LLM Code Agent*
</details>

<details>
<summary><b>Q2. Tại sao dùng "harness factory" thay vì "harness builder"? Điều này có cạnh tranh với Archon không?</b></summary>

**A.** Archon tạo cấu hình runtime xác định — đó là **Runtime-Configuration Factory**. Harness tạo kiến trúc nhóm agent (cấu trúc nhóm, giao thức tin nhắn, cổng review) — đó là **Team-Architecture Factory**. Chúng là **phụ-tầng lân cận của cùng L3 Meta-Factory** và phục vụ nhu cầu khác nhau. Chọn Archon cho tính xác định runtime, Harness cho mẫu kiến trúc nhóm, hoặc kết hợp cả hai (thiết kế kiến trúc với Harness → triển khai runtime với Archon).

**Bằng chứng:**
- Tự định nghĩa của Archon: [clawfit docs/reference-levels.md](https://github.com/hongsw/clawfit/blob/main/docs/reference-levels.md)
- Khai báo phụ-tầng: xem phần **Danh mục — Vị trí của Harness** ở trên
- Repo Archon: [github.com/coleam00/Archon](https://github.com/coleam00/Archon)
</details>

<details>
<summary><b>Q3. "Chỉ Claude Code" có quá hạn hẹp không? Còn Gemini/Codex thì sao?</b></summary>

**A.** Hiện tại runtime chính thức chỉ là Claude Code. Port Codex của cùng khái niệm — [SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness) — đã có công khai, nên các nhóm dùng Codex có thể bắt đầu từ đó. Harness chọn "native Claude Code, sâu" thay vì "đa runtime, nông"; cộng tác đa runtime với các repo anh chị em (meta-harness, harness-init, OpenRig) đang trong lộ trình.

**Bằng chứng:**
- Port Codex: [github.com/SaehwanPark/meta-harness](https://github.com/SaehwanPark/meta-harness)
- Scaffolder đa runtime: [github.com/Gizele1/harness-init](https://github.com/Gizele1/harness-init)
</details>

## Giấy phép

Apache 2.0
