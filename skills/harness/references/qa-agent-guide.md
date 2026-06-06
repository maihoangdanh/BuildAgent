# Hướng dẫn thiết kế QA Agent

Hướng dẫn tham khảo khi bao gồm QA agent trong build harness. Cung cấp phương pháp xác thực có hệ thống để bắt các lỗi dễ bị QA bỏ qua, dựa trên phân tích nguyên nhân gốc rễ từ các mẫu bug được phát hiện trong dự án thực tế (SatangSlide).

---

## Mục lục

1. Mẫu lỗi mà QA agent bỏ qua
2. Xác thực tích hợp (Integration Coherence Verification)
3. Nguyên tắc thiết kế QA agent
4. Template checklist xác thực
5. Template định nghĩa QA agent

---

## 1. Mẫu lỗi mà QA agent bỏ qua

### 1-1. Không nhất quán ranh giới (Boundary Mismatch)

Loại lỗi phổ biến nhất. Hai component được triển khai "đúng" riêng lẻ nhưng hợp đồng bị sai tại điểm kết nối.

| Ranh giới | Ví dụ không nhất quán | Lý do bỏ qua |
|----------|---------------------|------------|
| Phản hồi API → Hook frontend | API trả về `{ projects: [...] }`, hook kỳ vọng `SlideProject[]` | Xác thực riêng lẻ thì bình thường, không so sánh chéo |
| Tên trường phản hồi API → Định nghĩa type | API dùng `thumbnailUrl` (camelCase), type dùng `thumbnail_url` (snake_case) | Compiler không bắt được khi cast bằng TypeScript generic |
| Đường dẫn file → Link href | Trang ở `/dashboard/create` nhưng link trỏ `/create` | Không so sánh chéo cấu trúc file và href |
| Bản đồ chuyển tiếp trạng thái → Cập nhật status thực tế | Bản đồ định nghĩa `generating_template → template_approved`, code không có chuyển tiếp | Chỉ kiểm tra sự tồn tại của bản đồ, không truy vết tất cả code cập nhật |
| Endpoint API → Hook frontend | API tồn tại nhưng không có hook tương ứng (không được gọi) | Không ánh xạ 1:1 danh sách API và danh sách hook |
| Phản hồi ngay lập tức → Kết quả bất đồng bộ | API trả về ngay `{ status }`, frontend truy cập `data.failedIndices` | Không phân biệt phản hồi đồng bộ/bất đồng bộ, chỉ kiểm tra type |

### 1-2. Tại sao code review tĩnh không bắt được

- **Giới hạn của TypeScript generic**: `fetchJson<SlideProject[]>()` — ngay cả khi runtime response là `{ projects: [...] }` vẫn compile pass
- **`npm run build` pass ≠ hoạt động bình thường**: Type casting, `any`, generic thì build thành công nhưng thất bại lúc runtime
- **Khác nhau giữa xác thực sự tồn tại và xác thực kết nối**: "API có tồn tại không?" và "Phản hồi API có khớp với kỳ vọng của bên gọi không?" là hai xác thực hoàn toàn khác nhau

---

## 2. Xác thực tích hợp (Integration Coherence Verification)

Các lĩnh vực **xác thực so sánh chéo** phải bao gồm trong QA agent.

### 2-1. Xác thực chéo phản hồi API ↔ Type hook frontend

**Phương pháp**: So sánh call site `NextResponse.json()` của từng API route với tham số type `fetchJson<T>` của hook tương ứng.

```
Các bước xác thực:
1. Trích xuất shape của object truyền vào NextResponse.json() trong API route
2. Kiểm tra type T của fetchJson<T> trong hook tương ứng
3. So sánh shape và T có khớp không
4. Kiểm tra có wrapping không (nếu API trả về { data: [...] } thì hook có lấy .data ra không)
```

**Các mẫu cần đặc biệt chú ý:**
- Pagination API: `{ items: [], total, page }` vs frontend kỳ vọng mảng
- Không nhất quán giữa trường DB snake_case → phản hồi API camelCase → định nghĩa type frontend
- Khác biệt shape giữa phản hồi ngay lập tức (202 Accepted) và kết quả cuối cùng

### 2-2. Ánh xạ đường dẫn file ↔ Đường dẫn link/router

**Phương pháp**: Trích xuất URL path của page file trong `src/app/` và đối chiếu với tất cả giá trị `href`, `router.push()`, `redirect()` trong code.

