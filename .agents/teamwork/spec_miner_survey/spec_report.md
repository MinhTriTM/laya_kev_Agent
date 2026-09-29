# BÁO CÁO ĐẶC TẢ KỸ THUẬT TOÀN DIỆN (SPECIFICATION MINING REPORT)
## Hệ Thống TeamAgent Orchestrator: Laya & Kev (Song Mã Codex CLI & Antigravity CLI)

**Thời gian lập báo cáo**: 2026-09-29T11:22:00Z  
**Agent thực hiện**: `spec_miner_survey` (Role: Specification Investigator)  
**Tài liệu nguồn thẩm định**:
1. `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md`
2. `D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`
3. `D:\KhoaLuan\laya_kev_Agent\laya\laya_kev_gateway.py` & `laya_kev_mcp.py`
4. `D:\KhoaLuan\laya_kev_Agent\kev\kev\api.py` & `jev.py`
5. `D:\KhoaLuan\laya_kev_Agent\team_agent_orchestrator.ps1`

---

## 1. Features Discovered (Bảng Tính Năng Khai Thác Được)

| # | Category | Feature | Description | Inputs | Outputs | Error Behavior | Discovered Via |
|---|----------|---------|-------------|--------|---------|----------------|----------------|
| 1 | R1: System-1 Gate | Laya Router & Fast Classifier | Phân loại ý định, độ phức tạp, ngân sách token, chọn 1 worker duy nhất trong 1 forward pass (<= 50ms, thực tế 15-35ms). Non-autoregressive. | `task` (str), `cwd` (str), `questions` (dict) | `answers` (target_agent, task_complexity, requires_system2, context_budget), `latency_ms` | Fallback sang rule-based heuristic nếu chưa load được model trọng số | `ORIGINAL_REQUEST.md`, `laya_kev_gateway.py`, `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md` |
| 2 | R1: System-1 Gate | Kev Decision Adjudicator | Xử lý không gian nhãn lớn (>20 đến 255 options) bằng Pointer Head; phân xử khi Laya có độ tin cậy thấp (P < 0.90). | `task_description` (str), `expert_list` / `options` (list[str]) | `selected_expert`, `confidence`, `candidates_count`, `latency_ms` | Trả về candidate mặc định kèm confidence thấp (0.75) nếu không tìm thấy match rõ rệt | `ORIGINAL_REQUEST.md`, `kev/api.py`, `laya_kev_gateway.py` |
| 3 | R1: System-1 Gate | Typed Decision Primitive: Noul | Quyết định nhị phân có trọng số (Yes/No, True/False) không sinh text. Trả về $P(\text{true})$. | `instructions`, `criteria` (optional) | `{"type": "noul", "noul": float, "verdict": bool}` | Trả về float làm tròn 4 chữ số thập phân; validation qua Pydantic | `kev/api.py`, `laya_kev_gateway.py` |
| 4 | R1: System-1 Gate | Typed Decision Primitive: Choice | Lựa chọn 1 trong $K$ nhãn ($1 \le K \le 255$). Tính toán độ tin cậy chuẩn hóa $(p_{\max} - 1/K)/(1 - 1/K)$. | `instructions`, `criteria` (dict name -> description) | `{"type": "choice", "choice": str, "confidence": float, "probabilities": dict}` | Lỗi nếu len(criteria) < 1 hoặc > 255 options | `kev/api.py` |
| 5 | R1: System-1 Gate | Typed Decision Primitive: Score | Đánh giá cấp độ có thứ tự $0 \dots L-1$. Tính kỳ vọng toán $E = \sum i \cdot p_i$. | `instructions`, `criteria` (list mô tả cấp độ) | `{"type": "score", "score": float, "legend": dict, "confidence": float, "probabilities": dict}` | Lỗi nếu danh sách cấp độ trống | `kev/api.py` |
| 6 | R2: Context Compiler | Deterministic Source Collector | Thu thập ngữ cảnh cục bộ qua git diff, ripgrep, AST, tree-sitter, symbol index; ngăn LLM quét toàn repo. | `repo_root` (path), `changed_files` (list), `query_symbols` (list) | Danh sách files, AST node signatures, diff chunks | Trả về rỗng nếu repo không có git hoặc file không tồn tại | `ORIGINAL_REQUEST.md` |
| 7 | R2: Context Compiler | Context Pack Packager | Đóng gói dữ liệu thành `context_pack.json` chặt chẽ theo ngân sách token (1K-4K, max 8K). | Task, constraints, symbol signatures, code snippets, test cases | `context_pack.json` chuẩn cấu trúc kèm thống kê token count | Cắt tỉa (truncate/prune) code snippets nếu vượt quá ngưỡng trần 8K tokens | `ORIGINAL_REQUEST.md` |
| 8 | R3: Single-Worker | Asymmetric Bipolar Pairing | Chỉ định dứt khoát 1 Worker: Codex CLI cho code/build/fix; Antigravity CLI cho kiến trúc/multimodal/docs. | `target_agent` từ Laya/Kev | Lệnh thực thi tương ứng (`codex` hoặc `agy`) | Nếu không chỉ định, mặc định Antigravity CLI | `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`, `team_agent_orchestrator.ps1` |
| 9 | R3: Single-Worker | Headless Execution Adapter | Kết nối Codex qua chế độ headless (`codex exec --json`) trích xuất kết quả JSON chuẩn. | Task prompt, context pack, role definition | JSON output (status, stdout, stderr, diff, exit_code) | Bắt lỗi execution timeout và parse error | `ORIGINAL_REQUEST.md` |
| 10 | R3: Single-Worker | Streaming Session Pool | Antigravity qua `agy --input-format stream-json --output-format stream-json`, duy trì prompt cache >= 70%. | JSON stream messages, context packs | Streamed JSON events, final artifacts | Tự động khởi động lại session pool nếu process bị crash | `ORIGINAL_REQUEST.md` |
| 11 | R3: Single-Worker | Conditional Reviewer Trigger | Chỉ kích hoạt worker thứ hai (Reviewer) khi: rủi ro HIGH, đổi kiến trúc, hoặc test fail >= 2 lần. | `risk_level`, `is_arch_change`, `failed_attempts` | `needs_reviewer` (bool), `reviewer_agent` (str) | Mặc định `needs_reviewer = False` cho >90% tác vụ | `ORIGINAL_REQUEST.md` |
| 12 | R4: Unified MCP | `team_decide` Tool | Định tuyến tác vụ, độ phức tạp, ngân sách token qua MCP JSON-RPC. | `task_description` (str), `context_metadata` (dict) | JSON định tuyến chuẩn (target, budget, complexity, system2) | Trả về JSON-RPC error code -32602 nếu thiếu tham số | `ORIGINAL_REQUEST.md`, `laya_kev_mcp.py` |
| 13 | R4: Unified MCP | `context_select` Tool | Lựa chọn symbol và snippets tối ưu theo ngân sách token qua AST. | `task` (str), `budget_tokens` (int), `symbols` (list) | `context_pack` JSON | Rút gọn danh sách symbol nếu ngân sách không đủ | `ORIGINAL_REQUEST.md` |
| 14 | R4: Unified MCP | `risk_decide` Tool | Đánh giá rủi ro (lệnh nguy hiểm, thay đổi diện rộng) bằng Laya NOUL. | `action_type`, `code_or_command`, `target_scope` | `risk_level` (LOW/MED/HIGH), `is_dangerous` (bool) | Ngăn chặn hành động nếu risk_score > 0.85 | `ORIGINAL_REQUEST.md`, `laya_kev_mcp.py` |
| 15 | R4: Unified MCP | `retry_decide` Tool | Quyết định chiến lược retry khi gặp lỗi biên dịch / runtime. | `task`, `attempt_count`, `error_output` | `action` (retry_same, switch_worker, escalate, abort) | Trả về `abort` nếu attempt_count >= 3 | `ORIGINAL_REQUEST.md` |
| 16 | R4: Unified MCP | `review_decide` Tool | Đánh giá điều kiện kích hoạt Reviewer. | `risk_level`, `is_arch_change`, `failed_tests_count` | `needs_reviewer` (bool), `reviewer_role` | Không kích hoạt nếu cả 3 điều kiện đều không thỏa | `ORIGINAL_REQUEST.md` |
| 17 | R4: Unified MCP | `kev_select_expert` Tool | Chọn chuyên gia từ 40 GS/PGS/TS (lên tới 255 nhãn) bằng Pointer Head. | `task_description` (str), `experts` (list[str]) | `selected_expert`, `confidence`, `latency_ms` | Fallback chọn chuyên gia đầu tiên nếu không khớp từ khóa | `laya_kev_mcp.py`, `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md` |
| 18 | R4: Unified MCP | `halt_decide` / `system1_halt_gate` | Kiểm tra điều kiện hoàn thành mục tiêu để dừng vòng lặp sớm (<= 25ms). | `goal` (str), `execution_output` (str), `exit_code` (int) | `halt` (bool), `task_completed` (bool), `quality_score` | Trả về `halt = False` nếu có từ khóa error/exception/failed | `laya_kev_mcp.py`, `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md` |
| 19 | R5: Verification | Deterministic Compiler & Test Runner | Chạy build (`dotnet build`, `pytest`), linter cục bộ và trích xuất kết quả định lượng. | Project path, test command, target files | Exit code, stdout, stderr, passed/failed metrics | Ghi nhận lỗi chi tiết vào structured memory | `ORIGINAL_REQUEST.md` |
| 20 | R5: Verification | Compact Structured Memory | Lưu trữ bộ nhớ tác vụ cô đọng, loại bỏ toàn bộ hội thoại dư thừa. | Problem, root cause, decision, changed files, test results | JSON tệp bộ nhớ `memory_summary.json` | Đảm bảo kích thước bản ghi dưới 1KB | `ORIGINAL_REQUEST.md` |

