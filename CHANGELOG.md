# Nhật ký thay đổi

Dự án này tuân theo [Semantic Versioning](https://semver.org/).

## [Chưa phát hành]

### Thêm mới
- Bước kiểm tra trùng lặp trước khi tạo agent/skill mới (Phase 3-0, Phase 4-0)
- Mục "Thiết kế tái sử dụng agent" trong `references/agent-design-patterns.md`
- §9 "Thiết kế tái sử dụng skill" trong `references/skill-writing-guide.md`

### Thay đổi
- Nêu rõ 3-0/4-0 trong ma trận chọn Phase
- Thêm pointer bước kiểm tra tái sử dụng vào Phase 2-3
- Thêm 2 mục kiểm tra tái sử dụng vào checklist đầu ra

---

## [1.2.1] - 2026-04-18

### Sửa lỗi

- **Đồng bộ phiên bản** — Badge trong README.md / README_KO.md / README_JA.md hiển thị `v1.0.1`, `.claude-plugin/marketplace.json` là `1.1.0`, `.claude-plugin/plugin.json` là `1.2.0` (3 nơi không khớp) → thống nhất tất cả thành **v1.2.0** (theo plugin.json)
- **Chuẩn bị giải quyết tình trạng 0 tagged release** — Lập kế hoạch gắn tag hồi tố cho v1.0.0 / v1.0.1 / v1.1.0 / v1.2.0 (xem `_workspace/release/audit-2026-04-18.md` §4)

### Thêm mới

- **Tuyên bố định vị: "harness factory"** — Đưa văn mô tả tự định nghĩa danh mục vào đầu README. Chiếm vị trí danh mục "nhà máy harness tạo agent + skill theo domain" (phân biệt với framework agent/prompt đơn lẻ)
- **CONTRIBUTING.md** — Hướng dẫn đóng góp và SLA rõ ràng (phản hồi PR lần đầu 72h, triage Issue 48h). Giảm rào cản onboarding cộng đồng
- **Thư mục docs/** — Tạo không gian mới cho tài liệu dài hạn (kiến trúc, migration, danh mục mẫu). Ngăn README phình to và cải thiện khả năng tìm kiếm
- **Chính sách phản hồi Issue #3** — Thêm template phản hồi chính thức và quy trình triage cho các issue cộng đồng

### Thay đổi

- `.claude-plugin/marketplace.json` version: `1.1.0` → `1.2.0`
- Badge README (3 bản EN/KO/JA): `Version-1.0.1` → `Version-1.2.0`
- **Viết lại description trong `.claude-plugin/plugin.json`** — `"Agent Team & Skill Architect — Meta-skill that designs..."` → `"Nhà máy kiến trúc nhóm cho Claude Code — meta-skill biến mô tả domain thành nhóm agent và bộ skill, với sáu mẫu kiến trúc nhóm định sẵn..."` (song ngữ Anh+Hàn, phản ánh định vị L3 Meta-Factory)
- **Mở rộng keywords trong `.claude-plugin/plugin.json`** — 5 → 17 từ khóa (thêm `harness-factory`, `team-architecture-factory`, `claude-code-plugin`, `agent-scaffolding`, `multi-agent`, 6 từ khóa mẫu)

## [1.2.0] - 2026-04-08

### Thay đổi

- **Đơn giản hóa chính sách đăng ký CLAUDE.md (xóa trùng lặp)** — Chuyển "đăng ký context" Phase 5-4 thành "đăng ký pointer". Xóa danh sách agent, danh sách skill, cấu trúc thư mục, chi tiết quy tắc thực thi khỏi CLAUDE.md, chỉ giữ **quy tắc trigger + lịch sử thay đổi**. Danh sách agent/skill được quản lý nguồn đơn trong `.claude/agents/`, `.claude/skills/` và orchestrator skill
- **Xóa bước đồng bộ tạm thời Phase 3/4** — Loại bỏ chỉ thị đồng bộ tạm thời trong Phase 3/4 để giảm gánh nặng đồng bộ CLAUDE.md. Đăng ký pointer cuối cùng chỉ thực hiện 1 lần trong Phase 5-4
- **Tái định nghĩa nguyên tắc cốt lõi số 3** — "Đăng ký context harness vào CLAUDE.md" → "Đăng ký pointer harness vào CLAUDE.md"
- **Xóa bảng phân chia vai trò CLAUDE.md vs orchestrator** — Đã đơn giản hóa thành chính sách pointer, bảng không còn cần thiết

### Thêm mới

- **Phase 2-1: Chế độ thực thi hybrid** — Ngoài nhóm agent / subagent, thêm mẫu hybrid kết hợp chế độ theo từng Phase. Nêu rõ các tổ hợp thường dùng (thu thập song song→tích hợp đồng thuận, tạo nhóm→xác thực, tái cấu trúc nhóm giữa các Phase)
- **Bảng so sánh chế độ thực thi Phase 2-1** — Cung cấp 3 đặc điểm nhóm/subagent/hybrid và 3 bước ra quyết định
- **Phase 5-0 mẫu orchestrator hybrid** — Quy tắc ghi rõ chế độ thực thi ở đầu mỗi Phase khi dùng cấu hình hybrid
- **Truyền dữ liệu dựa trên giá trị trả về Phase 5-1** — Thêm chiến lược truyền dữ liệu dành riêng cho chế độ subagent (ngoài message/task/file + return value)
- **Tổ hợp khuyến nghị Phase 5-1 (subagent/hybrid)** — Nêu rõ tổ hợp truyền dữ liệu khuyến nghị trong chế độ subagent và hybrid ngoài chế độ nhóm

## [1.1.0] - 2026-04-05

### Thêm mới

- **Phase 0: Kiểm tra hiện trạng** — Khi trigger, kiểm tra trạng thái harness hiện có trước tiên và phân nhánh sang 3 hướng: xây mới / mở rộng hiện có / vận hành & bảo trì
- **Ma trận chọn Phase khi mở rộng hiện có** — Bảng quyết định ghi rõ Phase cần thiết cho từng loại: thêm agent/thêm skill/thay đổi kiến trúc
- **Đồng bộ CLAUDE.md tạm thời Phase 3/4** — Cập nhật CLAUDE.md ngay sau khi tạo agent/skill (đề kháng ngắt phiên)
- **Phase 5-4: Đăng ký context harness vào CLAUDE.md** — Ghi lại cấu trúc nhóm agent, danh sách skill, quy tắc thực thi, cấu trúc thư mục, lịch sử thay đổi. Bao gồm bảng phân chia vai trò CLAUDE.md vs orchestrator
- **Phase 5-5: Hỗ trợ tác vụ tiếp theo** — Orchestrator description bắt buộc bao gồm từ khóa tiếp theo, bước kiểm tra context Phase 0 tự động phân biệt lần chạy đầu/lần chạy lại một phần/lần chạy mới
- **Đường dẫn sửa đổi orchestrator Phase 5** — Hướng dẫn sửa đổi orchestrator thay vì tạo mới khi mở rộng hiện có
- **Phase 7: Cơ chế tiến hóa harness** — Thu thập phản hồi sau mỗi lần chạy → ánh xạ loại phản hồi đến mục tiêu sửa đổi → ghi lịch sử thay đổi → trigger tiến hóa tự động
- **Phase 7-5: Quy trình vận hành/bảo trì** — 4 bước: kiểm tra hiện trạng→sửa đổi dần dần→đồng bộ CLAUDE.md→xác thực thay đổi
- **Trigger vận hành/bảo trì trong description** — Các từ khóa: 'kiểm tra harness', 'kiểm toán harness', 'hiện trạng harness', 'đồng bộ agent/skill'
- **Tăng cường checklist đầu ra** — Thêm các mục: đồng bộ CLAUDE.md hoàn tất, ghi lịch sử thay đổi, kiểm tra context Phase 0
- Thêm Phase 0 (kiểm tra context) vào template orchestrator — áp dụng cho cả chế độ nhóm agent/subagent
- Thêm mẫu từ khóa tác vụ tiếp theo vào template description orchestrator

### Thay đổi

- Mở rộng nguyên tắc cốt lõi từ 2 → 4 (thêm đăng ký CLAUDE.md, hệ thống tiến hóa)
- **Thống nhất "nhật ký tiến hóa" → "lịch sử thay đổi"** — Thống nhất tên và schema (4 cột: ngày/nội dung thay đổi/mục tiêu/lý do) trên toàn bộ các phần
- **Phase 1 Bước 3** — Thay đổi để phân tích xung đột dựa trên kết quả kiểm tra Phase 0 (loại bỏ trùng lặp)
- **Khối code template CLAUDE.md 5-4** — Sửa lỗi render lồng nhau (3 backtick→4 backtick)
- **Mở rộng bảng phân chia vai trò** — Thêm hàng danh sách skill, cấu trúc thư mục, lịch sử thay đổi
- **Template orchestrator** — Thêm bước kiểm tra context Phase 0, hướng dẫn từ khóa tác vụ tiếp theo

## [1.0.1] - 2026-03-28

### Thay đổi

- Loại bỏ nội dung trùng lặp giữa SKILL.md ↔ references (330 dòng → 285 dòng)
  - Phase 2-1: Bảng/bullet so sánh chế độ thực thi → nguyên tắc cốt lõi + pointer agent-design-patterns.md
  - Phase 2-3: Bullet tiêu chí phân tách agent → tóm tắt 4 trục + pointer agent-design-patterns.md
  - Phase 3: Khối code template định nghĩa agent → liệt kê phần bắt buộc + pointer references
  - Phase 5-2: Bảng 5 hàng xử lý lỗi → nguyên tắc cốt lõi + pointer orchestrator-template.md

## [1.0.0] - 2026-03-27

### Thêm mới

- Meta-skill cấu hình harness dựa trên quy trình 6 Phase
- 6 mẫu kiến trúc agent (Pipeline, Fan-out/Fan-in, Expert Pool, Producer-Reviewer, Supervisor, Hierarchical Delegation)
- Hỗ trợ chế độ thực thi nhóm agent / subagent
- Hướng dẫn tạo skill dựa trên Progressive Disclosure
- Template orchestrator (chế độ nhóm agent + subagent)
- Hướng dẫn tích hợp QA agent (dựa trên 7 trường hợp bug thực tế)
- Phương pháp kiểm thử/đánh giá skill (so sánh With-skill vs Without-skill)
- 5 ví dụ cấu hình nhóm thực tế (nghiên cứu, tiểu thuyết, webtoon, đánh giá code, migration)