```
Các bước xác thực:
1. Trích xuất pattern URL từ đường dẫn page.tsx trong src/app/
   - (group) → Xóa khỏi URL
   - [param] → Dynamic segment
2. Thu thập tất cả giá trị href=, router.push(, redirect( trong code
3. Xác nhận mỗi link có khớp với đường dẫn page thực tế không
4. Chú ý tiền tố URL của trang trong route group (v.d.: trong dashboard/)
```

### 2-3. Truy vết tính đầy đủ của chuyển tiếp trạng thái

**Phương pháp**: Trích xuất tất cả cập nhật `status:` trong code và đối chiếu với bản đồ chuyển tiếp trạng thái.

```
Các bước xác thực:
1. Trích xuất danh sách chuyển tiếp được phép từ bản đồ chuyển tiếp (STATE_TRANSITIONS)
2. Tìm kiếm pattern .update({ status: "..." }) trong tất cả API route
3. Xác nhận mỗi chuyển tiếp có được định nghĩa trong bản đồ không
4. Xác định chuyển tiếp được định nghĩa trong bản đồ nhưng không được thực thi trong code (chuyển tiếp chết)
5. Đặc biệt: Kiểm tra chuyển tiếp từ trạng thái trung gian (v.d.: generating_template) sang trạng thái cuối (template_approved) không bị thiếu
```

### 2-4. Ánh xạ 1:1 Endpoint API ↔ Hook frontend

**Phương pháp**: Liệt kê tất cả API route và frontend hook để kiểm tra có khớp không.

```
Các bước xác thực:
1. Trích xuất danh sách endpoint theo HTTP method từ route.ts trong src/app/api/
2. Trích xuất danh sách URL fetch call từ use*.ts trong src/hooks/
3. Xác định endpoint API không được gọi trong hook → Đánh dấu "Chưa dùng"
4. Phán định "Chưa dùng" là có chủ ý (quản trị API, v.v.) hay không (thiếu lời gọi)
```

---

## 3. Nguyên tắc thiết kế QA agent

### 3-1. Dùng loại general-purpose không phải Explore

Nếu QA agent là loại `Explore`, chỉ có thể đọc. Nhưng QA hiệu quả cần:
- Tìm kiếm pattern bằng Grep (trích xuất tất cả `NextResponse.json()`)
- Đối chiếu tự động bằng script (API shape vs hook type)
- Có thể sửa đổi khi cần

**Khuyến nghị**: Đặt loại `general-purpose`, nhưng ghi rõ giao thức "xác thực → báo cáo → yêu cầu sửa đổi" trong định nghĩa agent.

### 3-2. Checklist ưu tiên "So sánh chéo" hơn "Kiểm tra sự tồn tại"

| Checklist yếu | Checklist mạnh |
|--------------|---------------|
| Endpoint API có tồn tại không? | Shape phản hồi của endpoint API có khớp với type của hook tương ứng không? |
| Bản đồ chuyển tiếp trạng thái có được định nghĩa không? | Tất cả code cập nhật status có khớp với chuyển tiếp trong bản đồ không? |
| File page có tồn tại không? | Tất cả link trong code có trỏ đến trang thực sự tồn tại không? |
| TypeScript strict mode không? | Có type safety nào bị bypass bằng generic cast không? |

### 3-3. Nguyên tắc "Đọc cả hai bên đồng thời"

Để QA bắt được bug ranh giới, không thể chỉ đọc một bên. Phải:
- Đọc **đồng thời** API route **và** hook tương ứng
- Đọc **đồng thời** bản đồ chuyển tiếp trạng thái **và** code cập nhật thực tế
- Đọc **đồng thời** cấu trúc file **và** đường dẫn link

Ghi rõ nguyên tắc này trong định nghĩa agent.

### 3-4. Chạy QA ngay sau khi mỗi module hoàn thành, không phải sau build

Khi chỉ đặt QA vào "Phase 4: Sau khi hoàn tất toàn bộ" trong orchestrator:
- Bug tích lũy làm tăng chi phí sửa
- Không nhất quán ranh giới ban đầu lan truyền sang module tiếp theo

**Mẫu khuyến nghị**: Khi mỗi API backend hoàn thành, ngay lập tức xác thực chéo API đó và hook tương ứng (incremental QA).

---

## 4. Template checklist xác thực

Checklist xác thực tích hợp cho ứng dụng web để bao gồm trong định nghĩa QA agent.