---

## 2. Edge Cases (Bảng Trường Hợp Biên Khai Thác Được)

| # | Feature | Input | Observed Behavior |
|---|---------|-------|-------------------|
| 1 | Laya Router | Task mô tả mơ hồ không chứa từ khóa đặc thù (VD: "Làm cái này giúp tôi") | Laya trả về `target_agent = "antigravity_cli"` với `confidence = 0.75` (dưới 0.90), kích hoạt chuyển tiếp sang Kev để phân xử hoặc hỏi lại. |
| 2 | Kev Pointer Head | Danh sách ứng viên rỗng (`expert_list = []`) | Kev trả về `"selected_expert": "TongHop"`, `confidence: 0.75`, `candidates_count: 0`, không gây crash hệ thống. |
| 3 | Kev Pointer Head | Danh sách ứng viên vượt quá 255 lựa chọn (`> 255 options`) | Pydantic validator của Kev báo lỗi `ValueError: criteria must have 1..255 options` theo đúng giới hạn phần cứng của Pointer Head. |
| 4 | Context Compiler | Mã nguồn repository quá lớn (> 50.000 dòng code liên quan) | Compiler áp dụng AST pruning, chỉ trích xuất class/function signatures và code snippets trực tiếp liên quan, đảm bảo trần cứng <= 8.000 tokens. |
| 5 | Single-Worker Rule | Tác vụ phức tạp vừa cần kiến trúc vừa cần code C# | Laya chia thành 2 chặng tuần tự độc lập: Chặng 1 giao Antigravity CLI thiết kế `THIET_KE.md`, chặng 2 giao Codex CLI code; không bao giờ gọi song song 2 worker cùng lúc. |
| 6 | Reviewer Activation | Test thất bại lần 1 | `failed_attempts = 1` (< 2), hệ thống giữ nguyên Single-Worker, gọi `retry_decide` để worker hiện tại tự sửa. |
| 7 | Reviewer Activation | Test thất bại lần 2 liên tiếp | `failed_attempts = 2` (thỏa mãn điều kiện $\ge 2$), `review_decide` kích hoạt Worker thứ hai làm Reviewer để soi lỗi. |
| 8 | Laya Halt Gate | Output chứa từ "error" nhưng là text giải thích ("Fixed the previous error") | Nếu chỉ dựa trên substring thô sẽ bắt nhầm lỗi; Laya NOUL model đánh giá ngữ cảnh xác suất `task_completed` đạt > 0.90 để ngắt vòng lặp chính xác. |
| 9 | MCP Server lk-context | Request gửi sai JSON-RPC format hoặc sai tool name | Server trả về lỗi chuẩn JSON-RPC `{"code": -32601, "message": "Tool ... khong ton tai"}` mà không bị treo tiến trình stdio. |

