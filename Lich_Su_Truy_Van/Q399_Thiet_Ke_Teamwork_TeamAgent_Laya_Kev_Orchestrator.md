# Q399: Phân Tích & Khởi Tạo Đề Cương Dự Án Teamwork TeamAgent (Laya, Kev, Codex CLI, Antigravity CLI)

- **Ngày thực hiện**: 29/09/2026
- **Người yêu cầu**: Tổng Giám Đốc (User)
- **Agent thực hiện**: Giám Đốc B (Gemini 3.1 Pro / Antigravity CLI)

---

## 1. Yêu Cầu Của Tổng Giám Đốc

> `/teamwork-preview` phân tích:
> cắm laya và kev vào codex cli và antygravity cli. teamagent cầm thế nào để code bằng antygravity cli và codex cli có áp dụng laya và kev để giảm có thể mạnh nhất.
> Kèm theo bản phân tích chi tiết điều chỉnh vai trò của Kev (là Decision Model / Adjudicator phân xử khi Laya không chắc, không sinh text) và nguyên tắc tối thượng "Codex OR Antigravity", Context Pack 1K-4K tokens, và Unified MCP.

---

## 2. Quá Trình Suy Nghĩ & Phân Tích Kỹ Thuật (Thinking)

### 2.1 Tiếp thu và khẳng định điều chỉnh kiến trúc cốt lõi
1. **Laya và Kev đều là System 1 Decision Engines**:
   - Đây là sự điều chỉnh mang tính cách mạng: Không biến Kev thành một LLM reasoning sinh text dài dòng. Cả Laya và Kev đều hoạt động dựa trên cơ chế phản xạ thần kinh (typed decisions: choice, score, noul).
   - **Laya**: First responder cực nhanh (<35ms).
   - **Kev**: Trọng tài (Adjudicator) phân xử các trường hợp Laya có độ tin cậy thấp (P < 0.90) hoặc không gian nhãn quá lớn (>20 nhãn).
2. **Quy tắc "Codex OR Antigravity" (Một Worker duy nhất)**:
   - Sai lầm đốt token lớn nhất là cho cả 2 LLM cùng đọc repo và cùng chạy.
   - Worker 1 thực thi. Worker 2 chỉ đóng vai trò reviewer trong các trường hợp rủi ro cao (security, architecture, test fail $\ge$ 2 lần).
3. **Context Pack (1K - 4K tokens) thay thế việc quét toàn bộ repo**:
   - Dùng AST, ripgrep, tree-sitter, symbol graph để chỉ trích xuất đúng các chữ ký hàm, method liên quan và test cases.
4. **Hợp nhất MCP thành `lk-context`**:
   - Chỉ giữ 5-7 tools tối giản, không để tool definitions chiếm dụng context của LLM.
5. **Session nóng và Headless Execution**:
   - Antigravity CLI chạy streaming session để tận dụng prompt cache.
   - Codex CLI chạy `codex exec --json`.

---

## 3. Các Hành Động Triển Khai

1. Đã khởi tạo bản thảo đề cương dự án Teamwork: [`prompt_draft.md`](file:///C:/Users/user/.gemini/antigravity-cli/brain/3f1cc71a-87d1-4adc-9d59-c455c471a49f/prompt_draft.md).
2. Thiết kế 5 khối yêu cầu (R1 - R5) và bộ tiêu chí nghiệm thu khách quan (Acceptance Criteria).
3. Trình bày phân tích chi tiết và chuẩn bị câu hỏi làm rõ các tùy chọn quy mô với Tổng Giám Đốc.