```markdown
### Xác thực tích hợp (Ứng dụng web)

#### Kết nối API ↔ Frontend
- [ ] Shape phản hồi của tất cả API route và type generic của hook tương ứng khớp nhau
- [ ] Phản hồi có wrapping ({ items: [...] }) được unwrap trong hook
- [ ] Chuyển đổi snake_case ↔ camelCase được áp dụng nhất quán
- [ ] Frontend phân biệt được shape của phản hồi ngay lập tức (202) và kết quả cuối cùng
- [ ] Có hook tương ứng cho tất cả endpoint API và được gọi thực sự

#### Nhất quán routing
- [ ] Tất cả giá trị href/router.push trong code khớp với đường dẫn file page thực tế
- [ ] Xác thực đường dẫn có tính đến việc route group ((group)) bị xóa khỏi URL
- [ ] Kiểm tra dynamic segment ([id]) được điền với tham số đúng

#### Nhất quán state machine
- [ ] Tất cả chuyển tiếp trạng thái đã định nghĩa được thực thi trong code (không có chuyển tiếp chết)
- [ ] Tất cả cập nhật status trong code được định nghĩa trong bản đồ chuyển tiếp (không có chuyển tiếp trái phép)
- [ ] Chuyển tiếp từ trạng thái trung gian sang trạng thái cuối không bị thiếu
- [ ] Phân nhánh dựa trên trạng thái trong frontend (if status === "X") với X thực sự có thể đạt tới

#### Nhất quán luồng dữ liệu
- [ ] Ánh xạ giữa tên cột DB schema và tên trường phản hồi API nhất quán
- [ ] Tên trường trong định nghĩa type frontend và phản hồi API khớp nhau
- [ ] Xử lý null/undefined cho trường optional nhất quán ở cả hai phía
```

---

## 5. Template định nghĩa QA agent

Phần cốt lõi để bao gồm trong QA agent của build harness.

```markdown
---
name: qa-inspector
description: "Chuyên gia QA. Xác thực tuân thủ spec, nhất quán tích hợp và chất lượng thiết kế."
---

# QA Inspector

## Vai trò cốt lõi
Xác thực chất lượng triển khai so với spec và **nhất quán tích hợp giữa các module**.

## Ưu tiên xác thực

1. **Nhất quán tích hợp** (Cao nhất) — Không nhất quán ranh giới là nguyên nhân chính của lỗi runtime
2. **Tuân thủ spec chức năng** — API/state machine/data model
3. **Chất lượng thiết kế** — Màu sắc/typography/responsive
4. **Chất lượng code** — Code không dùng, quy ước đặt tên

## Phương pháp xác thực: "Đọc cả hai bên đồng thời"

Xác thực ranh giới phải **mở đồng thời cả hai bên code** để so sánh:

| Mục xác thực | Bên trái (Producer) | Bên phải (Consumer) |
|-------------|-------------------|-------------------|
| Shape phản hồi API | NextResponse.json() trong route.ts | fetchJson<T> trong hooks/ |
| Routing | Đường dẫn page file trong src/app/ | Giá trị href, router.push |
| Chuyển tiếp trạng thái | Bản đồ STATE_TRANSITIONS | Code .update({ status }) |
| DB → API → UI | Tên cột table | Trường phản hồi API → Định nghĩa type |

## Giao thức giao tiếp nhóm

- Yêu cầu sửa đổi cụ thể ngay khi phát hiện đến agent đó (file:dòng + phương pháp sửa)
- Vấn đề ranh giới thông báo cho **cả hai** agent ở hai phía
- Báo cáo cho leader: Báo cáo xác thực (phân biệt mục pass/fail/chưa xác thực)
```

---

## Trường hợp thực tế: Bug phát hiện trong SatangSlide

Tất cả nội dung hướng dẫn này được rút ra từ các bug thực tế sau:

| Bug | Ranh giới | Nguyên nhân |
|-----|---------|------------|
| `projects?.filter is not a function` | API→hook | API trả về `{projects:[]}`, hook kỳ vọng mảng |
| Tất cả link dashboard 404 | Đường dẫn file→href | Thiếu tiền tố `/dashboard/` |
| Ảnh theme không hiển thị | API→component | `thumbnailUrl` vs `thumbnail_url` |
| Chọn theme không lưu | API→hook | API select-theme tồn tại, không có hook |
| Trang tạo chờ mãi | Chuyển tiếp trạng thái→code | Thiếu code chuyển tiếp `template_approved` |
| `data.failedIndices` crash | Phản hồi ngay lập tức→frontend | Truy cập kết quả nền trong phản hồi ngay lập tức |
| 404 sau khi hoàn thành | Đường dẫn file→href | `/projects/` → `/dashboard/projects/` |
