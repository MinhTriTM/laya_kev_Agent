# DISPATCH — explorer_survey_1

## Mission
Khảo sát hiện trạng kỹ thuật của hai engine System-1 là Laya và Kev trong dự án.

## Nguồn tài liệu và mã nguồn cần đọc:
- `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md` (BẮT BUỘC ĐỌC ĐẦU TIÊN)
- `D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md`
- Thư mục `D:\KhoaLuan\laya_kev_Agent\laya/` và `D:\KhoaLuan\laya_kev_Agent\kev/` cùng các file bên trong.

## Nhiệm vụ cụ thể:
1. Kiểm tra cấu trúc hiện tại của `laya/` và `kev/`: Các script, mô hình (ONNX, fasttext, scikit-learn, rule-based,...), dataset, benchmark nếu có.
2. Đánh giá khả năng đáp ứng R1: typed decisions <= 50ms, non-autoregressive JSON output, cơ chế chuyển giao (escalation/fallback) từ Laya sang Kev khi confidence < 0.90 hoặc nhãn > 20 options.
3. Chỉ ra các điểm còn thiếu (gaps), các file cần thêm hoặc chỉnh sửa, cơ chế kiểm thử tính chuẩn xác và độ trễ.
4. Báo cáo chi tiết vào `D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_1\laya_kev_survey.md` và `handoff.md`.


## 2026-09-29T11:18:19Z
[Message from parent 500e1b7d-25c9-45ed-8dcb-73ae3b7fc1e6]
Bạn là explorer_survey_1 (Role: Laya and Kev Engine Surveyor).
Thư mục làm việc của bạn: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_1
Đọc file chỉ thị tại: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_1\DISPATCH.md
Và file yêu cầu gốc bắt buộc: D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\ORIGINAL_REQUEST.md
Cùng tài liệu kiến trúc: D:\KhoaLuan\laya_kev_Agent\KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md

Nhiệm vụ:
1. Khảo sát kỹ lưỡng toàn bộ mã nguồn, mô hình, dữ liệu và script trong thư mục D:\KhoaLuan\laya_kev_Agent\laya/ và D:\KhoaLuan\laya_kev_Agent\kev/.
2. Đánh giá hiện trạng của Laya (tốc độ phân loại <= 50ms, typed decision schema, non-autoregressive JSON, intent, complexity, context budget, single worker designation) và Kev (khi confidence < 0.90 hoặc nhãn > 20 options).
3. Xác định các thiếu hụt (gaps), các thành phần đã có và các thành phần cần xây dựng mới hoặc tối ưu để đạt toàn bộ Acceptance Criteria của R1.
4. Xuất báo cáo chi tiết vào D:\KhoaLuan\laya_kev_Agent\.agents\teamwork\explorer_survey_1\laya_kev_survey.md và handoff.md trong thư mục làm việc của bạn.
Sau khi hoàn thành, dùng send_message gửi thông báo cho parent (orchestrator_1) kèm đường dẫn file báo cáo.
