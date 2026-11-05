# Hướng Dẫn Vận Hành Quy Trình Vòng Đời Sản Phẩm Toàn Diện

> [English Version](end-to-end-workflow-guide.md)

Tài liệu này hướng dẫn chi tiết cách xây dựng một tính năng thực tế đạt chuẩn Production: từ bước khám phá bài toán khách hàng, thiết kế kiến trúc, lập trình TDD, kiểm thử nghiệm thu tương tác, review bảo mật, đến xuất bản release notes và phòng ngừa sự cố vận hành bằng bộ kỹ năng công nghệ kết hợp hoàn chỉnh.

Ví dụ xuyên suốt là tính năng SaaS thực chiến: **Self-Serve Organization Data Export (Tự xuất dữ liệu tổ chức bất đồng bộ, migration không downtime và cô lập dữ liệu đa người thuê)**.

---

## Tổng quan Luồng Công Việc (Workflow Overview)

Toàn bộ vòng đời phát triển trải qua 9 giai đoạn có kỷ luật:

```
[1. /shape-problem] ──> [2. /grill-with-docs] ──> [3. /to-spec (+ ux-states, telemetry-design)]
                                                                  │
                                                                  ▼
[6. /qa-session] <── [5. /implement (+ /tdd)] <── [4. /to-tickets (+ expand-contract)]
        │
        ▼
[7. /code-review (+ security-review)] ──> [8. /pr & /release-notes] ──> [9. /post-mortem & /retro]
```

---

## Giai đoạn 1: Khám phá Bài toán Sản phẩm (`/shape-problem`)

### Bối cảnh
Khách hàng doanh nghiệp liên tục gửi ticket hỗ trợ phàn nàn vì không thể tự xuất dữ liệu lịch sử hóa đơn để kiểm toán cuối quý. Kỹ sư hỗ trợ phải chạy query SQL thủ công dưới database, gây rủi ro gửi nhầm dữ liệu bí mật sang khách hàng khác và lãng phí thời gian kỹ thuật.

### Prompt của Developer
```text
/shape-problem
Khách hàng doanh nghiệp đang gửi rất nhiều ticket phàn nàn vì không có cách nào tự export dữ liệu hóa đơn và giao dịch ra file CSV để làm kế toán cuối quý. Đội CSKH đang phải nhờ Backend chạy query tay. Hãy đóng khung bài toán này trước khi nói về code.
```

### Xử lý Dưới Nền (Under the Hood)
1. Kích hoạt các vòng phỏng vấn `grilling` tập trung hoàn toàn vào không gian bài toán: chân dung người dùng (persona), công việc cần làm (JTBD), giải pháp tạm thời hiện tại, chỉ số thành công đo lường được, và các mục tiêu kiên quyết loại trừ (non-goals).
2. Từ chối thảo luận về chi tiết kỹ thuật (Redis, hàng đợi background, S3 bucket).
3. Gọi `domain-modeling` để chuẩn hóa và cập nhật các thuật ngữ nghiệp vụ như `ExportJob` và `TenantScope` vào `GLOSSARY.md`.

### Đầu ra Thực tế (Problem Frame Artifact)
```markdown
# Problem Frame: Self-Serve Organization Data Export

## Problem Statement
Finance managers cannot independently extract their organization's historical transaction records for quarterly audits, forcing manual intervention from customer support and creating compliance delays.

## Target User & Context
- Primary Persona: Organization Billing Admins and Finance Leads.
- Context of Struggle: End-of-month and end-of-quarter accounting reconciliation.

## Job To Be Done (JTBD)
When closing monthly books, I want to download all organization invoice data in structured formats (CSV/JSON), so I can reconcile financial records without contacting support.

## Current Friction & Workarounds
Support engineers manually run ad-hoc SQL queries and email CSV files, introducing risk of sending data to the wrong recipient and wasting engineering hours.

## Measurable Success Indicators
- Reduction in support export requests to zero per week.
- 95% of export files generated and available for download within 60 seconds.

## Explicit Non-Goals
- Scheduled recurring automated exports (deferred to future milestone).
- Direct integration with third-party ERPs such as SAP or NetSuite.
```

---

## Giai đoạn 2: Thiết kế Kiến trúc Kỹ thuật & ADR (`/grill-with-docs`)

### Bối cảnh
Sau khi bài toán nghiệp vụ đã được đóng khung rõ ràng, nhóm tiến hành chốt phương án kiến trúc, lưu trữ và các phụ thuộc hạ tầng.