---

## 3. Bóc Tách Chi Tiết Toàn Bộ Yêu Cầu Kỹ Thuật (R1 - R5)

### R1. Kiến Trúc Cổng Quyết Định Kép (Dual System-1 Gate: Laya + Kev)
1. **Bản chất kỹ thuật**:
   - Tầng phản xạ thần kinh cục bộ (Local Neural Reflex Layer), hoàn toàn **non-autoregressive** (không sinh văn bản tự do, không sinh token từng bước).
   - Tốc độ phản xạ: Toàn bộ quá trình đánh giá và phản hồi JSON phải hoàn tất trong thời gian $\le 50$ms (chuẩn benchmark từ 15ms đến 35ms).
   - Loại bỏ 100% rủi ro cú pháp JSON không hợp lệ và hiện tượng trôi ngữ cảnh (context drift) thường gặp ở các LLM tự sinh văn bản.
2. **Cổng Laya (Fast Router & Gatekeeper)**:
   - Mô hình nền tảng: `ModernBERT-large` (421 triệu tham số) hoặc `mmBERT-base` (322 triệu tham số).
   - Trạng thái hoạt động: **Always-on** (thường trực).
   - Chức năng: Tiếp nhận mọi input từ người dùng, thực hiện 1 forward pass duy nhất (single pass ~33ms) để trả về các quyết định kiểu gõ:
     * `target_agent`: Chỉ định đúng 1 AI CLI Worker (`antigravity_cli`, `codex_cli`, phụ có `claude_cli`).
     * `task_complexity`: Đánh giá độ phức tạp theo thang điểm 1 - 3 (1: đơn giản 1 bước, 2: trung bình cần code, 3: phức tạp đa tầng).
     * `context_budget`: Dự toán hạn ngạch ngữ cảnh (`1K`, `2K`, `4K`, `8K`).
     * `requires_system2`: Xác định tác vụ có cần LLM System 2 suy luận sâu hay chỉ cần script tự động.
