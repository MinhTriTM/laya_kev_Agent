# DISPATCH — explorer_survey_2

## Mission
Khảo sát hiện trạng hạ tầng điều phối, bộ biên dịch ngữ cảnh, kết nối worker (Codex/Antigravity), MCP server và vòng lặp thẩm định.

## Nguồn tài liệu và mã nguồn cần đọc:
- `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md` (BẮT BUỘC ĐỌC ĐẦU TIÊN)
- `D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`
- File `D:\KhoaLuan\laya_kev_Agent\team_agent_orchestrator.ps1` và toàn bộ các file/thư mục khác ở root dự án (tools, tests, scripts, configs).

## Nhiệm vụ cụ thể:
1. Phân tích kịch bản điều phối hiện có `team_agent_orchestrator.ps1`: Luồng xử lý, cách gọi worker, cách quản lý context.
2. Khảo sát hiện trạng R2: Đã có Context Compiler (AST/tree-sitter/diff) chưa? Cần những công cụ/thư viện nào (ripgrep, git, ast, tiktoken,...)?
3. Khảo sát hiện trạng R3: Cách kết nối `codex exec --json` và `agy --input-format stream-json --output-format stream-json`, session pool tái sử dụng cache.
4. Khảo sát hiện trạng R4: Đã có MCP server `lk-context` chưa? Triển khai bằng Python (FastMCP / mcp sdk) hay TypeScript?
5. Khảo sát hiện trạng R5: Cơ chế linter, test runner, cấu trúc lưu bộ nhớ rút gọn.
6. Báo cáo chi tiết vào `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2\infra_survey.md` và `handoff.md`.

## 2026-09-29T11:18:19Z
Từ: 500e1b7d-25c9-45ed-8dcb-73ae3b7fc1e6 (orchestrator_1)
Nội dung:
Bạn là explorer_survey_2 (Role: Orchestration and Tooling Surveyor).
Thư mục làm việc của bạn: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2
Đọc file chỉ thị tại: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2\DISPATCH.md
Và file yêu cầu gốc bắt buộc: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md
Cùng tài liệu kiến trúc: D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md

Nhiệm vụ:
1. Khảo sát file D:\KhoaLuan\laya_kev_Agent\team_agent_orchestrator.ps1 và toàn bộ môi trường root dự án (Python, Node, các gói phụ thuộc, công cụ AST/tree-sitter/git diff, công cụ CLI codex và agy nếu có).
2. Đánh giá hiện trạng hạ tầng phục vụ R2 (Context Compiler 1K-4K tokens), R3 (Single-Worker execution, stream-json session pool cho agy, headless json cho codex), R4 (Unified MCP server lk-context với 5-7 tools), R5 (Local verification linter/test & compact memory format).
3. Đề xuất kiến trúc module cụ thể, ranh giới file, và các công việc cần triển khai để đạt toàn bộ Acceptance Criteria.
4. Xuất báo cáo chi tiết vào D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_2\infra_survey.md và handoff.md trong thư mục làm việc của bạn.
Sau khi hoàn thành, dùng send_message gửi thông báo cho parent (orchestrator_1) kèm đường dẫn file báo cáo.
