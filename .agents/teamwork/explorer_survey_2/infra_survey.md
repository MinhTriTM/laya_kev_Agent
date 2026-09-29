# BÁO CÁO KHẢO SÁT HẠ TẦNG & ĐỀ XUẤT KIẾN TRÚC TEAM AGENT ORCHESTRATOR
**Người thực hiện**: explorer_survey_2 (Role: Orchestration and Tooling Surveyor)  
**Thời gian khảo sát**: 2026-09-29  
**Mã dự án**: `laya_kev_Agent` — Thư mục làm việc: `D:\KhoaLuan\laya_kev_Agent`  
**Chỉ thị gốc**: `.agents/teamwork/ORIGINAL_REQUEST.md`  
**Tài liệu kiến trúc cơ sở**: `KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`  

---

## 1. TỔNG QUAN HIỆN TRẠNG HẠ TẦNG HỆ THỐNG CỤC BỘ

Qua quá trình quét trực tiếp môi trường Windows và thực thi các lệnh thăm dò hệ thống, hiện trạng hạ tầng ghi nhận như sau:

| Thành phần / Công cụ | Phiên bản / Vị trí thực tế | Đánh giá tính sẵn sàng cho R1 - R5 |
|---|---|---|
| **Hệ điều hành** | Windows 11 (Alienware x14 R2, GPU RTX 4060 8GB) | Tốt, sẵn sàng cho CUDA & local script |
| **Python** | `Python 3.12.0` | Sẵn sàng cho toàn bộ stack AI / MCP |
| **PyTorch & CUDA** | `torch 2.6.0+cu124`, `torchaudio`, `torchvision` | Có CUDA 12.4, chạy được mô hình Laya/Kev |
| **Transformers & Tokenizers** | `transformers 5.16.1`, `tokenizers 0.23.1`, `tiktoken 0.12.0` | Hoàn toàn sẵn sàng cho Context Compiler và Token Budget |
| **MCP SDKs** | `fastmcp 4.0.5`, `mcp 2.2.0`, `mcp-types 2.2.0` | Đã cài đặt, sẵn sàng cho Unified MCP Server `lk-context` |
| **Kiểm thử & Linter** | `pytest 9.0.2`, `pytest-asyncio 1.4.0`, Python `ast` (built-in) | Sẵn sàng cho Local Verification Loop (R5) |
| **Ripgrep (rg)** | `C:\Users\user\AppData\Local\Microsoft\WinGet\Links\rg.exe` | Sẵn sàng cho Symbol Search và Code Retrieval cục bộ |
| **Git CLI** | `git version 2.55.0.windows.4` (`C:\Program Files\Git\cmd\git.exe`) | Sẵn sàng trích xuất git diff và changed files |
| **Node.js** | `v22.23.2` | Sẵn sàng cho CLI hỗ trợ nếu cần |
| **Codex CLI** | `C:\Users\user\AppData\Local\hermes\node\codex.ps1` | Đã kiểm tra: hỗ trợ `codex exec --json` headless mode |
| **Antigravity CLI** | `C:\Users\user\AppData\Local\agy\bin\agy.exe` | Đã kiểm tra: hỗ trợ `--input-format stream-json --output-format stream-json` |
| **Gemini CLI** | `C:\Users\user\AppData\Roaming\npm\gemini.ps1` | Sẵn sàng làm phương án fallback cho Antigravity |

---

## 2. PHÂN TÍCH HIỆN TRẠNG KỊCH BẢN ĐIỀU PHỐI `team_agent_orchestrator.ps1`

### 2.1 Cấu trúc hiện tại
Tệp `team_agent_orchestrator.ps1` (105 dòng) thực hiện quy trình 3 bước:
1. **Bước 1: System 1 Gateway**: Chạy đoạn mã Python nhúng inline:
   ```powershell
   sys.path.append(r'$PSScriptRoot')
   from laya_kev_gateway import System1DecisionGateway
   gw = System1DecisionGateway(preload=False)
   res = gw.route_task_to_agent("""$TaskPrompt""")
   ```
2. **Bước 2: Phân phối Worker**:
   - Nếu là `codex_cli`: Gọi tiếp inline Python để chạy `gw.select_expert_from_team(...)` với danh sách cố định 5 chuyên gia mẫu (hardcoded), sau đó gọi `codex.ps1 -p "Vai trò: ... Thực thi nhiệm vụ: ..."`.
   - Nếu là `antigravity_cli`: Gọi `agy.exe -p "$TaskPrompt"` hoặc `gemini -p "$TaskPrompt"`.