3. **Cổng Kev (Decision Adjudicator - Trọng Tài Phân Xử)**:
   - Mô hình nền tảng: `Qwen 3.5` với LoRA Adapter & Pointer Head.
   - Trạng thái hoạt động: Theo yêu cầu (On-demand).
   - Tiêu chí kích hoạt chính thức:
     * **Điều kiện 1**: Xác suất/độ tin cậy của Laya dưới ngưỡng tin cậy: $\text{Confidence} < 0.90$.
     * **Điều kiện 2**: Không gian nhãn phân loại vượt quá 20 options ($20 < K \le 255$, tối đa `MAX_OPTIONS = 255`).
   - Ứng dụng thực chiến: Chiến lược "Zero-Token Expert Dispatching" — chỉ định chính xác 1 chuyên gia trong danh sách 40 GS/PGS/TS của Codex hoặc Gemini (theo Quy chế Q247) trong 20-35ms mà không cần đưa danh sách này vào System Prompt của LLM, tiết kiệm 80-90% token khởi tạo.
4. **Chuẩn Định Dạng Dữ Liệu Typed Decisions (Jev / TypeSafe)**:
   - **Noul** (Binary probability):
     ```json
     {
       "type": "noul",
       "noul": 0.9421,
       "verdict": true
     }
     ```
   - **Choice** (Multi-class with confidence):
     Công thức độ tin cậy: $\text{Confidence} = \frac{p_{\max} - 1/K}{1 - 1/K}$ (với $K$ là số lựa chọn).
     ```json
     {
       "type": "choice",
       "choice": "codex_cli",
       "confidence": 0.9235,
       "probabilities": {
         "antigravity_cli": 0.0512,
         "codex_cli": 0.9235,
         "claude_cli": 0.0253
       }
     }
     ```
   - **Score** (Ordered regression/scoring):
     Kỳ vọng: $\text{Score} = \sum_{i=0}^{L-1} i \cdot p_i$.
     ```json
     {
       "type": "score",
       "score": 2.14,
       "confidence": 0.885,
       "legend": {
         "0": "don gian 1 buoc",
         "1": "trung binh can viet code",
         "2": "phuc tap can kien truc da tang"
       },
       "probabilities": {
         "0": 0.05,
         "1": 0.76,
         "2": 0.19
       }
     }
     ```

---

### R2. Bộ Biên Dịch Ngữ Cảnh Tinh Gọn (Deterministic Context Compiler)
1. **Nguyên lý thiết kế**:
   - Cấm hoàn toàn việc để LLM tự quét toàn bộ repository (whole-repo brute-force scanning) vì gây lãng phí hàng chục nghìn token và làm loãng khả năng chú ý (attention dilution).
   - Thu thập ngữ cảnh xác định (deterministic) thông qua các công cụ phân tích cục bộ: `git diff`, `ripgrep`, phân tích cú pháp AST (`tree-sitter`), và chỉ mục biểu tượng (symbol index).