### Prompt của Developer
```text
/grill-with-docs
Dựa vào Problem Frame vừa chốt, hãy phỏng vấn tôi về kiến trúc kỹ thuật, cơ chế xử lý background worker, lưu trữ file tạm và chiến lược rate-limiting trên repo này.
```

### Xử lý Dưới Nền
1. Thực hiện các vòng phỏng vấn kỹ thuật có cấu trúc.
2. Thống nhất sử dụng hàng đợi bất đồng bộ thay vì giữ kết nối HTTP lâu dài (long-running connection).
3. Chọn giải pháp cấp URL tải về có chữ ký (presigned S3 URL) hết hạn sau 15 phút.
4. Ghi nhận quyết định kiến trúc thành Architectural Decision Record tại `docs/adr/0014-async-export-processing.md`.

---

## Giai đoạn 3: Soạn Đặc tả với UX & Telemetry (`/to-spec`)

### Bối cảnh
Tổng hợp toàn bộ ngữ cảnh trao đổi thành bản đặc tả kỹ thuật chi tiết trên issue tracker mà không cần phỏng vấn lại từ đầu.

### Prompt của Developer
```text
/to-spec
Hãy tổng hợp toàn bộ các quyết định đã thống nhất thành bản đặc tả kỹ thuật hoàn chỉnh và đẩy lên issue tracker với nhãn ready-for-agent.
```

### Xử lý Dưới Nền
1. Tự động nhận diện thành phần giao diện người dùng và gọi **`ux-states`** để dựng Ma trận Trạng thái Giao diện (UI State Matrix).
2. Tự động gọi **`telemetry-design`** để định nghĩa sự kiện phân tích hành vi, ngữ cảnh log có cấu trúc, chỉ số vận hành SLI và ngưỡng cảnh báo.

### Đầu ra Thực tế (Trích đoạn Spec)
```markdown
## UI State Matrix (via ux-states)

| State Dimension | Visual & Interaction Behavior | Technical Invariant |
| --- | --- | --- |
| **Empty (Zero Records)** | Modal states "No records match selected date range" with a reset filter button | `transactions.length === 0` |
| **Loading (Queued)** | Skeleton placeholder and progress indicator "Preparing export..." | `job.status === "QUEUED"` |
| **Optimistic Action** | Button instantly disables and transitions to "Export Requested" | Debounce handler prevents duplicate job dispatch |
| **Error Recovery** | Warning banner displays "Generation timed out. Retry?" with inline retry button | Catch block exposes single-click retry action |
| **Accessibility (a11y)** | Focus trapped in modal; status announced via `aria-live="polite"` | WCAG AA compliance |

## Telemetry & Observability (via telemetry-design)

- Behavioral Event: `export_requested` with `{ format: "csv" | "json", date_range_days: number }`
- Behavioral Event: `export_downloaded` with `{ export_id: string, file_size_bytes: number }`
- Structured Log: `INFO export.started` with `{ export_id, tenant_id, user_id }`
- Structured Log: `ERROR export.failed` with `{ export_id, tenant_id, error_code, stack_trace }`
- Metric: `exports_processing_seconds` (Histogram, P95 target < 60s)
- Alert: Trigger P2 alert when `rate(export_failed[5m]) / rate(export_started[5m]) > 0.05`
```

---

## Giai đoạn 4: Bẻ Nhỏ Tickets & Di Chuyển Dữ Liệu An Toàn (`/to-tickets`)

### Bối cảnh
Bẻ nhỏ spec thành các tracer-bullet tickets độc lập. Do có thay đổi cấu trúc bảng cơ sở dữ liệu và thêm background worker, cần đảm bảo không downtime.

### Prompt của Developer
```text
/to-tickets
Hãy chia spec trên thành các tracer-bullet tickets. Áp dụng expand-contract cho việc thay đổi database schema và bổ sung API.
```

### Xử lý Dưới Nền
1. Phân rã công việc thành các ticket tuần tự với quan hệ phụ thuộc rõ ràng (blocking edges).
2. Viện dẫn **`expand-contract`** để phân chia thay đổi schema qua 4 pha: Expand, Backfill, Switch, Contract.
3. Đánh giá rủi ro từng ticket theo tiêu chuẩn Cửa Hai Chiều (Two-Way Door) hoặc Cửa Một Chiều (One-Way Door).

