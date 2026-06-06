# Hướng dẫn kiểm thử & cải thiện lặp lại Skill

Phương pháp xác thực chất lượng và cải thiện lặp lại skill được tạo bởi harness. Tài liệu tham khảo bổ sung cho Phase 6 của SKILL.md.

---

## Mục lục

1. [Tổng quan framework kiểm thử](#1-tổng-quan-framework-kiểm-thử)
2. [Cách viết prompt kiểm thử](#2-cách-viết-prompt-kiểm-thử)
3. [Kiểm thử thực thi: With-skill vs Baseline](#3-kiểm-thử-thực-thi-with-skill-vs-baseline)
4. [Đánh giá định lượng: Chấm điểm dựa trên Assertion](#4-đánh-giá-định-lượng-chấm-điểm-dựa-trên-assertion)
5. [Sử dụng agent chuyên biệt](#5-sử-dụng-agent-chuyên-biệt)
6. [Vòng cải thiện lặp lại](#6-vòng-cải-thiện-lặp-lại)
7. [Xác thực trigger Description](#7-xác-thực-trigger-description)
8. [Cấu trúc workspace](#8-cấu-trúc-workspace)

---

## 1. Tổng quan framework kiểm thử

Xác thực chất lượng skill là sự kết hợp của **đánh giá định tính** và **đánh giá định lượng**.

| Loại đánh giá | Phương pháp | Phù hợp cho skill |
|--------------|------------|------------------|
| **Định tính** | Người dùng trực tiếp review đầu ra | Phong cách văn bản, thiết kế, sáng tạo và chất lượng chủ quan |
| **Định lượng** | Chấm điểm tự động dựa trên assertion | Tạo file, trích xuất dữ liệu, tạo code và những gì có thể xác thực khách quan |

Vòng cốt lõi: **Viết → Chạy kiểm thử → Đánh giá → Cải thiện → Kiểm thử lại**

---

## 2. Cách viết prompt kiểm thử

### Nguyên tắc

Prompt kiểm thử phải là **câu cụ thể, tự nhiên giống như người dùng thực sự sẽ nhập**. Prompt trừu tượng hoặc nhân tạo có giá trị kiểm thử thấp.

### Ví dụ xấu

```
"Xử lý PDF"
"Trích xuất dữ liệu"
"Tạo biểu đồ"
```

### Ví dụ tốt

```
"Trong file 'Q4_DoanhThu_Final_v2.xlsx' ở thư mục Downloads, dùng cột C (doanh thu)
và cột D (chi phí) để thêm cột tỷ lệ lợi nhuận (%). Sau đó sắp xếp giảm dần
theo tỷ lệ lợi nhuận."
```

```
"Trích xuất bảng trang 3 trong PDF này và chuyển sang CSV. Tiêu đề bảng
có 2 dòng — dòng đầu là danh mục, dòng thứ hai mới là tên cột thực."
```

### Đa dạng prompt

- Kết hợp tone **trang trọng / thông thường**
- Kết hợp ý định **rõ ràng / ngụ ý** (nói thẳng định dạng file vs cần suy luận từ ngữ cảnh)
- Kết hợp tác vụ **đơn giản / phức tạp**
- Một số có viết tắt, lỗi chính tả, cách diễn đạt thông thường

### Phủ toàn diện

Bắt đầu với 2~3 prompt, thiết kế để bao gồm:
- 1 trường hợp sử dụng cốt lõi
- 1 trường hợp edge case
- (Tùy chọn) 1 tác vụ phức hợp

---

## 3. Kiểm thử thực thi: With-skill vs Baseline

### 3-1. Cấu trúc chạy so sánh

Đối với mỗi prompt kiểm thử, spawn **đồng thời** hai subagent:

**Chạy With-skill:**
```
Prompt: "{Prompt kiểm thử}"
Đường dẫn skill: {Đường dẫn skill}
Đường dẫn đầu ra: _workspace/iteration-N/eval-{id}/with_skill/outputs/
```

**Chạy Baseline:**
```
Prompt: "{Prompt kiểm thử}"  (giống hệt)
Skill: Không có
Đường dẫn đầu ra: _workspace/iteration-N/eval-{id}/without_skill/outputs/
```

### 3-2. Chọn Baseline

| Tình huống | Baseline |
|----------|---------|
| Tạo skill mới | Chạy cùng prompt không có skill |
| Cải thiện skill hiện có | Phiên bản skill trước khi sửa (bảo toàn snapshot) |

### 3-3. Ghi lại dữ liệu timing

Lưu `total_tokens` và `duration_ms` **ngay lập tức** từ thông báo hoàn thành subagent. Dữ liệu này chỉ truy cập được tại thời điểm thông báo và không thể phục hồi sau đó.

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

---

## 4. Đánh giá định lượng: Chấm điểm dựa trên Assertion

### 4-1. Viết Assertion

Khi đầu ra có thể xác thực khách quan, định nghĩa assertion để chấm điểm tự động.

**Assertion tốt:**
- Có thể phán đoán đúng/sai một cách khách quan
- Tên mô tả rõ đang kiểm tra gì chỉ qua kết quả
- Xác thực giá trị cốt lõi của skill

**Assertion xấu:**
- Luôn pass bất kể có skill hay không (v.d.: "Có đầu ra")
- Cần phán đoán chủ quan (v.d.: "Được viết tốt")

### 4-2. Xác thực có thể lập trình

Nếu assertion có thể xác thực bằng code, hãy viết script. Nhanh hơn và đáng tin cậy hơn kiểm tra bằng mắt, có thể tái sử dụng cho mỗi iteration.

### 4-3. Chú ý assertion non-discriminating

Assertion "pass 100% cho cả hai cấu hình" không đo lường được giá trị phân biệt của skill. Khi phát hiện assertion như vậy, xóa đi hoặc thay bằng assertion thách thức hơn.

### 4-4. Schema kết quả chấm điểm

```json
{
  "expectations": [
    {
      "text": "Đã thêm cột tỷ lệ lợi nhuận",
      "passed": true,
      "evidence": "Xác nhận cột 'profit_margin_pct' ở cột E"
    },
    {
      "text": "Đã sắp xếp giảm dần theo tỷ lệ lợi nhuận",
      "passed": false,
      "evidence": "Giữ nguyên thứ tự gốc không sắp xếp"
    }
  ],
  "summary": {
    "passed": 1,
    "failed": 1,
    "total": 2,
    "pass_rate": 0.50
  }
}
```

---

## 5. Sử dụng agent chuyên biệt

Sử dụng agent với vai trò chuyên biệt trong quá trình kiểm thử/đánh giá giúp nâng cao chất lượng.

### 5-1. Grader (Người chấm điểm)

Thực hiện chấm điểm dựa trên assertion, trích xuất claim có thể xác thực từ đầu ra và xác thực chéo.

**Vai trò:**
- Phán định pass/fail cho mỗi assertion + đưa ra bằng chứng
- Trích xuất và xác thực các tuyên bố thực tế từ đầu ra
- Phản hồi về chất lượng của eval (đề xuất khi assertion quá dễ hoặc mơ hồ)

### 5-2. Comparator (Người so sánh mù)

Ẩn danh hai đầu ra thành A/B, phán định chất lượng trong trạng thái không biết cái nào dùng skill.

**Thời điểm dùng:** Khi muốn xác nhận nghiêm ngặt "phiên bản mới có thực sự tốt hơn không?". Có thể bỏ qua trong cải thiện lặp lại thông thường.

**Tiêu chí phán định:**
- Nội dung: Độ chính xác, tính đầy đủ
- Cấu trúc: Tổ chức, định dạng, tính sử dụng được
- Điểm tổng hợp

### 5-3. Analyzer (Người phân tích)

Phân tích mẫu thống kê từ dữ liệu benchmark:
- Assertion non-discriminating (cả hai cấu hình đều pass → không có khả năng phân biệt)
- Eval có phương sai cao (kết quả thay đổi lớn mỗi lần chạy → không ổn định)
- Trade-off thời gian/token (skill nâng cao chất lượng nhưng cũng tăng chi phí)

---

## 6. Vòng cải thiện lặp lại

### 6-1. Thu thập phản hồi

Cho người dùng xem đầu ra và nhận phản hồi. Phản hồi trống được hiểu là "không có vấn đề".

### 6-2. Nguyên tắc cải thiện

1. **Tổng quát hóa phản hồi** — Sửa hẹp chỉ khớp với ví dụ kiểm thử là overfitting. Sửa ở mức nguyên lý.
2. **Xóa những gì không tạo thêm giá trị** — Đọc transcript và xóa phần skill đang yêu cầu agent làm công việc không hiệu quả.
3. **Giải thích tại sao** — Dù phản hồi của người dùng ngắn gọn, hãy hiểu tại sao điều đó quan trọng và phản ánh sự hiểu đó vào skill.
4. **Đóng gói tác vụ lặp lại** — Nếu cùng helper script được tạo trong mọi lần kiểm thử, đóng gói trước vào `scripts/`.

### 6-3. Quy trình lặp lại

```
1. Sửa đổi skill
2. Chạy lại tất cả test case trong thư mục iteration-N+1/ mới
3. Trình bày kết quả cho người dùng (so sánh với iteration trước)
4. Thu thập phản hồi
5. Sửa đổi lại → Lặp lại
```

**Điều kiện dừng:**
- Người dùng hài lòng
- Tất cả phản hồi đều trống (tất cả đầu ra không có vấn đề)
- Không còn cải thiện đáng kể

### 6-4. Mẫu Bản nháp → Đọc lại

Khi sửa đổi skill, viết bản nháp rồi **đọc lại với góc nhìn mới** và cải thiện. Không cố viết hoàn hảo ngay từ đầu, hãy qua chu kỳ nháp-review.

---

## 7. Xác thực trigger Description

### 7-1. Viết Trigger Eval Queries

Viết 20 eval query — 10 should-trigger + 10 should-NOT-trigger.

**Tiêu chí chất lượng query:**
- Câu cụ thể, tự nhiên giống như người dùng thực sự sẽ nhập
- Bao gồm chi tiết cụ thể như đường dẫn file, ngữ cảnh cá nhân, tên cột, tên công ty
- Đa dạng về độ dài, tone, định dạng
- Tập trung vào **trường hợp ranh giới (edge case)** hơn là câu hỏi có đáp án rõ ràng

**Should-trigger queries (8~10 cái):**
- Cùng ý định với cách diễn đạt khác nhau (trang trọng/thông thường)
- Rõ ràng cần skill nhưng không nói rõ tên skill/loại file
- Trường hợp sử dụng ít phổ biến
- Cạnh tranh với skill khác nhưng skill này phải thắng

**Should-NOT-trigger queries (8~10 cái):**
- **Near-miss là cốt lõi** — Query có từ khóa tương tự nhưng công cụ/skill khác phù hợp hơn
- Query hoàn toàn không liên quan ("viết hàm Fibonacci") không có giá trị kiểm thử
- Domain liền kề, diễn đạt mơ hồ, từ khóa trùng nhưng ngữ cảnh khác

### 7-2. Xác thực xung đột skill hiện có

Kiểm tra description của skill mới có chồng chéo vùng trigger của skill hiện có không:

1. Thu thập description của danh sách skill hiện có
2. Kiểm tra should-trigger query của skill mới có kích hoạt nhầm skill hiện có không
3. Khi phát hiện xung đột, mô tả điều kiện ranh giới rõ hơn trong description

### 7-3. Tự động hóa (Tính năng nâng cao tùy chọn)

Khi cần tối ưu hóa description:

1. Chia 20 eval query thành Train(60%) / Test(40%)
2. Đo độ chính xác trigger với description hiện tại
3. Phân tích trường hợp thất bại để tạo description cải tiến
4. Chọn description tốt nhất theo Test set (không phải Train set — tránh overfitting)
5. Lặp lại tối đa 5 lần

> Quá trình này được thực hiện bằng script tự động dùng `claude -p`. Chi phí token cao nên chỉ thực thi ở bước cuối sau khi skill đã đủ ổn định.

---

## 8. Cấu trúc workspace

Cấu trúc thư mục quản lý có hệ thống kết quả kiểm thử/đánh giá:

```
{skill-name}-workspace/
├── iteration-1/
│   ├── eval-tên-mô-tả-1/
│   │   ├── eval_metadata.json
│   │   ├── with_skill/
│   │   │   ├── outputs/
│   │   │   ├── timing.json
│   │   │   └── grading.json
│   │   └── without_skill/
│   │       ├── outputs/
│   │       ├── timing.json
│   │       └── grading.json
│   ├── eval-tên-mô-tả-2/
│   │   └── ...
│   └── benchmark.json
├── iteration-2/
│   └── ...
└── evals/
    └── evals.json
```

**Quy tắc:**
- Thư mục eval dùng **tên mô tả** thay vì số (v.d.: `eval-trich-xuat-bang-nhieu-trang`)
- Mỗi iteration được bảo toàn trong thư mục riêng biệt (cấm ghi đè iteration trước)
- Không xóa `_workspace/` — để xác thực hậu kỳ và audit