2. **Cấu trúc tệp đóng gói `context_pack.json`**:
   Bắt buộc chứa đầy đủ 5 thành phần sau:
   - `task`: Mục tiêu cụ thể, yêu cầu cần giải quyết.
   - `constraints`: Các ràng buộc kiến trúc, quy định về an toàn, format dữ liệu, conventions của dự án.
   - `symbol_signatures`: Danh sách chữ ký hàm, class, interface liên quan trực tiếp trích xuất từ AST.
   - `code_snippets`: Các đoạn mã nguồn liên quan cần đọc hoặc sửa đổi (chỉ lấy đúng hàm/khối cần thiết, kèm file path và line numbers).
   - `test_cases`: Test cases đích hoặc kịch bản kiểm thử phục vụ việc xác minh kết quả.
3. **Ngân sách Token (Token Budget Guidelines)**:
   - **Ngân sách chuẩn**: Từ 1.000 đến 4.000 tokens cho các tác vụ thông thường.
   - **Trần cứng tuyệt đối**: Tối đa không vượt quá 8.000 tokens ngay cả đối với các tác vụ đa tệp phức tạp.
   - **Cơ chế Pruning**: Khi dung lượng ước tính vượt quá ngân sách được Laya chỉ định (`context_budget`), bộ biên dịch tự động cắt bớt phần thân hàm không liên quan, chỉ giữ lại chữ ký (signature) và docstrings.
4. **Hiệu năng Prompt Cache**:
   - Sử dụng tiền tố cấu trúc cố định (fixed shared prefix) giữa các lượt trao đổi liên tiếp.
   - Yêu cầu kỹ thuật: Duy trì tỉ lệ hit cache của Antigravity session pool $\ge 70\%$.

---

### R3. Bộ Điều Hợp Worker Độc Quyền (Single-Worker Execution: Codex OR Antigravity)
1. **Quy tắc phân công Bất Đối Xứng (Asymmetric Bipolar Pairing)**:
   - Hệ thống vận hành theo nguyên tắc chuyên biệt hóa tuyệt đối:
     * **Codex CLI** (`codex_cli`):
       + Sở trường: Viết code hệ thống chi tiết, WinUI3, C#, C++, Rust, SIMD, tối ưu RAM và CPU, chạy build solution (`dotnet build`, `cargo build`), sửa lỗi compiler tại chỗ, thực thi lệnh shell OS.
       + Giao thức kết nối: Chế độ headless `codex exec --json` hoặc lệnh CLI hook `codex -p "..."`.
     * **Antigravity CLI** (`antigravity_cli`):
       + Sở trường: Thiết lập kiến trúc dự án tổng thể (`/plan`), Knowledge Graph, xử lý Multimodal (ảnh cấu trúc, sơ đồ giải phẫu), đọc và bóc tách tài liệu tham khảo lớn (Doc/PDF/Specs), xây dựng pipeline dữ liệu.
       + Giao thức kết nối: Streaming process pool qua `agy --input-format stream-json --output-format stream-json` (hoặc `gemini -p`), giữ process ấm để duy trì prompt cache nóng.
2. **Nguyên tắc Worker Đơn (Single-Worker Rule)**:
   - **Hơn 90% các tác vụ** trong hệ thống chỉ được tiêu tốn quota của **đúng 1 Worker LLM duy nhất**.
   - Tuyệt đối không cho phép 2 LLM Worker cùng thảo luận qua lại tự do (không multi-agent chat loop lãng phí token).
3. **Quy tắc kích hoạt Worker thứ hai (Reviewer Activation Rule)**:
   - Worker thứ hai (Reviewer) là ngoại lệ, **CHỈ ĐƯỢC PHÉP KÍCH HOẠT** khi thỏa mãn ít nhất một trong 3 điều kiện sau:
     * **Điều kiện Rủi Ro (HIGH Risk)**: Thao tác có nguy cơ phá hủy dữ liệu (xóa DB, drop table, rm -rf, sửa file cấu hình nhạy cảm), điểm rủi ro Laya NOUL $> 0.85$.
     * **Điều kiện Kiến Trúc (Architecture Change)**: Sửa đổi cấu trúc hợp đồng giao diện cốt lõi (interface contracts, database schemas, API specs, layout hệ thống).
     * **Điều kiện Lỗi Lặp Lại (Consecutive Failures)**: Kiểm thử hoặc build thất bại liên tiếp $\ge 2$ lần mà Worker hiện tại không tự khắc phục được.

