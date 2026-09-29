# HANDOFF REPORT — spec_miner_survey

**Thời gian**: 2026-09-29T11:23:00Z  
**Loại chuyển giao**: Hard Handoff (Hoàn tất toàn bộ nhiệm vụ điều tra và bóc tách đặc tả)  
**Agent thực hiện**: `spec_miner_survey` (Role: Specification Investigator)  
**Agent tiếp nhận**: `orchestrator_1` (Project Orchestrator)

---

## 1. Observation (Quan Sát Trực Tiếp)

1. **Tài liệu yêu cầu gốc `ORIGINAL_REQUEST.md`**:
   - Dòng 12-32: Xác định rõ 5 trụ cột kỹ thuật:
     * R1: Dual System-1 Gate (Laya + Kev), typed decisions, non-autoregressive, độ trễ $\le 50$ms. Laya luôn trực tuyến (always-on); Kev là trọng tài phân xử khi Laya có $P < 0.90$ hoặc không gian nhãn $> 20$ options.
     * R2: Deterministic Context Compiler thu thập qua git diff, ripgrep, AST, tree-sitter, symbol index; đóng gói `context_pack.json` theo ngân sách 1K - 4K tokens, trần cứng tối đa 8K tokens.
     * R3: Single-Worker Execution (Codex OR Antigravity); chỉ định 1 Worker cho $>90\%$ tác vụ; kích hoạt Reviewer khi rủi ro HIGH, thay đổi kiến trúc, hoặc test fail $\ge 2$ lần. Codex qua `codex exec --json`, Antigravity qua streaming process pool `stream-json`.
     * R4: Unified LK-Context MCP Server (`lk-context`) gói 5-7 core tools (`team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`).
     * R5: Deterministic Verification (compiler, linter, tests) & cấu trúc bộ nhớ rút gọn (problem, root cause, decision, changed files, test results).
   - Dòng 35-49: Liệt kê 9 tiêu chí nghiệm thu định lượng (Acceptance Criteria).

2. **Tài liệu kiến trúc `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`**:
   - Dòng 17-25: Xác định nền tảng mô hình của Laya (`ModernBERT-large` 421M / `mmBERT-base` 322M, 1 forward pass 33ms) và Kev (`Qwen 3.5` với LoRA Adapter & Pointer Head, xử lý high-cardinality nhãn từ 20 đến 255 options).
   - Dòng 30-48: Hướng dẫn cấu hình MCP Server cho Antigravity CLI và Codex CLI.
   - Dòng 62-96: Mô hình Song Mã 3 Tầng và 4 chiến lược cầm cương thực chiến (Zero-Token Expert Dispatching, Asymmetric Bipolar Pairing, Single-Pass Fast Gate, Early Termination Halt Gate).

3. **Mã nguồn tham chiếu hiện có**:
   - `D:\KhoaLuan\laya_kev_Agent\laya\laya_kev_gateway.py` (dòng 34-190): Đã định nghĩa lớp `System1DecisionGateway` với các phương thức `route_task_to_agent`, `select_expert_from_team`, và `evaluate_quality_and_halt`.
   - `D:\KhoaLuan\laya_kev_Agent\laya\laya_kev_mcp.py` (dòng 50-105): Đã định nghĩa bước đầu 4 tools MCP (`system1_route_agent`, `kev_select_expert`, `system1_guardrail`, `system1_halt_gate`).
   - `D:\KhoaLuan\laya_kev_Agent\kev\kev\api.py` (dòng 10-160): Định nghĩa các kiểu gõ chuẩn `Noul`, `Choice`, `Score` với validation Pydantic, hằng số `MAX_OPTIONS = 255`, công thức tính `choice_confidence = (p_max - 1/K)/(1 - 1/K)` và `score_confidence`.
   - `D:\KhoaLuan\laya_kev_Agent\team_agent_orchestrator.ps1` (dòng 25-105): Triển khai PowerShell orchestration kết nối gateway, phân luồng Worker, chọn chuyên gia và chạy Quality Gate.

---

## 2. Logic Chain (Chuỗi Lập Luận)

1. **Từ Quan sát 1 & 2 $\rightarrow$ Xác định Bản Chất Hệ Thống System 1**:
   - Hệ thống không sử dụng LLM System 2 cho các quyết định điều hướng, định tuyến hay đánh giá chất lượng vi mô nhằm loại bỏ hoàn toàn độ trễ (giảm từ 2-10s xuống $\le 50$ms) và triệt tiêu 100% rủi ro sinh JSON hỏng (hallucination).
   - Sự kết hợp giữa Laya (nhanh, 1 forward pass, nhãn $\le 20$) và Kev (Pointer Head, nhãn lớn đến 255) tạo thành thế liên hoàn phân xử tối ưu chi phí token (tiết kiệm 80-90% token).

2. **Từ Quan sát 1 & 3 $\rightarrow$ Chuẩn Hóa Bộ Công Cụ MCP `lk-context` (R4)**:
   - File `laya_kev_mcp.py` hiện tại mới có 4 tools. Theo yêu cầu R4 tại `ORIGINAL_REQUEST.md`, máy chủ MCP chuẩn `lk-context` cần mở rộng thành bộ 5 - 7 công cụ đồng nhất:
     * `team_decide` (hợp nhất từ `system1_route_agent`)
     * `context_select` (biên dịch AST và lọc snippet theo token budget)
     * `risk_decide` (kiểm tra an toàn / lệnh nguy hiểm)
     * `retry_decide` (chiến lược khắc phục lỗi)
     * `review_decide` (phán quyết gọi worker thứ 2)
     * `kev_select_expert` (chọn chuyên gia từ tập nhãn lớn bằng Kev Pointer Head)
     * `halt_decide` (ngắt vòng lặp tự động ngay khi pass test)

