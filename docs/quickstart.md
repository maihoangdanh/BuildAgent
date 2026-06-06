# Quickstart — Cài BuildAgent (Harness) từ GitHub

> **Harness là gì?** Một meta-skill cho Claude Code. Cài một lần, dùng mãi mãi. Ở bất kỳ project nào, chỉ cần nói **"xây harness cho dự án này"** — Harness phân tích domain của bạn và tự sinh ra đội agent chuyên biệt + skill tương ứng ngay trong project đó.

---

## Bước 1 — Clone repo về máy

```bash
git clone https://github.com/maihoangdanh/BuildAgent.git
cd BuildAgent
```

---

## Bước 2 — Cài Harness skill

Chọn một trong hai cách:

### Cách A — Global Skill (Khuyến nghị)

Cài vào thư mục global, dùng được trong **mọi project**.

**macOS / Linux:**
```bash
cp -r skills/harness ~/.claude/skills/harness
```

**Windows (PowerShell):**
```powershell
Copy-Item -Recurse skills\harness "$env:USERPROFILE\.claude\skills\harness"
```

### Cách B — Plugin link (để phát triển / chỉnh sửa)

```bash
# Chạy từ thư mục BuildAgent
claude plugin link .
```

Kiểm tra đã cài chưa:
```bash
claude plugin list | grep harness
```

---

## Bước 3 — Bật Agent Teams

Harness cần tính năng Agent Teams của Claude Code.

**macOS / Linux:**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Lưu vĩnh viễn — thêm vào `~/.zshrc` hoặc `~/.bashrc`:
```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
```

**Windows (PowerShell — phiên hiện tại):**
```powershell
$env:CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = "1"
```

**Windows (vĩnh viễn):**
```powershell
[System.Environment]::SetEnvironmentVariable("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS", "1", "User")
```

---

## Bước 4 — Dùng thử

Mở Claude Code tại **bất kỳ project nào của bạn** và gõ:

```
Xây harness cho dự án này
```

Hoặc mô tả cụ thể hơn:

```
Xây harness cho dự án này — backend Node.js xử lý thanh toán
```

```
Xây harness cho dự án này — frontend React e-commerce bán hàng
```

**Kết quả:** Claude phân tích codebase, chọn mẫu kiến trúc phù hợp và sinh ra:
- `.claude/agents/` — file định nghĩa từng agent chuyên biệt
- `.claude/skills/` — skill hướng dẫn cách agent làm việc

---

## Gỡ cài đặt

**Nếu dùng Cách A (global skill):**

```bash
# macOS / Linux
rm -rf ~/.claude/skills/harness
```

```powershell
# Windows
Remove-Item -Recurse "$env:USERPROFILE\.claude\skills\harness"
```

**Nếu dùng Cách B (plugin link):**
```bash
claude plugin unlink harness
```