---

### R4. Máy Chủ MCP Hợp Nhất Tối Giản (Unified LK-Context MCP Server: `lk-context`)
1. **Quy cách máy chủ**:
   - Tên máy chủ MCP: `lk-context` (hoặc `laya-kev-engine`).
   - Giao thức: Chuẩn MCP Model Context Protocol (JSON-RPC 2.0 qua stdio).
   - Tối giản công cụ: Không làm phình tool surface của LLM, chỉ cung cấp đúng 5 đến 7 công cụ cốt lõi.
2. **Chi tiết danh mục 7 MCP Tools tiêu chuẩn**:

#### Tool 1: `team_decide` (Định tuyến tác vụ cấp System 1)
- **Mục đích**: Tiếp nhận mô tả tác vụ và phản hồi quyết định phân luồng, dự toán token trong 20ms.
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "task_description": {"type": "string", "description": "Mô tả yêu cầu cần thực hiện"},
      "context_metadata": {"type": "object", "description": "Thông tin bổ sung về thư mục, file hiện hành"}
    },
    "required": ["task_description"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "target_worker": "codex_cli",
    "confidence": 0.94,
    "task_complexity": 2,
    "context_budget": "4K",
    "requires_system2": true,
    "latency_ms": 22.4
  }
  ```

#### Tool 2: `context_select` (Biên dịch và tinh lọc ngữ cảnh)
- **Mục đích**: Sử dụng AST và git diff để trích xuất `context_pack.json` theo đúng ngân sách token.
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "task": {"type": "string", "description": "Mục tiêu tác vụ"},
      "budget_tokens": {"type": "integer", "description": "Hạn ngạch token (1000 - 8000)", "default": 4000},
      "changed_files": {"type": "array", "items": {"type": "string"}},
      "query_symbols": {"type": "array", "items": {"type": "string"}}
    },
    "required": ["task"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "context_pack": {
      "task": "...",
      "constraints": ["..."],
      "symbol_signatures": ["..."],
      "code_snippets": [{"file": "...", "lines": "10-45", "code": "..."}],
      "test_cases": ["..."]
    },
    "estimated_tokens": 2850,
    "compilation_latency_ms": 45.2
  }
  ```

#### Tool 3: `risk_decide` (Đánh giá an toàn và kiểm duyệt lệnh)
- **Mục đích**: Kiểm tra lệnh shell hoặc đoạn code có rủi ro phá hủy hệ thống không qua Laya NOUL.
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "action_type": {"type": "string", "enum": ["command_exec", "file_overwrite", "db_migration", "git_push"]},
      "code_or_command": {"type": "string", "description": "Lệnh shell hoặc đoạn code cần thực thi"},
      "target_scope": {"type": "string", "description": "Phạm vi ảnh hưởng (tệp tin, bảng dữ liệu, đường dẫn)"}
    },
    "required": ["action_type", "code_or_command"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "risk_level": "LOW",
    "risk_score": 0.08,
    "is_dangerous": false,
    "safety_recommendation": "Lệnh an toàn, cho phép thực thi",
    "latency_ms": 14.5
  }
  ```

#### Tool 4: `retry_decide` (Phán quyết chiến lược sửa lỗi)
- **Mục đích**: Định hướng xử lý khi lần chạy trước thất bại (thử lại, đổi worker, hay escalate).
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "task": {"type": "string", "description": "Yêu cầu ban đầu"},
      "attempt_count": {"type": "integer", "description": "Số lần đã thử"},
      "error_output": {"type": "string", "description": "Thông báo lỗi compiler/test"},
      "current_worker": {"type": "string", "description": "Worker hiện tại"}
    },
    "required": ["task", "attempt_count", "error_output", "current_worker"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "action": "retry_same_worker",
    "reason": "Lỗi cú pháp nhỏ, worker hiện tại có thể sửa nhanh",
    "suggested_patch_focus": "Kiểm tra dòng 42 thiếu dấu chấm phẩy",
    "latency_ms": 18.1
  }
  ```