### Đầu ra Thực tế (Danh sách Tickets)
```markdown
### Ticket Breakdown:
1. **Ticket #101 [Expand]**: Add `export_jobs` table and nullable `last_exported_at` column to `invoices`.
   - Risk: Two-way door (Safe rollback via down migration)
   - Blocked by: None
2. **Ticket #102 [Core Logic & Worker]**: Implement asynchronous export worker and S3 upload seam using TDD.
   - Blocked by: #101
3. **Ticket #103 [UI & State Matrix]**: Build Export Modal adhering to the UX State Matrix.
   - Blocked by: #102
4. **Ticket #104 [Telemetry & Alerts]**: Wire up behavioral events, logging context, and metric probes.
   - Blocked by: #102
5. **Ticket #105 [Contract]**: Add strict foreign key constraints and index on `export_jobs.tenant_id`.
   - Risk: One-way door (Requires database snapshot before execution)
   - Blocked by: #101, #102, #103
```

---

## Giai đoạn 5: Lập Trình Hướng Kiểm Thử (`/implement` cùng `/tdd`)

### Bối cảnh
Bắt tay thực hiện Ticket #102 (Xây dựng Worker và Seam lưu trữ file S3).

### Prompt của Developer
```text
/implement
Bắt đầu làm Ticket #102. Áp dụng TDD nghiêm ngặt, kiểm thử hành vi tại seam cao nhất và assert cả payload của telemetry.
```

### Xử lý Dưới Nền
1. Viết test tích hợp đỏ (`test/export-worker.test.ts`) bao quát xử lý job, stream dữ liệu và tạo presigned S3 URL.
2. Kiểm tra assert telemetry và structured logger được gọi với đúng metadata (bao gồm `tenant_id`).
3. Viết code tối thiểu để test chuyển xanh.
4. Tái cấu trúc (refactor) loại bỏ code smells nhưng vẫn giữ test xanh.

---

## Giai đoạn 6: Nghiệm Thu Tương Tác (`/qa-session`)

### Bối cảnh
Lập trình hoàn tất và các bài unit test tự động đều xanh. Server phát triển đang chạy tại `http://localhost:3000`.

### Prompt của Developer
```text
/qa-session
Tôi đã hoàn thành tính năng export trên nhánh feat/data-export. Hãy đối chiếu với spec ban đầu và hướng dẫn tôi thực hiện phiên QA nghiệm thu tương tác.
```

### Xử lý Dưới Nền
1. Đọc tiêu chí nghiệm thu từ spec và xây dựng Ma trận Kiểm thử 3 tầng: Luồng Chính (Happy Path), Biên Ngoại Lệ (Edge Cases), và Trạng Thái Lỗi (Failure Modes).
2. Hướng dẫn developer thao tác từng bước trực tiếp trên trình duyệt hoặc API.
3. **Kỷ luật cốt lõi:** Khi phát hiện lỗi bất thường, TUYỆT ĐỐI KHÔNG sửa code ngay trong phiên QA; thay vào đó, ghi lại báo cáo lỗi chuẩn hóa lên issue tracker.

### Đầu ra Thực tế (Defect Report Artifact)
```markdown
# Bug: Rapid Multi-Click Triggers Duplicate Export Jobs

## Environment
- Branch: `feat/data-export` (Commit: `8a9f3b1`)
- Browser: Chrome 130 / macOS

## Steps to Reproduce
1. Open Billing > Export Data dialog.
2. Rapidly click the "Start Export" button 5 times in under 1 second.

## Expected Behavior
Button disables immediately on first click, queuing exactly 1 job.

## Actual Behavior
Five duplicate jobs were inserted into `export_jobs`, causing redundant worker load.

## Severity
- **P2 - Major**: Redundant processing load on background queue.
```

---

## Giai đoạn 7: Review Code Đa Chiều (`/code-review`)

### Bối cảnh
Kiểm tra toàn bộ diff trên nhánh trước khi tạo Pull Request chính thức.

### Prompt của Developer
```text
/code-review main
Review toàn bộ diff giữa main và HEAD.
```

### Xử lý Dưới Nền
Khởi chạy đồng thời 3 sub-agents độc lập trong fresh context windows:
1. **Standards Sub-agent**: Rà soát coding standards của dự án và bảng code smells chuẩn.
2. **Spec Sub-agent**: So sánh diff với các yêu cầu nghiệp vụ trong spec gốc.
3. **Security Sub-agent (gọi `security-review`)**: Kiểm tra 5 trụ cột an ninh thông tin (AuthZ, PII, Injection, Network, Dependencies).