3. **Bước 3: Quality Gate**: In ra dòng chữ `Write-Host "Kết quả thẩm định: PASS..."`.

### 2.2 Các khiếm khuyết & Lỗ hổng kỹ thuật nghiêm trọng
1. **Lỗi đường dẫn (Path Bug)**:
   - Trong script ghi `sys.path.append(r'$PSScriptRoot')` và `from laya_kev_gateway import System1DecisionGateway`.
   - Tuy nhiên, tệp `laya_kev_gateway.py` thực tế đang nằm trong `D:\KhoaLuan\laya_kev_Agent\laya\laya_kev_gateway.py`, **không hề nằm tại root** `$PSScriptRoot`. Khi thực thi từ root, script lập tức báo lỗi `ModuleNotFoundError: No module named 'laya_kev_gateway'`.
2. **Thiếu vắng hoàn toàn Context Compiler (Vi phạm R2)**:
   - Script truyền thẳng `$TaskPrompt` thô sang cho Codex hoặc Antigravity mà không hề có ngữ cảnh thu thập cục bộ, không có git diff, không có symbol signatures, không kiểm soát ngân sách token (1K - 4K tokens).
3. **Gọi Worker sai giao thức headless / streaming (Vi phạm R3)**:
   - Cả hai Worker đều bị gọi dạng interactive prompt (`-p`) đơn lẻ một lượt.
   - **Codex CLI**: Chưa sử dụng `codex exec --json` để lấy sự kiện JSONL và cấu trúc hoá kết quả trả về.
   - **Antigravity CLI**: Chưa sử dụng persistent process pool với cờ `--input-format stream-json --output-format stream-json`, làm mất hoàn toàn prompt cache giữa các turn trao đổi (không đạt tiêu chí hit cache >= 70%).
   - Chưa có logic kích hoạt Reviewer (Worker thứ hai) khi rủi ro HIGH hoặc test thất bại liên tiếp >= 2 lần.
4. **Giả lập kết quả thẩm định (Vi phạm R5)**:
   - Bước 3 hoàn toàn là "mock" bằng `Write-Host "PASS"`, không hề chạy linter cú pháp, không chạy `pytest`, không có cơ chế tự ngắt vòng lặp khi test pass.
   - Không có cấu trúc lưu trữ bộ nhớ nhiệm vụ rút gọn (Compact Task Memory).

---

## 3. ĐÁNH GIÁ HIỆN TRẠNG PHỤC VỤ CÁC YÊU CẦU R1 - R5

### R1. Kiến Trúc Cổng Quyết Định Kép (Laya + Kev)
- **Yêu cầu**: Typed decisions phi tự do (non-autoregressive), latency <= 50ms. Laya phân loại intent, complexity, budget (1K/2K/4K/8K), chỉ định 1 Worker. Kev là trọng tài phân xử khi Laya confidence < 0.90 hoặc nhãn > 20 options.
- **Hiện trạng**:
  - Mã nguồn Laya (`laya/`) và Kev (`kev/`) đã có sẵn trong dự án.
  - Tệp `laya/laya_kev_gateway.py` đã phác thảo `System1DecisionGateway`, nhưng khi gọi qua PyTorch `Router.predict()` có thể bị trễ do nạp mô hình qua mạng hoặc khởi động nặng.
- **Giải pháp**:
  - Thiết kế `LayaFastGateway` kết hợp cơ chế dual-mode:
    1. Fast Heuristic & Language Heuristic (< 5ms): Dùng `laya.lang.analyse` và quy tắc mẫu nhanh khi offline hoặc cold start.
    2. Local ONNX / ModernBERT Fast Pass (< 35ms) khi trọng số sẵn sàng.
  - Phân tầng rõ ràng: Laya xử lý phân loại chính (`confidence >= 0.90`). Nếu `confidence < 0.90` hoặc danh sách chuyên gia/lựa chọn > 20 options, tự động bàn giao cho `KevAdjudicator` (Pointer Head).

