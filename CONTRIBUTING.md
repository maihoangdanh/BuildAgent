# Đóng góp cho Harness

Cảm ơn bạn đã cân nhắc đóng góp cho **Harness** — nhà máy meta-skill của Claude Code giúp thiết kế nhóm agent và tạo skill.

Tài liệu này bao gồm: SLA phản hồi, cách đóng góp, thiết lập môi trường phát triển, quy tắc PR, quy ước commit, quy tắc ứng xử và danh sách maintainer.

---

## SLA phản hồi (cam kết)

Đây là mục tiêu thời gian phản hồi của maintainer cho repo này. Các mục tiêu được đặt **thận trọng** để nhóm maintainer nhỏ có thể duy trì thực tế khi mở rộng.

| Bề mặt | Mục tiêu | Ghi chú |
|--------|----------|---------|
| PR — phản hồi lần đầu | **< 72h** | Ngày làm việc. "Phản hồi lần đầu" nghĩa là tối thiểu thêm nhãn + một bình luận xác nhận PR. |
| Phân loại & gắn nhãn Issue | **< 48h** | Mỗi issue mới sẽ được gỡ nhãn `needs-triage` và thêm nhãn loại (`bug` / `enhancement` / `question` / `discussion`) trong vòng 48h. |
| Giải quyết bug (P0 / P1) | **< 14 ngày** | P0 = mất dữ liệu / bảo mật / lỗi cài đặt. P1 = đường chính bị hỏng. P2/P3 theo dõi trên roadmap không có SLA cứng. |
| Báo cáo bảo mật | **< 7 ngày** | Xác nhận ban đầu trong 7 ngày. Mục tiêu vá lỗi 30 ngày. Xem phần **Bảo mật** bên dưới để biết kênh riêng tư. |
| Chu kỳ release | **2 tuần một lần** | Tag 2 tuần/lần trừ khi không có gì đáng ship. Lỗi P0 có thể cắt một patch release ngoài lịch. |

Nếu chúng tôi bỏ lỡ SLA, vui lòng ping issue/PR — điều đó không thô lỗ, đó là vòng phản hồi đã thỏa thuận.

---

## Cách đóng góp

Các loại đóng góp khác nhau đi qua các điểm vào khác nhau. Chọn cái phù hợp.

### Báo cáo bug

- Mở issue bằng form **Bug report** (`.github/ISSUE_TEMPLATE/bug_report.yml`).
- Bắt buộc: phiên bản Claude Code, trạng thái flag `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`, các bước tái hiện, kết quả mong đợi vs thực tế, hệ điều hành.
- Tái hiện nhỏ (< 30 dòng) là lý tưởng. Nếu cần dự án đầy đủ để tái hiện, hãy link một fork công khai.

### Yêu cầu tính năng

- Mở issue bằng form **Feature request**.
- Chúng tôi cần một đoạn ngắn "vấn đề này giải quyết được gì". Nếu bạn có đề xuất, hãy đặt vào dạng sẵn sàng cho PR (mẫu nào trong 6 mẫu kiến trúc nhóm được mở rộng / thay thế?).

### Câu hỏi

