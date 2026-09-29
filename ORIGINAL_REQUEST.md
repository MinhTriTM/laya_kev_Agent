# Original User Request

## 2026-09-29T11:16:10Z

Xây dựng hệ thống **TeamAgent Orchestrator** toàn diện điều phối song mã **Codex CLI** và **Antigravity CLI**, tích hợp hai System-1 Decision Engines là **Laya** (Fast Router & Gatekeeper) và **Kev** (Decision Adjudicator khi Laya không chắc chắn hoặc nhãn phân loại lớn) kết hợp bộ **Context Compiler** (AST/tree-sitter/diff) nhằm cô đọng Context Pack chỉ từ 1K đến 4K tokens, giảm tối đa hạn ngạch quota LLM và tăng tốc độ code tự động.

Working directory: D:\KhoaLuan\laya_kev_Agent
Integrity mode: development

## Requirements

### R1. Kiến Trúc Cổng Quyết Định Kép (Dual System-1 Gate: Laya + Kev)
- Xây dựng tầng quyết định kiểu gõ (Typed Decisions) cục bộ không sinh văn bản tự do (non-autoregressive), đạt độ trễ <= 50ms.
- **Laya**: Là cổng tiếp nhận tức thì (always-on), phân loại ý định (intent), độ phức tạp (complexity), dự toán ngân sách ngữ cảnh (context budget: 1K/2K/4K/8K), và chỉ định 1 Worker duy nhất.
- **Kev**: Là trọng tài phân xử (Decision Adjudicator), chỉ kích hoạt khi xác suất phân loại của Laya dưới ngưỡng tin cậy (P < 0.90) hoặc khi không gian nhãn vượt quá 20 options.

### R2. Bộ Biên Dịch Ngữ Cảnh Tinh Gọn (Deterministic Context Compiler)
- Thu thập ngữ cảnh bằng công cụ mã nguồn cục bộ (git diff, ripgrep, AST, tree-sitter, symbol index) thay vì để LLM tự quét toàn bộ repository.
- Biên dịch dữ liệu thành `context_pack.json` chặt chẽ theo ngân sách (Target 1K - 4K tokens, tối đa 8K tokens) gồm: task, constraints, symbol signatures, code snippets liên quan, và test cases đích.

### R3. Bộ Điều Hợp Worker Độc Quyền (Single-Worker Execution: Codex OR Antigravity)
- Quy tắc điều phối bất đối xứng dứt khoát: mỗi tác vụ chỉ kích hoạt đúng 1 Worker lớn (`Codex CLI` cho sửa code/build/fix bug hoặc `Antigravity CLI` cho nghiên cứu/kiến trúc/multimodal).
- Chỉ kích hoạt worker thứ hai (Reviewer) khi: rủi ro ở mức HIGH, thay đổi kiến trúc hệ thống, hoặc test thất bại liên tiếp >= 2 lần.
- Kết nối Codex qua chế độ headless (`codex exec --json`) và Antigravity qua streaming process pool (`agy --input-format stream-json --output-format stream-json`) để tái sử dụng prompt cache nóng giữa các turn.

### R4. Máy Chủ MCP Hợp Nhất Tối Giản (Unified LK-Context MCP Server)
- Đóng gói toàn bộ khả năng System 1 thành 1 MCP Server duy nhất (`lk-context`) với tối đa 5-7 công cụ cốt lõi (`team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`) để tránh làm phình tool surface của LLM.

### R5. Vòng Thẩm Định Cục Bộ & Bộ Nhớ Rút Gọn (Deterministic Verification & Memory)
- Kiểm tra tính đúng đắn bằng trình biên dịch, linter và unit test trước khi bàn giao.
- Bộ nhớ task được cô đọng dưới dạng cấu trúc ngắn gọn (problem, root cause, decision, changed files, test results) thay vì lưu toàn bộ lịch sử hội thoại.

## Acceptance Criteria

### Tính Chuẩn Xác & Tốc Độ Của Tầng Quyết Định
- [ ] Laya phân loại task và chỉ định worker trong thời gian <= 50ms.
- [ ] Kev chỉ được kích hoạt khi Laya confidence < 0.90 hoặc khi task có gắn cờ rủi ro/nhãn lớn.
- [ ] Không có thành phần nào trong tầng System 1 sinh văn bản tự do ngoài định dạng JSON có cấu trúc.

### Giảm Tiêu Hao Ngữ Cảnh & Token
- [ ] Context pack cho các tác vụ thông thường không vượt quá 4.000 tokens.
- [ ] Hơn 90% các tác vụ chỉ tiêu tốn quota của đúng 1 Worker LLM duy nhất.
- [ ] Session pool của Antigravity duy trì tỉ lệ hit cache >= 70% đối với các lượt trao đổi liên tiếp.

### Tích Hợp & Vận Hành
- [ ] Khởi chạy lệnh headless của Codex và Antigravity tự động trích xuất kết quả thành JSON chuẩn.
- [ ] MCP Server `lk-context` nạp thành công trên cả Antigravity CLI và Codex CLI mà không gây xung đột tool schema.
- [ ] Script điều phối tự động ngắt vòng lặp ngay khi test pass mà không cần thêm lượt chat xác nhận dư thừa từ LLM.
