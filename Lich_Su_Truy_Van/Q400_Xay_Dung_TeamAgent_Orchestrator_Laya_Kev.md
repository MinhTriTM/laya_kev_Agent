# Q400: Xây Dựng TeamAgent Orchestrator (Laya + Kev) Điều Phối Song Mã Codex CLI & Antigravity CLI

## 1. Câu hỏi người dùng
Xây dựng hệ thống **TeamAgent Orchestrator** toàn diện điều phối song mã **Codex CLI** và **Antigravity CLI**, tích hợp hai System-1 Decision Engines là **Laya** (Fast Router & Gatekeeper) và **Kev** (Decision Adjudicator khi Laya không chắc chắn hoặc nhãn phân loại lớn) kết hợp bộ **Context Compiler** (AST/tree-sitter/diff) nhằm cô đọng Context Pack chỉ từ 1K đến 4K tokens, giảm tối đa hạn ngạch quota LLM và tăng tốc độ code tự động.

Working directory: D:\KhoaLuan\laya_kev_Agent
Integrity mode: development

Requirements:
- R1. Kiến Trúc Cổng Quyết Định Kép (Dual System-1 Gate: Laya + Kev)
- R2. Bộ Biên Dịch Ngữ Cảnh Tinh Gọn (Deterministic Context Compiler)
- R3. Bộ Điều Hợp Worker Độc Quyền (Single-Worker Execution: Codex OR Antigravity)
- R4. Máy Chủ MCP Hợp Nhất Tối Giản (Unified LK-Context MCP Server)
- R5. Vòng Thẩm Định Cục Bộ & Bộ Nhớ Rút Gọn (Deterministic Verification & Memory)

Acceptance Criteria:
- Laya phân loại task và chỉ định worker trong thời gian <= 50ms.
- Kev chỉ được kích hoạt khi Laya confidence < 0.90 hoặc khi task có gắn cờ rủi ro/nhãn lớn.
- Context pack không vượt quá 4.000 tokens cho tác vụ thông thường.
- Hơn 90% tác vụ chỉ dùng 1 Worker LLM.
- Khởi chạy headless chuẩn JSON và nạp MCP lk-context thành công.

## 2. Suy nghĩ (Thinking) của Sentinel
- Đây là một bài toán kỹ thuật phần mềm phức tạp đa tầng (SWE / System Integration / Multi-agent Orchestration).
- Phân loại lộ trình theo Bảng Quyết Định Định Tuyến (Routing Decision Table):
  + Không phải Document Review (không có tài liệu cần phản biện/đánh giá).
  + Không phải Math / Proof.
  + Không phải SWE Light (đây là hệ thống lớn đa thành phần R1-R5, không phải 1 code change đơn lẻ cần làm nhanh/rẻ).
  + Lộ trình chính xác: **General** (`teamwork_preview_orchestrator`).
- Vai trò Sentinel:
  + Lưu giữ yêu cầu gốc vào `ORIGINAL_REQUEST.md`.
  + Khởi động Project Orchestrator (`teamwork_preview_orchestrator`).
  + Thiết lập cron theo dõi tiến độ (`*/8 * * * *`) và liveness check (`*/10 * * * *`).
  + Giữ context siêu nhẹ, không trực tiếp viết mã hay can thiệp kỹ thuật.

## 3. Câu trả lời / Hành động
- Đã ghi nhận yêu cầu vào `ORIGINAL_REQUEST.md`.
- Khởi tạo thư mục làm việc và kích hoạt `teamwork_preview_orchestrator`.
- Lập lịch giám sát tự động để theo dõi quá trình xây dựng hệ thống.