### R2. Bộ Biên Dịch Ngữ Cảnh Tinh Gọn (Context Compiler 1K - 4K tokens)
- **Yêu cầu**: Thu thập ngữ cảnh cục bộ (`git diff`, `ripgrep`, AST, symbol signatures, test target). Biên dịch thành `context_pack.json` chặt chẽ trong ngân sách 1K - 4K tokens (tối đa 8K), loại bỏ việc để LLM tự quét toàn repo.
- **Hiện trạng**: Chưa có.
- **Công cụ có sẵn**: `git.exe`, `rg.exe`, Python `ast`, thư viện `tiktoken 0.12.0`.
- **Giải pháp**: Xây dựng module `team_agent.context_compiler`:
  - `GitDiffCollector`: Lấy diff thay đổi chưa commit hoặc commit gần nhất.
  - `AstSignatureCollector`: Đọc tệp nguồn Python bằng `ast.parse`, trích xuất signatures của class, method, function, docstrings, lược bỏ thân code (implementation body).
  - `SearchCollector`: Dùng `rg` quét các symbol references và target tests liên quan.
  - `TokenBudgetEnforcer`: Dùng `tiktoken` đếm token chuẩn hoá, phân bổ ngân sách theo thứ tự ưu tiên: Task & Constraints (500t) > Target Tests (500t) > AST Signatures (1500t) > Relevant Snippets (1500t), tự động tỉa gọn để tổng không vượt quá 4K tokens.

### R3. Bộ Điều Hợp Worker Độc Quyền (Single-Worker: Codex OR Antigravity)
- **Yêu cầu**:
  - Mỗi tác vụ kích hoạt đúng 1 Worker lớn (`Codex` cho coding/fix/build, `Antigravity` cho architecture/multimodal).
  - Kích hoạt Worker 2 (Reviewer) khi: Rủi ro HIGH, Thay đổi kiến trúc, hoặc Test fail >= 2 lần.
  - Codex kết nối qua headless mode: `codex exec --json`.
  - Antigravity kết nối qua streaming process pool: `agy --input-format stream-json --output-format stream-json` giữ session hot cache >= 70%.
- **Hiện trạng**:
  - CLI `codex` và `agy` đều có sẵn trên máy và hỗ trợ chính xác các flag nói trên.
  - Chưa có adapter và process pool manager trong Python/PowerShell.
- **Giải pháp**:
  - Xây dựng `CodexHeadlessAdapter`: Gọi `codex exec --json --dangerously-bypass-approvals-and-sandbox`, parse từng dòng JSONL từ stdout, thu thập mã lỗi, diff và thông điệp hoàn tất.
  - Xây dựng `AgyStreamingSessionPool`: Duy trì tiến trình `agy.exe --input-format stream-json --output-format stream-json --dangerously-skip-permissions` thường trực trong background, giao tiếp qua stdin/stdout NDJSON, giữ nguyên session/conversation ID giữa các turn nhằm đạt tỉ lệ hit cache >= 70%.
  - Xây dựng `ReviewerTrigger`: Bộ đếm số lần test fail và bộ kiểm tra cờ rủi ro từ Laya/Kev để quyết định có triệu hồi Reviewer hay dừng lại.

### R4. Máy Chủ MCP Hợp Nhất Tối Giản (Unified LK-Context MCP Server)
- **Yêu cầu**: Đóng gói System 1 vào 1 MCP server duy nhất (`lk-context`) với tối đa 5-7 tools: `team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`.
- **Hiện trạng**: Tệp `laya/laya_kev_mcp.py` chỉ có 4 tools theo format cũ, nằm sai vị trí, chưa dùng thư viện chuẩn FastMCP.
- **Giải pháp**:
  - Viết lại server bằng `FastMCP` (đã có `fastmcp 4.0.5` trong môi trường):
    1. `team_decide(task_description)`: Laya phân loại intent, complexity, budget, chỉ định worker.
    2. `context_select(task_description, target_files)`: Kích hoạt Context Compiler xuất `context_pack.json`.
    3. `risk_decide(code_diff_or_plan)`: Đánh giá độ rủi ro (LOW/MEDIUM/HIGH).
    4. `retry_decide(failure_output, attempt_count)`: Phân tích nguyên nhân lỗi và đưa ra chiến lược retry có cấu trúc.
    5. `review_decide(risk_level, test_failures, is_arch_change)`: Ra quyết định có kích hoạt Worker thứ hai hay không.
  - Cấu hình server vào `settings.json` của Antigravity CLI và `config.toml` của Codex CLI.

