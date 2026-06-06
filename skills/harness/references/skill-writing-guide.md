# Hướng dẫn viết Skill

Hướng dẫn viết chi tiết để nâng cao chất lượng skill được tạo bởi harness. Tài liệu tham khảo bổ sung cho Phase 4 của SKILL.md.

---

## Mục lục

1. [Mẫu viết Description](#1-mẫu-viết-description)
2. [Phong cách viết nội dung](#2-phong-cách-viết-nội-dung)
3. [Mẫu định nghĩa định dạng đầu ra](#3-mẫu-định-nghĩa-định-dạng-đầu-ra)
4. [Mẫu viết ví dụ](#4-mẫu-viết-ví-dụ)
5. [Mẫu Progressive Disclosure](#5-mẫu-progressive-disclosure)
6. [Tiêu chí quyết định đóng gói script](#6-tiêu-chí-quyết-định-đóng-gói-script)
7. [Chuẩn data schema](#7-chuẩn-data-schema)
8. [Những gì không nên đưa vào skill](#8-những-gì-không-nên-đưa-vào-skill)

---

## 1. Mẫu viết Description

Description là cơ chế trigger duy nhất của skill. Claude quyết định có dùng skill hay không chỉ dựa trên name + description trong danh sách `available_skills`.

### Hiểu cơ chế trigger

Claude có xu hướng không gọi skill cho tác vụ đơn giản mà nó có thể xử lý dễ dàng bằng công cụ mặc định. Yêu cầu đơn giản như "đọc PDF này cho tôi" có thể không trigger dù description hoàn hảo. Tác vụ càng phức tạp, nhiều bước và chuyên biệt thì xác suất trigger skill càng cao.

### Nguyên tắc viết

1. Mô tả cả **những gì skill làm** + **tình huống trigger cụ thể**
2. Ghi rõ điều kiện ranh giới phân biệt với trường hợp tương tự nhưng không nên trigger
3. Hơi "pushy" một chút — Bù đắp xu hướng đánh giá trigger bảo thủ của Claude

### Ví dụ tốt

```yaml
description: "Thực hiện tất cả tác vụ PDF bao gồm đọc, trích xuất văn bản/bảng,
  hợp nhất, tách, xoay, watermark, mã hóa/giải mã, OCR. Khi nhắc đến file
  .pdf hoặc yêu cầu đầu ra PDF, phải dùng skill này. Đặc biệt hữu ích khi
  cần chuyển đổi/chỉnh sửa/phân tích, không phải chỉ đơn giản là 'đọc'."
```

```yaml
description: "Tất cả tác vụ bảng tính bao gồm thêm cột, tính toán công thức,
  định dạng, biểu đồ, làm sạch dữ liệu cho file Excel/CSV/TSV. Khi người dùng
  nhắc đến file bảng tính — dù chỉ nói thông thường ('xlsx trong thư mục
  downloads') — hãy dùng skill này."
```

### Ví dụ xấu

- `"Skill xử lý dữ liệu"` — Quá mơ hồ, không rõ file/tác vụ nào
- `"Tác vụ liên quan đến PDF"` — Không liệt kê hành động cụ thể, không mô tả tình huống trigger

---

## 2. Phong cách viết nội dung

### Nguyên tắc Why-First

LLM hiểu lý do sẽ phán đoán đúng ngay cả trong trường hợp edge case. Truyền đạt ngữ cảnh hiệu quả hơn quy tắc cưỡng bức.

**Ví dụ xấu:**
```markdown
ALWAYS use pdfplumber for table extraction. NEVER use PyPDF2 for tables.
```

**Ví dụ tốt:**
```markdown
Dùng pdfplumber để trích xuất bảng. PyPDF2 chuyên về trích xuất văn bản
nên không bảo toàn được cấu trúc hàng/cột của bảng. pdfplumber nhận ra
ranh giới ô và trả về dữ liệu có cấu trúc.
```

### Nguyên tắc tổng quát hóa

Khi phát hiện vấn đề từ phản hồi hoặc kết quả kiểm thử, thay vì sửa hẹp chỉ khớp với ví dụ cụ thể, hãy **tổng quát hóa ở mức nguyên lý**.

**Sửa overfitting:**
```markdown
Nếu có cột "Doanh thu Q4", chuyển cột đó sang dạng số.
```

**Sửa đã tổng quát hóa:**
```markdown
Nếu tên cột có từ khóa ám chỉ giá trị số như "doanh thu", "số tiền", "số lượng",
chuyển cột đó sang kiểu số. Nếu chuyển đổi thất bại, giữ nguyên giá trị gốc.
```

### Giọng văn mệnh lệnh

Dùng dạng mệnh lệnh/chỉ thị "~làm điều này", "~hãy làm" thay vì "~có thể", "~nên". Skill là tài liệu chỉ thị.

### Tiết kiệm context

Context window là tài nguyên chung. Hỏi mình xem mỗi câu có xứng đáng với chi phí token không:
- "Claude đã biết điều này chưa?" → Xóa
- "Không có giải thích này thì Claude có mắc lỗi không?" → Giữ
- "Một ví dụ cụ thể có hiệu quả hơn giải thích dài không?" → Thay bằng ví dụ

---

## 3. Mẫu định nghĩa định dạng đầu ra

Dùng trong skill có định dạng đầu ra quan trọng:

```markdown
## Cấu trúc báo cáo
Theo chính xác template sau:

# [Tiêu đề]
## Tóm tắt
## Phát hiện chính
## Khuyến nghị
```

Định nghĩa định dạng ngắn gọn, bao gồm ví dụ thực tế sẽ hiệu quả hơn.

---

## 4. Mẫu viết ví dụ

Ví dụ hiệu quả hơn giải thích dài:

```markdown
## Định dạng commit message

**Ví dụ 1:**
Đầu vào: Thêm xác thực người dùng dựa trên JWT token
Đầu ra: feat(auth): Triển khai xác thực dựa trên JWT

**Ví dụ 2:**
Đầu vào: Sửa bug nút hiển thị mật khẩu trên trang login không hoạt động
Đầu ra: fix(login): Sửa nút toggle hiển thị mật khẩu
```

---

## 5. Mẫu Progressive Disclosure

### Mẫu 1: Tách theo domain

```
bigquery-skill/
├── SKILL.md (Tổng quan + hướng dẫn chọn domain)
└── references/
    ├── finance.md (Chỉ số doanh thu, billing)
    ├── sales.md (Cơ hội, pipeline)
    └── product.md (Sử dụng API, tính năng)
```

Khi người dùng hỏi về doanh thu, chỉ load finance.md.

### Mẫu 2: Chi tiết có điều kiện

```markdown
# Xử lý DOCX

## Tạo tài liệu
Tạo tài liệu mới bằng docx-js. → Tham khảo [DOCX-JS.md](references/docx-js.md).

## Chỉnh sửa tài liệu
Chỉnh sửa đơn giản thì sửa trực tiếp XML.
**Khi cần track changes**: Tham khảo [REDLINING.md](references/redlining.md)
```

### Mẫu 3: Cấu trúc file reference lớn

File reference trên 300 dòng phải có mục lục ở đầu:

```markdown
# API Reference

## Mục lục
1. [Xác thực](#xác-thực)
2. [Danh sách endpoint](#danh-sách-endpoint)
3. [Mã lỗi](#mã-lỗi)
4. [Rate limit](#rate-limit)

---

## Xác thực
...
```

---

## 6. Tiêu chí quyết định đóng gói script

Quan sát transcript của agent khi kiểm thử. Đây là dấu hiệu cần đóng gói:

| Tín hiệu | Hành động |
|---------|----------|
| 3/3 kiểm thử tạo cùng helper script | Đóng gói vào `scripts/` |
| Chạy pip install/npm install giống nhau mỗi lần | Ghi rõ bước cài dependency trong skill |
| Lặp lại cùng cách tiếp cận nhiều bước | Mô tả trong skill như quy trình chuẩn |
| Mỗi lần gặp lỗi giống nhau rồi áp dụng cùng cách giải quyết | Ghi vấn đề đã biết và giải pháp trong skill |

Script đóng gói phải qua kiểm thử thực thi.

---

## 7. Chuẩn data schema

Dùng schema chuẩn để đảm bảo nhất quán khi trao đổi dữ liệu giữa các skill. Có thể dùng để kiểm thử/đánh giá skill được tạo bởi harness.

### eval_metadata.json

Metadata của mỗi test case:

```json
{
  "eval_id": 0,
  "eval_name": "tên-mô-tả-ở-đây",
  "prompt": "Prompt tác vụ của người dùng",
  "assertions": [
    "Đầu ra chứa X",
    "File đã được tạo theo định dạng Y"
  ]
}
```

### grading.json

Kết quả chấm điểm dựa trên assertion:

```json
{
  "expectations": [
    {
      "text": "Đầu ra chứa 'Hà Nội'",
      "passed": true,
      "evidence": "Xác nhận 'Trích xuất dữ liệu khu vực Hà Nội' ở bước 3"
    }
  ],
  "summary": {
    "passed": 2,
    "failed": 1,
    "total": 3,
    "pass_rate": 0.67
  }
}
```

**Lưu ý tên trường:** Dùng chính xác `text`, `passed`, `evidence` (không dùng biến thể như `name`/`met`/`details`).

### timing.json

Đo thời gian thực thi/token:

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

Lưu `total_tokens` và `duration_ms` **ngay lập tức** từ thông báo hoàn thành subagent. Dữ liệu này chỉ truy cập được tại thời điểm thông báo và không thể phục hồi sau đó.

---

## 8. Những gì không nên đưa vào skill

- README.md, CHANGELOG.md, INSTALLATION_GUIDE.md và tài liệu phụ trợ khác
- Meta thông tin về quá trình tạo skill (kết quả kiểm thử, lịch sử lặp lại)
- Tài liệu hướng dẫn người dùng (skill là tài liệu chỉ thị cho AI agent)
- Kiến thức phổ biến mà Claude đã biết

---

## 9. Thiết kế tái sử dụng Skill

Trước khi tạo skill mới, kiểm tra trùng lặp với skill hiện có. Khi xây harness lặp đi lặp lại, dễ tích lũy các skill chức năng trùng nhau dưới các tên khác nhau.

| Tình huống | Hành động |
|----------|----------|
| Skill hiện có bao gồm hoàn toàn chức năng mới | Cấm tạo mới — Kết nối skill hiện có với agent |
| Skill hiện có bao gồm một phần và có thể tổng quát hóa | Tổng quát hóa và mở rộng skill hiện có |
| Bao gồm một phần là domain đặc hóa có chủ ý | Tiến hành tạo mới — Giữ làm skill riêng biệt |
| Phạm vi chức năng hoàn toàn khác | Tiến hành tạo mới |

**Nguyên tắc:** Skill càng tập trung vào một vai trò, khả năng tái sử dụng càng cao và trùng lặp càng giảm. Nếu có hai vai trò trở lên, trước tiên cân nhắc xem có thể phân tách không.

### Tổng quát hóa đến đâu

Tổng quát hóa có thể tiếp tục vô hạn nên dừng lại ở **phạm vi trách nhiệm đã định**. Giữ lại domain đặc hóa có chủ ý, chỉ loại bỏ phụ thuộc tình cờ.

Ví dụ: Skill "PDF đánh giá rủi ro fintech"

| Bước | Kết quả |
|------|--------|
| Loại bỏ phụ thuộc fintech | "PDF báo cáo đánh giá" — Nếu phạm vi trách nhiệm là báo cáo đánh giá thì dừng ở đây |
| Loại bỏ phụ thuộc đánh giá | "Định dạng PDF" — Nếu đã tồn tại thì không tạo skill riêng mà tái sử dụng |

Nếu phạm vi trách nhiệm là "đánh giá rủi ro fintech" là đặc hóa có chủ ý thì không tổng quát hóa mà giữ làm skill riêng biệt.

Hành vi của agent phụ thuộc vào skill đó có thể thay đổi. Kiểm tra dependency trước khi mở rộng, và phản ánh phạm vi sử dụng mở rộng vào description.
