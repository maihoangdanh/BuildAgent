# Dependency vào Cờ Thử nghiệm

> **Trạng thái:** Đang hoạt động · **Người phụ trách:** revfactory · **Cập nhật lần cuối:** 2026-04-18 · **SLA:** Xem [Cam kết Giám sát](#cam-kết-giám-sát)

Tài liệu này giải thích tại sao `harness` yêu cầu `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, ba tương lai khả dĩ của cờ đó, và repo này sẽ làm gì trong từng trường hợp — với các cam kết có thời hạn để các tổ chức doanh nghiệp có thể lập kế hoạch.

---

## Trạng thái hiện tại

### Tại sao cần cờ này

`harness` là nhà máy meta-skill được xây dựng trên **Agent Teams API** của Claude Code. Ba primitive của Claude Code được gọi nội bộ mỗi khi người dùng chạy `claude "xây harness cho <domain>"`:

| Primitive | Mục đích | Bị cổng bởi cờ? |
|-----------|---------|-----------------|
| `TeamCreate` | Khởi tạo nhóm đa agent với context dùng chung | **Có** |
| `SendMessage` | Định tuyến tin nhắn giữa thành viên nhóm (supervisor ↔ worker) | **Có** |
| `TaskCreate` | Tạo subtask chạy dài trong một nhóm | **Có** |
| `Agent` tool (gọi trực tiếp) | Dispatch agent đơn lẻ | Không (GA) |

Tất cả ba primitive bị cổng bởi cờ đều yêu cầu:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Nếu không đặt biến này trong shell khởi động `claude`, các nhóm do harness tạo ra sẽ fallback sang thực thi đơn agent, điều này phá vỡ im lặng các mẫu Pipeline / Fan-out-in / Supervisor / Hierarchical Delegation.

### Tài liệu tham khảo của Anthropic (bắt buộc đọc trước khi mở issue)

Cơ sở thiết kế và lộ trình của cờ này nằm trong ba bài đăng của Anthropic Engineering. Những người áp dụng harness nên đọc ít nhất bài đầu tiên:

1. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — định nghĩa danh mục "harness" Anthropic ủng hộ và hợp đồng agent chạy dài.
2. [Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) — các mẫu mà `harness` hệ thống hóa (Pipeline, Producer-Reviewer, Supervisor, v.v.).
3. [Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents) — con đường tương lai có thể thay thế cờ Thử nghiệm (xem Kịch bản B).

---

## Đồ thị phụ thuộc

```
harness (v1.2.0)
  └── Agent Teams API (Claude Code)
        ├── TeamCreate            ← EXPERIMENTAL_AGENT_TEAMS=1
        ├── SendMessage           ← EXPERIMENTAL_AGENT_TEAMS=1
        ├── TaskCreate            ← EXPERIMENTAL_AGENT_TEAMS=1
        └── Agent (gọi trực tiếp) ← GA (không phụ thuộc cờ)
              └── Lộ trình Anthropic
                    ├── Kịch bản A: Cờ bị xóa (thăng cấp GA)
                    ├── Kịch bản B: Managed Agents GA (con đường song song)
                    └── Kịch bản C: Thay đổi signature gây phá vỡ
```

**Đọc đồ thị này từ trên xuống:** harness phụ thuộc vào Agent Teams API, vốn phụ thuộc vào một cờ Thử nghiệm duy nhất, vốn phụ thuộc vào lộ trình của chính Anthropic. Nếu bất kỳ node nào ở trên thay đổi, repo này có trách nhiệm thích nghi trong SLA bên dưới.

---

## 3 Kịch bản

Mỗi kịch bản liệt kê **trigger phát hiện** (cách chúng tôi biết nó đã xảy ra), **các hành động T+24h / T+48h / T+72h** mà repo này cam kết, và **artifact thấy được của người dùng** tại mỗi checkpoint.

### Kịch bản A — Cờ bị xóa (Agent Teams thăng cấp GA)

**Trigger phát hiện:** Changelog Claude Code của Anthropic xuất bản "Agent Teams is now GA" **hoặc** binary `claude-code` không còn yêu cầu `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (được phát hiện bởi CI hàng đêm trong [P-13](#)).

**Xác suất (chủ quan):** Cao — đây là con đường mà ba bài đăng trên ám chỉ.

| Checkpoint | Hành động | Artifact |
|------------|-----------|---------|
| **T+24h** | Mở nhánh `feat/drop-experimental-flag`. Xóa dòng `export` khỏi mọi README / docs / Quickstart. Thêm giới hạn dưới `claude-code >= X.Y.Z` trong `plugin.json`. | Nhánh + PR (draft) |
| **T+48h** | Xuất bản `docs/migrating-from-experimental.md`. Cập nhật `docs/experimental-dependency.md` (file này) tiêu đề thành "không cần cờ kể từ vX.Y". Ghim issue GitHub: "Cần hành động: bỏ dòng export". | Hướng dẫn migration + issue ghim |
| **T+72h** | Ship release **v1.3.0** với: (a) entry CHANGELOG, (b) `gh release create` kèm ghi chú migration, (c) follow-up HN: "Chúng tôi đã bỏ cờ thử nghiệm". | Tag git `v1.3.0` + GH Release |

**Tác động đến người dùng:** Tích cực. Rào cản phê duyệt doanh nghiệp giảm — một checkbox ("không có cờ thử nghiệm") trở nên thỏa mãn được. Không có breaking change với code người dùng harness.

---

### Kịch bản B — Managed Agents đạt GA (con đường song song)

**Trigger phát hiện:** Anthropic xuất bản "[Managed Agents](https://www.anthropic.com/engineering/managed-agents) is generally available" với một CLI hoặc bề mặt SDK ổn định `claude-agents`.

**Xác suất (chủ quan):** Trung bình-cao trong 90 ngày. Managed Agents là mô hình thực thi phía server; điều phối nhóm client-side của harness **không** tự động chuyển đổi được.

| Checkpoint | Hành động | Artifact |
|------------|-----------|---------|
| **T+24h** | Mở PR `feat/managed-agents-compat`. Thêm scaffold `adapters/managed-agents/` ánh xạ 6 mẫu nhóm của harness sang lời gọi Managed Agents. Xác định các mẫu không tương thích (có thể: Hierarchical Delegation). | PR compat (draft) |
| **T+48h** | Xuất bản bài blog: **"Harness + Managed Agents: một tầng cao hơn, không bị thay thế"** trên Dev.to và trong repo. Tái định vị harness là tầng **thiết kế** xuất ra config Managed Agents, không phải đối thủ runtime. | Blog định vị cùng tồn tại |
| **T+72h** | Xuất bản `docs/managed-agents-migration.md` với ma trận per-pattern (mẫu nào trong 6 mẫu ánh xạ 1:1, mẫu nào cần viết lại). Cập nhật phần repo anh chị em của README. | Hướng dẫn migration |

**Ghi chú chiến lược:** harness tái định vị là **tầng trên của Managed Agents** — "Managed Agents chạy nhóm, harness thiết kế nó." Đây là khung cùng tồn tại trong §4.2 của kế hoạch GTM.

**Tác động đến người dùng:** Trung lập đến tích cực. Người dùng harness hiện tại tiếp tục hoạt động trên con đường cờ Thử nghiệm; người dùng mới có thể chọn đầu ra Managed Agents.

---

### Kịch bản C — Breaking change (đột biến API signature)

**Trigger phát hiện:** CI hàng đêm (`.github/workflows/nightly-compat.yml`, theo dõi là roadmap P-13) thất bại với build nightly mới nhất của Claude Code **hoặc** Changelog thông báo biến môi trường bị đổi tên / signature `TeamCreate` thay đổi.

**Xác suất (chủ quan):** Trung bình. API Thử nghiệm có thể bị đổi tên mà không có cửa sổ deprecation.

| Checkpoint | Hành động | Artifact |
|------------|-----------|---------|
| **T+0 đến T+24h** | Alert CI hàng đêm kích hoạt trong Slack/Discord. Tác giả mở nhánh `hotfix/compat-<date>`, vá các call site bị ảnh hưởng. Unit test xanh trên cả signature cũ + mới (nỗ lực tốt nhất). | Nhánh hotfix |
| **T+24h** | Merge hotfix. Push tag patch `v1.2.x`. Cập nhật hàng phiên bản Claude Code bị ảnh hưởng trong `docs/compatibility-matrix.md`. | Patch release `v1.2.x` |
| **T+72h** | Nếu thay đổi không tầm thường (ảnh hưởng hợp đồng công khai của harness), xuất bản thông báo ngắn trên tab Discussions của repo + X. Ngược lại, entry CHANGELOG là đủ. | Bài Discussions (có điều kiện) |

**Tác động đến người dùng:** Người dùng đã ghim với phiên bản Claude Code trước không bị ảnh hưởng. Người dùng trên phiên bản mới nhất nhận được patch trong cùng tuần.

---

## Cam kết Giám sát

Chúng tôi cam kết **SLA quan sát được** sau. Bỏ lỡ là cơ sở để mở issue với nhãn `sla-breach`.

| Sự kiện | SLA | Đo lường |
|---------|-----|---------|
| Anthropic xuất bản thay đổi Agent Teams / Managed Agents trong Changelog chính thức | Tài liệu này cập nhật trong **72 giờ** | So sánh timestamp bài đăng Changelog với dòng `Cập nhật lần cuối` của file này |
| CI hàng đêm phát hiện lỗi compat | Nhánh hotfix mở trong **24 giờ** | Timestamp chạy GitHub Actions vs timestamp tạo nhánh |
| Release ổn định Claude Code mới (minor hoặc major) | Thêm hàng `docs/compatibility-matrix.md` trong **7 ngày** | Diff ma trận tương thích |

**Nguồn chúng tôi theo dõi tích cực:**

- Release notes Claude Code — theo dõi qua RSS [Anthropic Engineering blog](https://www.anthropic.com/engineering)
- GitHub Releases `anthropics/claude-code` (tag nightly)
- Kênh Anthropic Discord `#claude-code` (tín hiệu cộng đồng)

---

## FAQ cho Người dùng Doanh nghiệp

### Q1. Chúng tôi trong ngành được quản lý chặt (tài chính, y tế, khu vực công) và không thể bật cờ `EXPERIMENTAL` trong production. Làm sao áp dụng harness?

**Nguyên nhân:** Nhiều framework tuân thủ (SOC 2 Type II, ISO 27001, K-ISMS) không cho phép tính năng không ổn định/preview trong production.
**Hành động:** Dùng harness **chỉ ở design-time**: chạy nó trong một workstation sandbox để tạo file `.claude/agents/` và `.claude/skills/`, sau đó commit các artifact được tạo vào repo production. Claude Code production không bao giờ cần cờ — chỉ runtime `TeamCreate` bị cổng bởi cờ mới cần. Các skill agent đơn lẻ được tạo tương thích với con đường GA.

### Q2. Nếu Agent Teams đạt GA (Kịch bản A), code do harness tạo của tôi có bị hỏng không?

**Nguyên nhân:** Thăng cấp GA trong Claude Code của Anthropic trước đây không gây breaking change cho artifact được tạo; cờ đơn giản là không còn bắt buộc.
**Hành động:** Không cần hành động nào từ người dùng cuối. Các file `.claude/agents/*.md` và `.claude/skills/*` của bạn là Markdown thuần và vẫn hợp lệ. Bạn sẽ có thể `unset CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` vào ngày GA. Chúng tôi sẽ xuất bản ghi chú migration trong 48 giờ (xem Kịch bản A).

### Q3. Bạn có đảm bảo SLA bằng văn bản không? Điều gì xảy ra nếu bạn bỏ lỡ?

**Nguyên nhân:** Doanh nghiệp cần cam kết có hợp đồng hoặc ít nhất quan sát được trước khi phê duyệt.
**Hành động:** Bảng SLA trên là **cam kết công khai** và được thực thi bởi: (a) một GitHub Action comment trên file này nếu dòng `Cập nhật lần cuối` của nó cũ hơn 72 giờ sau khi phát hiện sự kiện Changelog, (b) nhãn issue `sla-breach` mà người dùng có thể áp dụng, (c) nghĩa vụ post-mortem trong `CONTRIBUTING.md` cho bất kỳ vi phạm nào. Đây không phải SLA có trả phí — đây là cam kết cộng đồng. Để có SLA có trả phí, liên hệ maintainer (xem README của repo).

---

**Tài liệu liên quan:**
- [`docs/quickstart.md`](./quickstart.md) — Hướng dẫn cài đặt 5 phút
- [`docs/show-hn-launch-kit.md`](./show-hn-launch-kit.md) — Gói ra mắt công khai
- `docs/compatibility-matrix.md` *(chờ P-13)* — Bảng phiên bản Claude Code × harness
