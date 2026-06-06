# Quickstart — Cài Harness từ máy local

> **Harness là gì?** Một meta-skill cho Claude Code. Cài một lần, dùng mãi mãi. Ở bất kỳ project nào, chỉ cần nói **"xây harness cho dự án này"** — Harness phân tích domain của bạn và tự sinh ra đội agent chuyên biệt + skill tương ứng ngay trong project đó.

---

## Yêu cầu trước

- Claude Code `v2.x` trở lên (`claude --version`)
- Đã clone repo này về máy (bạn đang đọc file này nghĩa là đã xong bước này)

---

## Cách 1 — Cài làm Global Skill (Khuyến nghị)

Copy skill vào thư mục global của Claude Code. Sau đó dùng được trong **mọi project**.

**macOS / Linux:**
```bash
cp -r skills/harness ~/.claude/skills/harness
```

**Windows (PowerShell):**
```powershell
Copy-Item -Recurse skills\harness "$env:USERPROFILE\.claude\skills\harness"
```

Xong. Không cần làm gì thêm.

---

## Cách 2 — Link plugin local

Đăng ký repo này như một plugin cục bộ. Thay đổi trong repo sẽ được phản ánh ngay.

```bash
# Chạy từ thư mục repo này
claude plugin link .
```

Kiểm tra:
```bash
claude plugin list | grep harness
```

---

## Bật Agent Teams

Harness cần tính năng Agent Teams của Claude Code. Đặt biến môi trường:

**macOS / Linux:**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Để lưu vĩnh viễn, thêm dòng trên vào `~/.zshrc` hoặc `~/.bashrc`.

**Windows (PowerShell — chỉ cho phiên hiện tại):**
```powershell
$env:CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = "1"
```

**Windows (vĩnh viễn):**
```powershell
[System.Environment]::SetEnvironmentVariable("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS", "1", "User")
```

---

## Kiểm thử

Mở Claude Code ở **bất kỳ project nào của bạn** và thử:

```
Xây harness cho dự án này
```

Hoặc cụ thể hơn:

```
Xây harness cho dự án này — đây là backend API Node.js xử lý thanh toán
```

```
Xây harness cho dự án này — frontend React e-commerce
```

**Kết quả mong đợi:** Claude phân tích codebase, chọn mẫu kiến trúc phù hợp và sinh ra các file trong:
- `.claude/agents/` — định nghĩa từng agent chuyên biệt
- `.claude/skills/` — skill hướng dẫn agent làm việc

---

## Gỡ cài đặt

**Nếu dùng Cách 1 (global skill):**
```bash
# macOS / Linux
rm -rf ~/.claude/skills/harness

# Windows
Remove-Item -Recurse "$env:USERPROFILE\.claude\skills\harness"
```

**Nếu dùng Cách 2 (plugin link):**
```bash
claude plugin unlink harness
```
