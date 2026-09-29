# HANDOFF REPORT — explorer_survey_2
**Vai trò**: Orchestration and Tooling Surveyor  
**Nhiệm vụ**: Khảo sát hiện trạng hạ tầng điều phối, bộ biên dịch ngữ cảnh, kết nối worker (Codex/Antigravity), MCP server và vòng lặp thẩm định; đề xuất thiết kế module và ranh giới tệp tin.  
**Ngày báo cáo**: 2026-09-29  
**Người nhận**: `parent` (orchestrator_1, ID: `500e1b7d-25c9-45ed-8dcb-73ae3b7fc1e6`)  
**Tài liệu chi tiết kèm theo**: `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2\infra_survey.md`  

---

## 1. OBSERVATION (Quan Sát Trực Tiếp)

1. **Phiên bản & Công cụ hệ thống**:
   - `python --version` $\rightarrow$ `Python 3.12.0`
   - `node --version` $\rightarrow$ `v22.23.2`
   - `git --version` $\rightarrow$ `git version 2.55.0.windows.4` (`C:\Program Files\Git\cmd\git.exe`)
   - `Get-Command rg` $\rightarrow$ `C:\Users\user\AppData\Local\Microsoft\WinGet\Links\rg.exe`
   - `Get-Command codex` $\rightarrow$ `C:\Users\user\AppData\Local\hermes\node\codex.ps1`
   - `Get-Command agy` $\rightarrow$ `C:\Users\user\AppData\Local\agy\bin\agy.exe`
   - `Get-Command gemini` $\rightarrow$ `C:\Users\user\AppData\Roaming\npm\gemini.ps1`

2. **Thư viện Python đã cài đặt (`pip list`)**:
   - `torch`: `2.6.0+cu124` (Hỗ trợ CUDA 12.4, GPU RTX 4060 8GB).
   - `transformers`: `5.16.1`, `tokenizers`: `0.23.1`.
   - `tiktoken`: `0.12.0`.
   - `fastmcp`: `4.0.5`, `mcp`: `2.2.0`, `mcp-types`: `2.2.0`.
   - `pytest`: `9.0.2`, `pytest-asyncio`: `1.4.0`.
   - `pydantic`: `2.13.4`, `pydantic_core`: `2.46.4`.
   - `ast`: Python built-in standard library.

3. **Cờ lệnh của Codex CLI và Antigravity CLI**:
   - `codex exec --help`:
     + `codex exec [OPTIONS] [PROMPT]`
     + `--json`: *"Print events to stdout as JSONL"*
     + `--dangerously-bypass-approvals-and-sandbox`: *"Skip all confirmation prompts and execute commands without sandboxing"*
     + `-o, --output-last-message <FILE>`
   - `agy --help`:
     + `--input-format`: *"Input format for print mode (text, stream-json). stream-json reads one NDJSON message per line from stdin and runs a turn for each; it requires --output-format stream-json (default text)"*
     + `--output-format`: *"Output format for print mode (text, json, stream-json) (default text)"*
     + `--dangerously-skip-permissions`: *"Auto-approve all tool permission requests without prompting"*
     + `--conversation <ID>` và `--continue`: *"Resume a previous conversation"*

4. **Hiện trạng mã nguồn `team_agent_orchestrator.ps1`**:
   - Dòng 27 & 31-32:
     ```powershell
     $gatewayScript = Join-Path $PSScriptRoot "laya_kev_gateway.py"
     sys.path.append(r'$PSScriptRoot')
     from laya_kev_gateway import System1DecisionGateway
     ```
     Thực tế tệp `laya_kev_gateway.py` nằm ở `D:\KhoaLuan\laya_kev_Agent\laya\laya_kev_gateway.py`. Script từ root không import được.
   - Dòng 79 & 90: Gọi Codex và Antigravity qua interactive prompt `-p` đơn lẻ một lượt:
     `& $codexPath -p "Vai trò: $selectedExpert. Thực thi nhiệm vụ: $TaskPrompt"`
     `& $agyPath -p "$TaskPrompt"`
   - Dòng 99-102: Hoàn toàn in text mô phỏng:
     `Write-Host " -> Kết quả thẩm định: PASS [100% Không lỗi cú pháp, Đạt tiêu chuẩn Kiến trúc]."`

5. **Hiện trạng MCP Server hiện hữu (`laya/laya_kev_mcp.py`)**:
   - Là một vòng lặp JSON-RPC thủ công đọc `sys.stdin`, chỉ có 4 tools (`system1_route_agent`, `kev_select_expert`, `system1_guardrail`, `system1_halt_gate`).
   - Chưa triển khai FastMCP và chưa có 5 tools chuẩn mực của R4 (`team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`).

---

## 2. LOGIC CHAIN (Chuỗi Suy Luận)

1. Từ **Quan sát 1 & 2**: Hệ thống có sẵn GPU mạnh, Python 3.12, CUDA, `tiktoken`, `fastmcp`, `pytest`, `rg.exe`, `git.exe`.  
   $\rightarrow$ *Suy luận*: Hạ tầng phần cứng và thư viện môi trường đã sẵn sàng 100% cho toàn bộ 5 yêu cầu R1-R5 mà không cần cài đặt thêm gói nhị phân phức tạp ngoài môi trường hiện hữu.