### Đầu ra Thực tế (Trích đoạn Báo cáo Bảo mật)
```markdown
## Security & Privacy Findings (via security-review)

### [CRITICAL] Missing Tenant Scope in Export Download Endpoint
- **Location**: `src/api/routes/exports.ts:64-72`
- **Vulnerability**: Insecure Direct Object Reference (AuthZ)
- **Attack Scenario**: An authenticated user in Tenant A can pass an `exportId` belonging to Tenant B and receive a valid presigned S3 URL, exposing competitor invoices.
- **Remediation**:
  ```ts
  const job = await db.exportJob.findFirst({
    where: { id: params.exportId, tenantId: session.tenantId }
  });
  if (!job) throw new NotFoundError();
  ```
```

---

## Giai đoạn 8: Pull Request & Thông Báo Bản Phát Hành (`/pr` & `/release-notes`)

### 1. Tạo Pull Request
Agent tự động phân loại rủi ro triển khai theo `expand-contract`:
```markdown
## Summary
Enables self-serve CSV and JSON data export for organization billing admins with asynchronous worker processing.

## Merge Danger & Migration Safety
- **Phase**: Expand (Phase 1)
- **Door Classification**: Two-Way Door
- **Rollback Strategy**: Can be reverted immediately via feature flag `ff_self_serve_export = false` without data loss.
```

### 2. Soạn Thảo Release Notes Cho Khách Hàng
Sau khi merge PR vào nhánh chính main:

### Prompt của Developer
```text
/release-notes
Nhánh feat/data-export đã được merge vào main. Hãy tổng hợp các PR và commit thành bản release notes hướng tới người dùng cuối.
```

### Đầu ra Thực tế (Customer Release Notes)
```markdown
# Release Notes: Version 2.8.0

## Highlights
Organizations can now independently export their complete transaction and invoice history directly from their billing dashboard, eliminating wait times for accounting audits.

## New Features
- **Self-Serve Data Export**: Billing administrators can download CSV and JSON summaries of all historical transactions with one click from Settings > Billing > Export.

## Improvements
- Background generation ensures large data requests process reliably without slowing down your active dashboard session.

## Bug Fixes
- Resolved an issue where filtered transaction tables did not persist date selections across page refreshes.
```

---

## Giai đoạn 9: Xử Lý Sự Cố Vận Hành & Học Hỏi (`/post-mortem` & `/retro`)

### Bối cảnh
Trên môi trường Production, một doanh nghiệp lớn chứa 500.000 dòng dữ liệu tiến hành export khiến worker bị lỗi tràn bộ nhớ (Out of Memory - OOM). Sự cố đã được xử lý khẩn bằng việc chuyển đổi sang database streaming cursor.

### Prompt của Developer
```text
/post-mortem
Sự cố OOM worker export đã được khắc phục. Hãy cùng tôi xây dựng timeline, chạy 5 Whys, tìm ra Detection Gap và ghi lại ADR phòng ngừa.
```

### Xử lý Dưới Nền
1. Tính toán Time to Detect (TTD = 18 phút) và Time to Mitigate (TTM = 25 phút).
2. Phân tích 5 Whys tìm ra nguyên nhân cấu trúc (nạp toàn bộ 500.000 dòng vào bộ nhớ RAM thay vì streaming theo chunks).
3. Ghi nhận Architectural Decision Record `docs/adr/0015-stream-large-database-exports.md`.
4. Mở các ticket phòng ngừa trên issue tracker.
5. Hướng dẫn developer chạy `/retro` để cập nhật `CODING_STANDARDS.md` và bổ sung rule kiểm tra tự động.

---

## Tóm Tắt Giá Trị Đạt Được

1. **Không Lãng Phí Nỗ Lực:** `shape-problem` chặn đứng việc lập trình sai tính năng ngay từ đầu.
2. **Giao Diện Nhất Quán:** `ux-states` bao quát toàn bộ trạng thái mạng, tải dữ liệu, lỗi và khả năng tiếp cận.
3. **Triển Khai An Toàn Tuyệt Đối:** `expand-contract` loại trừ nguy cơ downtime cơ sở dữ liệu và đứt gãy API.
4. **Cô Lập Dữ Liệu Đa Người Thuê:** `security-review` phát hiện kịp thời các lỗ hổng IDOR trước khi merge.
5. **Cải Tiến Hệ Thống Liên Tục:** `post-mortem` và `retro` biến sự cố vận hành thành các chốt chặn tự động cho tương lai.