### R5. Vòng Thẩm Định Cục Bộ & Bộ Nhớ Rút Gọn (Deterministic Verification & Memory)
- **Yêu cầu**:
  - Kiểm tra tính đúng đắn bằng compiler, linter (syntax check), unit test (`pytest`).
  - Script tự động ngắt vòng lặp ngay khi test pass, không cần LLM chat xác nhận dư thừa.
  - Bộ nhớ task cô đọng dưới dạng cấu trúc ngắn gọn: `{problem, root_cause, decision, changed_files, test_results}`.
- **Hiện trạng**: Chưa có.
- **Giải pháp**:
  - Xây dựng `LocalVerifier`:
    - Chạy AST syntax check cho mọi tệp Python bị thay đổi.
    - Chạy `pytest` cục bộ trên các tệp test liên quan, lấy exit code.
    - Nếu exit code == 0 -> Báo cáo PASS, phát lệnh HALT ngắt vòng lặp ngay lập tức.
  - Xây dựng `CompactMemoryManager`:
    - Ghi nhận thông tin vào tệp `compact_memory.json` sau mỗi vòng thực thi với 5 trường dữ liệu chuẩn, tuyệt đối không lưu lại transcript hội thoại thô.

---

## 4. ĐỀ XUẤT KIẾN TRÚC MÔ-ĐUN VÀ RANH GIỚI TỆP TIN (FILE BOUNDARIES)

Để đảm bảo tính độc lập, dễ bảo trì, dễ mở rộng và tuân thủ chặt chẽ nguyên tắc Separation of Concerns, đề xuất cấu trúc mô-đun hoá tập trung tại thư mục `team_agent/`:

```
D:\KhoaLuan\laya_kev_Agent\
│
├── team_agent_orchestrator.ps1           # Script PowerShell mỏng gọi Python CLI
├── team_agent/                           # Mô-đun lõi Python của hệ thống điều phối
│   ├── __init__.py
│   ├── cli.py                            # Giao diện dòng lệnh chính (python -m team_agent run "...")
│   ├── config.py                         # Cấu hình đường dẫn CLI, ngưỡng tin cậy, token budgets
│   │
│   ├── system1/                          # R1: Cổng quyết định kép Laya + Kev
│   │   ├── __init__.py
│   │   ├── laya_router.py                # Laya phân loại intent, complexity, budget (<= 50ms)
│   │   ├── kev_adjudicator.py            # Kev phân xử khi conf < 0.90 hoặc >20 options
│   │   └── gateway.py                    # Gateway hợp nhất (tích hợp offline/fast heuristic)
│   │
│   ├── context_compiler/                 # R2: Bộ biên dịch ngữ cảnh tinh gọn
│   │   ├── __init__.py
│   │   ├── compiler.py                   # Điểm vào chính tạo context_pack.json
│   │   ├── git_collector.py              # Trích xuất git diff và status
│   │   ├── ast_collector.py              # Bóc tách symbol signatures (bỏ body)
│   │   ├── search_collector.py           # Dùng rg.exe truy xuất vị trí mã nguồn
│   │   └── token_budget.py               # Dùng tiktoken kiểm soát ngân sách 1K - 4K tokens
│   │
│   ├── workers/                          # R3: Bộ điều hợp Worker & Quản lý Session Pool
│   │   ├── __init__.py
│   │   ├── base.py                       # Interface chuẩn cho Worker Adapter
│   │   ├── codex_adapter.py              # Điều khiển headless 'codex exec --json'
│   │   ├── agy_adapter.py                # Điều khiển streaming 'agy --input-format stream-json'
│   │   ├── session_pool.py               # Quản lý process pool Antigravity giữ cache nóng >= 70%
│   │   └── reviewer_trigger.py           # Bộ kích hoạt Reviewer khi rủi ro HIGH hoặc test fail >= 2
│   │
│   ├── mcp/                              # R4: Máy chủ Unified MCP Server 'lk-context'
│   │   ├── __init__.py
│   │   ├── server.py                     # FastMCP Server với 5 tools chuẩn
│   │   └── tools.py                      # Triển khai 5 công cụ lõi
│   │
│   └── verification/                     # R5: Vòng thẩm định cục bộ & Bộ nhớ rút gọn
│       ├── __init__.py
│       ├── verifier.py                   # AST syntax linter & test runner tự động ngắt
│       └── compact_memory.py             # Quản lý lưu trữ compact_memory.json
│
└── tests/                                # Bộ kiểm thử tự động toàn diện
    ├── test_system1_latency.py           # Kiểm tra độ trễ <= 50ms
    ├── test_context_compiler.py          # Kiểm tra ngân sách token 1K - 4K
    ├── test_worker_adapters.py           # Kiểm tra Codex headless và Agy streaming
    ├── test_mcp_server.py                # Kiểm tra schema 5 tools của MCP server
    └── test_verifier_memory.py           # Kiểm tra linter, test runner và compact memory
```