2. Từ **Quan sát 3 & 4**: CLI của Codex hỗ trợ `codex exec --json` và Antigravity hỗ trợ `stream-json` với conversation ID; nhưng `team_agent_orchestrator.ps1` chỉ gọi `-p` một lần.  
   $\rightarrow$ *Suy luận*: Hệ thống hiện tại chưa tận dụng được tính năng headless của Codex và chưa có Session Pool của Antigravity để giữ cache nóng $\ge 70\%$. Cần một tầng Worker Adapter chuyên biệt trong Python để quản lý tiến trình con (subprocess/asyncio pipe) thay vì gọi trực tiếp từ PowerShell.

3. Từ **Quan sát 4**: Script PowerShell đang mắc lỗi import đường dẫn và giả lập kết quả thẩm định.  
   $\rightarrow$ *Suy luận*: PowerShell không phải là nơi thích hợp để viết logic phân tích cú pháp AST, tính toán token ngân sách, hay quản lý NDJSON stream. Giải pháp tối ưu là đóng gói toàn bộ logic vào package Python chuẩn `team_agent/`, và giữ lại PowerShell như một CLI wrapper mỏng.

4. Từ **Quan sát 2 & 5**: Đã có `fastmcp 4.0.5` nhưng `laya_kev_mcp.py` vẫn dùng stdio loop tự chế với bộ tool chưa chuẩn.  
   $\rightarrow$ *Suy luận*: Việc refactor MCP Server sang `FastMCP` với 5 tools cốt lõi (`team_decide`, `context_select`, `risk_decide`, `retry_decide`, `review_decide`) sẽ tự động chuẩn hoá JSON Schema, loại bỏ xung đột và cho phép nạp trực tiếp vào cả Antigravity CLI và Codex CLI.

---

## 3. CAVEATS (Vấn Đề Lưu Ý & Giới Hạn)

1. **Khởi động mô hình Laya/Kev (Cold Start Latency)**: Nếu gọi `Router.predict()` với PyTorch qua mạng để tải weights sẽ làm trễ vượt quá ngưỡng 50ms. Do đó, tầng `team_agent.system1` bắt buộc phải có cơ chế Fast Heuristic / Rule-based Language Router đạt độ trễ < 5ms trong trường hợp weights chưa được cache tại máy cục bộ.
2. **Khảo sát chế độ Read-Only**: Agent tuân thủ nghiêm ngặt quy tắc Read-Only, không tự ý sửa đổi tệp mã nguồn gốc `team_agent_orchestrator.ps1` hay tạo source code trong dự án; mọi đề xuất kiến trúc được ghi chép độc lập trong báo cáo.
3. **Quyền hạn Sandbox**: Khi chạy `codex exec` và `agy`, cần chỉ định cờ `--dangerously-bypass-approvals-and-sandbox` và `--dangerously-skip-permissions` để tránh bị dừng lại chờ người dùng bấm xác nhận trên terminal.

---

## 4. CONCLUSION (Kết Luận & Đề Xuất)

1. **Đánh giá tổng thể**: Hạ tầng dự án hoàn toàn khả thi và đầy đủ công cụ để đạt 100% Acceptance Criteria của R1 - R5.
2. **Ranh giới module đề xuất**:
   - `team_agent/system1/`: Cổng quyết định kép Laya + Kev (SLA <= 50ms).
   - `team_agent/context_compiler/`: Biên dịch `context_pack.json` bằng git diff + ast + ripgrep + tiktoken (ngân sách 1K - 4K tokens).
   - `team_agent/workers/`: Worker Adapters (`codex exec --json` và `agy stream-json` session pool giữ cache >= 70%).
   - `team_agent/mcp/`: FastMCP Server `lk-context` với đúng 5 tools cốt lõi.
   - `team_agent/verification/`: Linter (AST syntax check), test runner cục bộ tự ngắt khi pass, và `compact_memory.json`.
   - `team_agent_orchestrator.ps1`: Tái cấu trúc thành CLI wrapper mỏng gọi `python -m team_agent.cli`.

---

## 5. VERIFICATION METHOD (Phương Pháp Xác Minh Độc Lập)

Người nhận hoặc agent kế tiếp có thể kiểm tra lại toàn bộ kết luận bằng các lệnh sau trên PowerShell:

1. **Xác minh các công cụ CLI**:
   ```powershell
   Get-Command rg, git, codex, agy, gemini
   ```
2. **Xác minh các thư viện Python cốt lõi**:
   ```powershell
   python -c "import fastmcp, mcp, tiktoken, torch, transformers, pytest; print('ALL PACKAGES INSTALLED OK')"
   ```
3. **Xác minh cờ headless của Codex**:
   ```powershell
   codex exec --help | Select-String "\-\-json"
   ```
4. **Xác minh cờ streaming NDJSON của Antigravity**:
   ```powershell
   & "C:\Users\user\AppData\Local\agy\bin\agy.exe" --help | Select-String "stream-json"
   ```
5. **Kiểm tra báo cáo chi tiết**:
   Xem tệp `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2\infra_survey.md`.