- Mở issue bằng form **Question**, **hoặc** bắt đầu thread trên [GitHub Discussions](https://github.com/revfactory/harness/discussions) nếu vấn đề mang tính mở.

### Thảo luận (ý tưởng cỡ RFC)

- Ưu tiên GitHub Discussions. Chỉ chuyển thành issue khi có đồng thuận sơ bộ về hướng đi.

### Pull Request

- Xem **Hướng dẫn Pull Request** bên dưới.
- PR nhỏ merge nhanh hơn. Bất cứ thứ gì > 400 dòng diff nên được thảo luận trước.

### Bảo mật

- **Không** mở issue công khai cho bất cứ điều gì có thể bị lợi dụng.
- Email: `robin.hwang@kakaocorp.com` với tiền tố chủ đề `[harness-security]`.
- Chúng tôi hướng tới xác nhận trong 7 ngày (xem bảng SLA).

---

## Thiết lập môi trường phát triển

### Yêu cầu trước

- Claude Code `v2.x` (cần Agent Teams API)
- Node.js `>= 18` (cho tooling cục bộ dùng trong CI)
- Git

### Cờ môi trường

Harness hiện yêu cầu tính năng Agent Teams thử nghiệm của Claude Code. Đặt cờ trong shell profile hoặc theo phiên:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Chúng tôi theo dõi dependency này trong `docs/experimental-dependency.md` (nếu Anthropic đưa cờ này vào stable, chúng tôi cập nhật README trong vòng 72h theo SLA ở trên).

### Liên kết plugin cục bộ

Để kiểm tra thay đổi trong phiên Claude Code cục bộ mà không cần publish lên marketplace:

```bash
# Từ thư mục checkout
claude plugin link ./harness

# Xác minh
claude plugin list | grep harness
```

Hủy liên kết bằng `claude plugin unlink harness` khi hoàn thành.

### Chạy meta-skill

```bash
claude "xây harness cho nhóm đánh giá rủi ro fintech"
```

Các agent và skill được tạo sẽ nằm trong `.claude/agents/` và `.claude/skills/` của dự án đích.

### Kiểm thử & linting

- Markdown lint: `npx markdownlint '**/*.md'`
- YAML lint (issue templates & workflows): `npx yaml-lint .github/`
- Kiểm tra metadata skill: `python scripts/validate_skills.py` (nếu có)

CI chạy những lệnh này trên mỗi PR. Thực thi cục bộ được khuyến khích nhưng không bắt buộc — chúng tôi sẽ không chặn vì lỗi CI có thể sửa dễ dàng khi merge.

---

## Hướng dẫn Pull Request

### Đặt tên nhánh

Dùng dạng `type/short-description`:

| Tiền tố | Dùng cho | Ví dụ |
|---------|----------|-------|
| `feat/` | Tính năng mới thấy được của người dùng | `feat/expert-pool-variance-mode` |
| `fix/` | Sửa bug | `fix/agent-teams-flag-detection` |
| `docs/` | Chỉ thay đổi tài liệu | `docs/quickstart-gemini-section` |
| `refactor/` | Cấu trúc nội bộ, không thay đổi hành vi | `refactor/skill-loader-split` |
| `chore/` | Build, deps, housekeeping | `chore/upgrade-markdownlint` |
| `test/` | Chỉ tests | `test/fan-out-fan-in-e2e` |

### Ngôn ngữ commit message

- **Tiếng Anh và tiếng Việt đều được chấp nhận.** Viết bằng ngôn ngữ bạn diễn đạt chính xác hơn.
- Nếu thay đổi sẽ xuất hiện trong CHANGELOG hoặc release notes, vui lòng cung cấp tiêu đề tiếng Anh trong phần mô tả PR để người đọc từ nơi khác có thể hiểu.

### Template PR

Mỗi body PR được điền sẵn từ `.github/PULL_REQUEST_TEMPLATE.md`. Vui lòng điền:

- **Tóm tắt** (what & why, 2–4 câu)
- **Động lực** (link issue, tham chiếu nghiên cứu, hoặc lý do 1 dòng)
- **Phạm vi thay đổi** (checklist các bề mặt bị ảnh hưởng)
- **Kiểm thử** (những gì bạn đã chạy / thêm)
- **CHANGELOG** (bạn có cập nhật `CHANGELOG.md` không? Y/N/NA)
- **Tác động SemVer** (patch / minor / major — xem phần tiếp theo)

### Kỳ vọng review

- Cần một review chấp thuận từ maintainer.
- Chúng tôi cố gắng phản hồi PR trong 72h (xem SLA). Nếu bị chặn, hãy ping.

---

## Quy ước Commit Message

Chúng tôi tuân theo biến thể nhẹ của **Conventional Commits** ánh xạ trực tiếp đến SemVer.

```
<type>(<scope>)!: <tóm tắt ngắn>

<body — tùy chọn>

<footer — tùy chọn>
```

### Loại & ánh xạ SemVer

| Loại commit | Tác động SemVer | Ví dụ |
|-------------|-----------------|-------|
| `feat!:` hoặc `BREAKING CHANGE:` trong footer | **major** (v.d. 1.x → 2.0) | `feat!: đổi tên mẫu chính "Supervisor" → "Orchestrator"` |
| `feat:` | **minor** (v.d. 1.2 → 1.3) | `feat: thêm chỉ số phương sai Producer-Reviewer` |
| `fix:` | **patch** (v.d. 1.2.3 → 1.2.4) | `fix: sửa nhận diện flag trên zsh` |
| `docs:` / `chore:` / `refactor:` / `test:` | không bump release | `docs: làm rõ lộ trình Gemini` |

- Tóm tắt tiếng Việt được chấp nhận: `feat: thêm chỉ số phân tán cho mẫu expert pool`.
- Hậu tố `!` (hoặc footer `BREAKING CHANGE:`) là **trigger duy nhất** cho major version. Đừng dùng bừa.

### Gắn tag release

- Release được cắt 2 tuần một lần (xem SLA).
- Gắn tag từ `main` sau khi CI pass và CHANGELOG được cập nhật.
- Tag theo định dạng `vMAJOR.MINOR.PATCH` (v.d. `v1.3.0`).

---

## Quy tắc ứng xử

Dự án này tuân thủ **Contributor Covenant v1.4** — tóm tắt:

- Chào đón và hòa nhập. Giả định thiện chí.
- Không quấy rối, không tấn công cá nhân, không ngôn ngữ phân biệt đối xử.
- Phê bình ý tưởng, không phê bình người. Hỗ trợ tuyên bố bằng tham chiếu khi có thể.
- Maintainer có thể kiểm duyệt, chỉnh sửa hoặc xóa bình luận/commit/issue/PR vi phạm các nguyên tắc này, và có thể cấm người vi phạm.

Toàn văn: <https://www.contributor-covenant.org/version/1/4/code-of-conduct/>

Báo cáo vi phạm Quy tắc ứng xử riêng tư đến `robin.hwang@kakaocorp.com` với tiền tố chủ đề `[harness-coc]`.

---

## Maintainer

| Vai trò | Handle | Lĩnh vực |
|---------|--------|----------|
| Lead maintainer | [@revfactory](https://github.com/revfactory) | Định hướng dự án, release, review cuối |
| Contributor | [@hnts03](https://github.com/hnts03) | Template skill, tài liệu tiếng Hàn |
| Contributor | [@JunghwanNA](https://github.com/JunghwanNA) | Mẫu agent, kiểm thử tích hợp |
| Contributor | [@shaun0927](https://github.com/shaun0927) | Tooling, CI, hạ tầng |

Contributor mới được thêm vào đây sau khi đóng góp bền vững (không chỉ một PR). Để lại ghi chú trong Discussion nếu bạn muốn thảo luận về lộ trình trở thành maintainer.

---

## Giấy phép

Khi đóng góp, bạn đồng ý rằng đóng góp của mình sẽ được cấp phép theo cùng giấy phép với repo này (xem [`LICENSE`](./LICENSE)).