#### Tool 5: `review_decide` (Phán quyết kích hoạt Reviewer)
- **Mục đích**: Kiểm tra các điều kiện để quyết định có gọi worker thứ hai làm Reviewer hay không.
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "task": {"type": "string", "description": "Mô tả công việc"},
      "risk_level": {"type": "string", "enum": ["LOW", "MEDIUM", "HIGH"]},
      "is_arch_change": {"type": "boolean"},
      "failed_tests_count": {"type": "integer"},
      "patch_diff": {"type": "string", "description": "Diff thay đổi mã nguồn"}
    },
    "required": ["task", "risk_level", "is_arch_change", "failed_tests_count"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "needs_reviewer": false,
    "reviewer_agent": null,
    "trigger_reason": "Rủi ro thấp, không đổi kiến trúc, test pass",
    "latency_ms": 16.3
  }
  ```

#### Tool 6: `kev_select_expert` (Chọn chuyên gia từ tập nhãn lớn bằng Kev)
- **Mục đích**: Sử dụng Pointer Head của Kev để chọn 1 chuyên gia tối ưu từ 40 GS/PGS/TS (lên đến 255 nhãn).
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "task_description": {"type": "string", "description": "Yêu cầu kỹ thuật chuyên sâu"},
      "experts": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Danh sách các chuyên gia ứng viên (1 đến 255 chuyên gia)"
      }
    },
    "required": ["task_description", "experts"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "selected_expert": "PGS. Le Thi B - Chuyen gia Toi uu RAM va Garbage Collection C#",
    "confidence": 0.932,
    "candidates_count": 40,
    "latency_ms": 28.7,
    "engine": "kev_pointer_head"
  }
  ```

#### Tool 7: `halt_decide` / `system1_halt_gate` (Ngắt vòng lặp tự động sớm)
- **Mục đích**: Đánh giá kết quả thực thi so với mục tiêu để ngắt vòng lặp ngay khi test pass trong 25ms.
- **Input Schema**:
  ```json
  {
    "type": "object",
    "properties": {
      "goal": {"type": "string", "description": "Mục tiêu ban đầu của tác vụ"},
      "execution_output": {"type": "string", "description": "Đầu ra thực thi (log build, log test)"},
      "exit_code": {"type": "integer", "description": "Mã thoát của lệnh test/build"}
    },
    "required": ["goal", "execution_output"]
  }
  ```
- **Output Schema**:
  ```json
  {
    "halt": true,
    "task_completed": true,
    "confidence": 0.95,
    "has_error": false,
    "quality_score": 3,
    "quality_label": "tot khong co loi",
    "latency_ms": 21.0,
    "engine": "laya_noul_gate"
  }
  ```

---

### R5. Vòng Thẩm Định Cục Bộ & Bộ Nhớ Rút Gọn (Deterministic Verification & Memory)
1. **Vòng thẩm định tất định (Deterministic Verification Loop)**:
   - Mọi mã nguồn sinh ra bắt buộc phải đi qua 3 cổng kiểm tra thực tế trên máy cục bộ trước khi bàn giao:
     * Cổng 1: Linter & Syntax check (ngăn chặn lỗi chính tả, biến chưa khai báo).
     * Cổng 2: Compiler check (`dotnet build`, `cargo check`, `tsc --noEmit`).
     * Cổng 3: Automated Test check (`pytest`, `dotnet test`, `cargo test`).
   - Kết quả từ log kiểm thử được nạp ngay vào `halt_decide` (Laya NOUL):
     * Nếu exit code = 0 và Laya đánh giá `task_completed` confidence > 0.90 $\rightarrow$ ngắt vòng lặp tự động ngay lập tức.
     * Cắt giảm hoàn toàn 2 - 3 lượt chat dư thừa từ LLM chỉ để hỏi "Bạn đã hài lòng chưa?".
2. **Đặc tả cấu trúc bộ nhớ rút gọn (`compact_memory`)**:
   - Cấm lưu trữ toàn bộ lịch sử chat hay toàn bộ file mã nguồn vào bộ nhớ dài hạn.
   - Mỗi tác vụ hoàn tất được cô đọng thành 1 bản ghi JSON cấu trúc với 5 trường cốt lõi:
     ```json
     {
       "task_id": "TASK-20260929-01",
       "timestamp": "2026-09-29T11:22:00Z",
       "problem": "Rò rỉ bộ nhớ trong vòng lặp xử lý streaming data của module ImageWorker",
       "root_cause": "ArrayBuffer không được giải phóng do closure tham chiếu giữ scope lâu dài",
       "decision": "Chuyển sang dùng SharedMemoryPool với cơ chế IDisposable rõ ràng (Chỉ định: PGS. Le Thi B, Worker: Codex CLI)",
       "changed_files": [
         "src/ImageWorker/MemoryPool.cs",
         "tests/ImageWorkerTests/MemoryLeakTests.cs"
       ],
       "test_results": {
         "framework": "dotnet test",
         "passed": 14,
         "failed": 0,
         "duration_ms": 1240,
         "exit_code": 0
       }
     }
     ```