---

## 5. MA TRẬN ĐỐI CHIẾU ACCEPTANCE CRITERIA & CÔNG VIỆC TRIỂN KHAI

| Tiêu chí Acceptance Criteria | Trạng thái hiện tại | Giải pháp triển khai cụ thể |
|---|---|---|
| **1. Laya phân loại task <= 50ms** | Chưa đạt ổn định (do load torch nặng) | Xây dựng Fast Heuristic & Language Pre-pass trong `laya_router.py` đạt < 10ms. |
| **2. Kev chỉ kích hoạt khi Laya conf < 0.90 hoặc nhãn > 20** | Chưa có logic kích hoạt điều kiện | Cài đặt trong `gateway.py`: Kiểm tra `if conf < 0.90 or len(options) > 20: trigger_kev()`. |
| **3. Không sinh văn bản tự do ngoài JSON** | Chưa đạt ở các tầng ngoài | Mọi output của System 1 và Gateway đều ép kiểu qua Pydantic models. |
| **4. Context pack <= 4.000 tokens** | Chưa có Context Compiler | `token_budget.py` dùng `tiktoken` giới hạn chặt chẽ: tổng token tối đa 4.000 tokens. |
| **5. > 90% tác vụ chỉ tốn quota 1 Worker** | Chưa có cơ chế bảo vệ | Strict Asymmetric Routing: mặc định kích hoạt đúng 1 worker, chỉ triệu hồi reviewer khi thỏa điều kiện nghiêm ngặt. |
| **6. Antigravity session pool hit cache >= 70%** | Chưa có session pool | Giữ persistent process qua `stream-json` với `--conversation <ID>` cố định. |
| **7. Headless Codex & Antigravity trích xuất JSON** | Chưa có adapter | Viết `codex_adapter.py` và `agy_adapter.py` tự động parse stdout JSONL/NDJSON. |
| **8. MCP Server `lk-context` nạp không lỗi schema** | Bản cũ sai schema và thiếu tool | Viết lại bằng `FastMCP` với 5 tools chuẩn mực, tương thích cả Codex và Antigravity. |
| **9. Tự động ngắt vòng lặp khi test pass** | Chỉ có fake print | `verifier.py` bắt exit code của pytest/compiler, tự động gửi tín hiệu HALT ngắt vòng lặp. |

---

## 6. KẾT LUẬN & KHUYẾN NGHỊ BÀN GIAO CHO IMPLEMENTATION

1. **Hạ tầng cơ bản của máy trạm rất mạnh**: Có GPU RTX 4060, CUDA 12.4, Python 3.12, đầy đủ các thư viện hiện đại (`fastmcp`, `mcp`, `tiktoken`, `transformers`, `torch`, `pytest`) và các công cụ CLI (`codex.ps1`, `agy.exe`, `rg.exe`, `git.exe`).
2. **Kịch bản điều phối hiện tại `team_agent_orchestrator.ps1` cần được tái cấu trúc hoàn toàn**: Thay vì nhúng các đoạn script Python chắp vá trong PowerShell, PowerShell chỉ nên đóng vai trò là một CLI wrapper mỏng truyền tham số vào `python -m team_agent.cli`, nơi toàn bộ logic R1-R5 được xử lý hướng đối tượng, an toàn kiểu và dễ dàng unit test.
3. **Các nhóm module cần được triển khai tuần tự theo thứ tự phụ thuộc**:
   - Bước 1: `team_agent.system1` (R1)
   - Bước 2: `team_agent.context_compiler` (R2)
   - Bước 3: `team_agent.verification` (R5)
   - Bước 4: `team_agent.workers` (R3)
   - Bước 5: `team_agent.mcp` (R4)
   - Bước 6: Tích hợp vào `team_agent.cli` và nâng cấp `team_agent_orchestrator.ps1`.
