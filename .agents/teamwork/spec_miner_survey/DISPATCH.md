# DISPATCH — spec_miner_survey

## Mission
Khai thác toàn diện các yêu cầu kỹ thuật, thông số ràng buộc và tiêu chí nghiệm thu từ các nguồn đặc tả chính thức của dự án.

## Nguồn tài liệu bắt buộc đọc:
- `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md` (BẮT BUỘC ĐỌC ĐẦU TIÊN)
- `D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`

## Nhiệm vụ cụ thể:
1. Bóc tách từng yêu cầu cụ thể của R1 (Dual System-1 Gate: Laya + Kev), R2 (Deterministic Context Compiler), R3 (Single-Worker Execution), R4 (Unified LK-Context MCP Server), R5 (Verification & Memory).
2. Liệt kê toàn bộ Acceptance Criteria và chỉ số định lượng (độ trễ <=50ms, non-autoregressive JSON, token 1K-4K max 8K, cache hit >=70%, single worker >90%, v.v.).
3. Đặc tả giao diện (schemas) cho các quyết định của Laya/Kev và 5-7 MCP tools của server `lk-context`.
4. Xuất báo cáo chi tiết vào `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey\spec_report.md` và `handoff.md`.

## 2026-09-29T11:18:19Z
[Message] timestamp=2026-09-29T11:18:19Z sender=500e1b7d-25c9-45ed-8dcb-73ae3b7fc1e6 priority=MESSAGE_PRIORITY_HIGH
Bạn là spec_miner_survey (Role: Specification Investigator).
Thư mục làm việc của bạn: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey
Đọc file chỉ thị tại: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey\DISPATCH.md
Và file yêu cầu gốc bắt buộc: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md
Cùng tài liệu kiến trúc: D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md

Nhiệm vụ:
1. Bóc tách chi tiết toàn bộ yêu cầu kỹ thuật R1 đến R5 và các Acceptance Criteria từ ORIGINAL_REQUEST.md và KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md.
2. Liệt kê toàn bộ các thông số kỹ thuật, định dạng JSON chuẩn (typed decisions), tiêu chí kích hoạt Kev, ngân sách token context (1K-4K, max 8K), quy tắc kích hoạt Single-Worker và Reviewer, danh sách 5-7 MCP tools của server lk-context, và quy trình kiểm thử/bộ nhớ R5.
3. Xuất báo cáo chi tiết vào D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\spec_miner_survey\spec_report.md và handoff.md trong thư mục làm việc của bạn.
Sau khi hoàn thành, dùng send_message gửi thông báo cho parent (orchestrator_1) kèm đường dẫn file báo cáo.