3. **Từ Quan sát 1 & 2 $\rightarrow$ Xác Định Ranh Giới Single-Worker & Reviewer (R3)**:
   - Phân công bất đối xứng dứt khoát: Codex cho coding/build/OS, Antigravity cho kiến trúc/multimodal/data.
   - $>90\%$ tác vụ không được kích hoạt quá 1 worker.
   - Reviewer chỉ được kích hoạt khi thỏa mãn 1 trong 3 điều kiện định lượng: (1) Rủi ro HIGH (Laya NOUL > 0.85 hoặc lệnh xóa phá hủy), (2) Đổi kiến trúc/schema cốt lõi, (3) Test fail $\ge 2$ lần.

4. **Từ Quan sát 1 $\rightarrow$ Ràng Buộc Ngân Sách Ngữ Cảnh (R2) & Nghiệm Thu (R5)**:
   - `context_pack.json` bắt buộc tuân thủ 5 trường: `task`, `constraints`, `symbol_signatures`, `code_snippets`, `test_cases`.
   - Dung lượng token: 1K - 4K tokens, trần cứng $\le 8$K tokens.
   - Bộ nhớ tác vụ R5 bắt buộc cô đọng thành 5 trường cấu trúc: `problem`, `root_cause`, `decision`, `changed_files`, `test_results`, loại bỏ toàn bộ chat log dư thừa.

---

## 3. Caveats (Các Điểm Lưu Ý & Giới Hạn Khảo Sát)

1. **Mô hình Trọng Số Deep Learning**:
   - `laya_kev_gateway.py` hiện đang tích hợp cơ chế fallback rule-based tốc độ cao trong trường hợp môi trường chưa tải sẵn checkpoint trọng số PyTorch/ONNX của ModernBERT và Qwen 3.5 LoRA. Để đạt độ chính xác tối đa trong production, cần đảm bảo ONNX runtime hoặc PyTorch đã được nạp trọng số tương ứng.
2. **Tiến trình Streaming Process Pool của Antigravity CLI**:
   - Chế độ streaming `stream-json` của Antigravity CLI phụ thuộc vào việc duy trì tiến trình chạy ngầm (warm background process) để tái sử dụng prompt cache nóng ($\ge 70\%$). Cần đảm bảo wrapper quản lý tiến trình không tự ngắt connection giữa các turn.
3. **Môi trường Hệ điều hành**:
   - Dự án vận hành trên Windows (pwsh), các đường dẫn sử dụng định dạng Windows path (`D:\KhoaLuan\laya_kev_Agent\...`).

---

## 4. Conclusion (Kết Luận Bàn Giao)

1. Toàn bộ đặc tả kỹ thuật của 5 nhóm yêu cầu (R1 đến R5), các định dạng JSON Typed Decisions, các schema của 7 MCP tools, các ngưỡng kích hoạt Kev/Worker/Reviewer và toàn bộ 9 Acceptance Criteria đã được bóc tách chi tiết và ghi nhận đầy đủ tại:
   `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey\spec_report.md`
2. Tài liệu đặc tả đã sẵn sàng, hoàn toàn rõ ràng và đầy đủ định lượng để Orchestrator (`orchestrator_1`) phân công cho các worker tiến hành hiện thực hóa mã nguồn (Code Implementation) và xây dựng bộ kiểm thử (Test Automation) mà không còn bất kỳ điểm mơ hồ nào.

---

## 5. Verification Method (Phương Pháp Thẩm Định Độc Lập)

Người nhận bàn giao (`orchestrator_1` hoặc Sentinel) có thể thẩm định tính chính xác của báo cáo đặc tả thông qua các bước sau:
1. **Kiểm tra sự hiện diện và tính toàn vẹn của tệp báo cáo**:
   - File: `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey\spec_report.md`
   - Đảm bảo chứa đủ các bảng: `Features Discovered` (20 tính năng), `Edge Cases` (9 ca biên), và chi tiết R1-R5, Acceptance Criteria Matrix.
2. **Đối chiếu các tham số kỹ thuật cốt lõi**:
   - Độ trễ System 1: $\le 50$ms (chuẩn 15-35ms).
   - Ngưỡng kích hoạt Kev: Laya confidence $< 0.90$ hoặc số nhãn $> 20$ (lên đến 255).
   - Ngân sách token: Chuẩn 1K-4K tokens, trần cứng tối đa 8K tokens.
   - Tỉ lệ Single-Worker: $> 90\%$ tác vụ.
   - Điều kiện kích hoạt Reviewer: HIGH risk, kiến trúc thay đổi, hoặc failed tests $\ge 2$ lần.
   - Số lượng MCP tools chuẩn: 5 đến 7 tools (`team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`, `kev_select_expert`, `halt_decide`).
   - Tỉ lệ hit cache: $\ge 70\%$ trên Antigravity session pool.