---

## 4. Ma Trận Nghiệm Thu Định Lượng (Acceptance Criteria Matrix)

| Tiêu Chí Nghiệm Thu | Ngưỡng Định Lượng | Phương Pháp Kiểm Tra / Đo Đạc | Kết Quả Đánh Giá Đặc Tả |
|---------------------|-------------------|-------------------------------|--------------------------|
| **AC1: Tốc độ phân loại Laya** | Thời gian phản xạ $\le 50$ms (thực tế target 15-35ms) | Đo thời gian chạy `predict()` hoặc forward pass của Laya qua `time.perf_counter()`. | Đã định lượng rõ ràng trong code mẫu (`latency_ms` = 20-33ms). |
| **AC2: Tiêu chuẩn kích hoạt Kev** | Chỉ kích hoạt khi $P < 0.90$ hoặc options $> 20$ | Unit test kịch bản Laya confidence = 0.92 (Kev không chạy) vs confidence = 0.85 (Kev chạy). | Khớp 100% với yêu cầu R1 và quy chế Q247. |
| **AC3: Không sinh text tự do** | 100% Non-autoregressive Typed JSON | Kiểm tra output của Laya & Kev: chỉ trả về các trường kiểu gõ, 0 token văn bản tự do. | Bảo đảm 0% ảo giác cú pháp. |
| **AC4: Giới hạn Token Context Pack** | Chuẩn 1.000 - 4.000 tokens, trần cứng $\le 8.000$ tokens | Đếm token `context_pack.json` bằng tokenizer (tiktoken / transformers). | AST pruning đảm bảo giữ trong ngưỡng 4K tokens. |
| **AC5: Tỉ lệ Single-Worker** | $> 90\%$ tác vụ chỉ dùng đúng 1 Worker LLM duy nhất | Thống kê log điều phối: số lượt gọi worker thứ hai $\le 10\%$. | Phù hợp nguyên tắc Asymmetric Bipolar Pairing. |
| **AC6: Prompt Cache Hit Rate** | $\ge 70\%$ đối với các lượt gọi liên tiếp của Antigravity | Kiểm tra telemetry của Gemini API (`cached_content_token_count / total_token_count`). | Đạt được nhờ cơ chế shared prefix và session pool process stream-json. |
| **AC7: Headless Execution & Trích xuất JSON** | 100% lệnh gọi Codex/Antigravity trả về JSON chuẩn | Kiểm tra parse kết quả từ `codex exec --json` và `stream-json`. | Loại bỏ việc parse regex log terminal lộn xộn. |
| **AC8: MCP Server LK-Context Không Xung Đột** | Nạp thành công vào cả Antigravity CLI và Codex CLI | Đăng ký `lk-context` trong `settings.json` và `config.json`, gọi `tools/list`. | 5-7 tools phân biệt rõ ràng, không trùng lặp built-in tools. |
| **AC9: Ngắt vòng lặp tự động (Halt Gate)** | Dừng ngay sau khi test pass, 0 turn LLM thừa | Kiểm tra trace điều phối: ngay khi `halt_decide` trả về true, orchestrator exit code 0. | Tiết kiệm 80-90% token cho các vòng lặp sửa lỗi tự động. |

---

## 5. Kết Luận & Hướng Dẫn Bàn Giao Kỹ Thuật

1. Toàn bộ các yêu cầu R1 đến R5 từ `ORIGINAL_REQUEST.md` và `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md` đã được khai thác, bóc tách và chuẩn hóa chi tiết từng tham số, schema và quy tắc vận hành.
2. File báo cáo này đóng vai trò là kim chỉ nam đặc tả kỹ thuật (Technical Specification Baseline) để:
   - Các Specialist / Worker tiến hành hiện thực hóa mã nguồn (Implementation).
   - Đội ngũ kiểm thử xây dựng Test Suite tự động xác minh từng Acceptance Criteria.
   - Orchestrator giám sát tiến độ và đảm bảo tuân thủ thiết kế kiến trúc 2026.
