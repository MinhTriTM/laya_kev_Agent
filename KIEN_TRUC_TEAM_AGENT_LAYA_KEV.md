# KIẾN TRÚC ĐA AGENT SONG MÃ (TEAM AGENT) 2026: TÍCH HỢP LAYA & KEV VÀO ANTIGRAVITY CLI VÀ CODEX CLI

- **Tổng Chỉ Huy**: Tổng Giám Đốc (User)
- **Tầng Phản Xạ Thần Kinh (System 1 Decision Engines)**: Laya & Kev (~5ms - 35ms)
- **Tầng Động Não & Thực Thi Sâu (System 2 Multi-Agent CLI)**: Antigravity CLI (Giám Đốc B) & Codex CLI (Phó Giám Đốc Codex)

---

## 1. Bản Chất Của Laya & Kev Trong Hệ Thống Agent 2026

Trong các hệ thống Agent trước đây, mọi quyết định vi mô (chọn tool, chọn subagent, kiểm tra logic yes/no, đánh giá code xong chưa) đều phải gọi qua các LLM khổng lồ (System 2: GPT-4o, Claude 3.5 Sonnet, Gemini Pro). Điều này gây ra:
1. **Lãng phí token**: Mỗi quyết định mất 500 - 3.000 token system prompt.
2. **Độ trễ cao (Latency)**: Mất từ 2 đến 10 giây cho một câu trả lời yes/no hoặc chọn 1 lựa chọn.
3. **Ảo giác (Hallucination)**: LLM sinh văn bản có thể bị sai định dạng JSON, drift ngữ cảnh.

**Laya** và **Kev** ra đời để giải quyết triệt để vấn đề này theo cơ chế **System 1 Non-Autoregressive Decision Engine**:
- **Laya** (ModernBERT-large 421M / mmBERT-base 322M):
  - Đánh giá câu hỏi kiểu gõ (`choice`, `score`, `noul`) trong **1 forward pass duy nhất (33ms)**.
  - Tự động định tuyến đa ngôn ngữ (100+ ngôn ngữ).
  - Không sinh token văn bản $\rightarrow$ **0% ảo giác cú pháp, tốc độ tức thì**.
- **Kev** (Qwen 3.5 với LoRA Adapter & Pointer Head):
  - Xử lý không gian nhãn khổng lồ (**High-cardinality label space > 20 đến 255 options** cùng lúc).
  - Chọn chính xác 1 chuyên gia trong nhóm 40 GS/PGS/TS của Codex hay Antigravity mà không làm nghẽn token budget.
- **Ollaya**: Runtime máy chủ cục bộ (tương tự Ollama) cung cấp API chuẩn Jev/TypeSafe cho Laya & Kev.

---

## 2. Cách "Cắm" Laya & Kev Vào Antigravity CLI Và Codex CLI

### 2.1 Cắm vào Antigravity CLI (Gemini 3.1 Pro / Flash)
Antigravity CLI hỗ trợ giao thức **Model Context Protocol (MCP)** gốc.

1. **Khởi động MCP Server Laya/Kev**:
   Tệp `laya_kev_mcp.py` đã được tạo sẵn trong thư mục dự án.
2. **Khai báo trong cấu hình Antigravity CLI**:
   Mở tệp `C:\Users\user\.gemini\antigravity-cli\settings.json`, bổ sung vào mục `mcpServers`:
   ```json
   "laya-kev-engine": {
     "command": "python",
     "args": ["D:\\KhoaLuan\\laya_kev_Agent\\laya_kev_mcp.py"]
   }
   ```
3. **Các Tool System 1 Antigravity CLI sở hữu ngay lập tức**:
   - `system1_route_agent`: Phân loại ý định, rẽ nhánh nhiệm vụ trong 20ms.
   - `kev_select_expert`: Chọn 1 chuyên gia phù hợp trong hàng chục chuyên gia mà không tốn prompt.
   - `system1_guardrail`: Quét lỗ hổng, phát hiện lỗi cú pháp, tính an toàn của mã nguồn.
   - `system1_halt_gate`: Đánh giá điều kiện dừng vòng lặp (Halt criteria) sớm, tiết kiệm 90% token.

---

### 2.2 Cắm vào Codex CLI (GPT-5 / Claude 3.7 Sonnet)
Codex CLI hỗ trợ cấu hình MCP hoặc qua lớp bọc điều phối CLI Hook.

1. **Cách 1: Khai báo MCP vào Codex CLI**:
   Bổ sung MCP Server vào `C:\Users\user\.codex\config.json` (tương tự như Antigravity CLI).
2. **Cách 2: Sử dụng Shell Pre-flight & Post-flight Hook (Khuyên dùng)**:
   - Trước khi gửi lệnh vào `codex.ps1`, chạy một lệnh gọi nhanh qua `laya_kev_gateway.py` để rút gọn prompt.
   - Sau khi Codex CLI sinh mã, Kev chạy 1 forward pass để verify code trước khi commit vào git.

---

## 3. Cách "Cầm Cương" Để Code Đạt Sức Mạnh Tối Đa & Giảm Hao Tổn Nhất

Để đạt sức mạnh vô địch ("Mạnh nhất") và giảm thiểu chi phí/độ trễ ("Giảm có thể"), Tổng Giám Đốc chỉ đạo theo **Mô hình Song Mã 3 Tầng**:

```
                  [TỔNG GIÁM ĐỐC]
                         │ (Mệnh lệnh cấp cao)
                         ▼
        ┌───────────────────────────────────┐
        │ TẦNG 1: SYSTEM 1 GATEWAY          │
        │ - Laya Router: Phân luồng task    │  Thời gian: 15 - 30ms
        │ - Kev: Match chuyên gia 1/40      │  Token LLM: 0 TOKEN
        └─────────────────┬─────────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────┐
│ GIÁM ĐỐC B (ANTIGRAVITY) │    │ PHÓ GIÁM ĐỐC CODEX       │
│ - Gemini 3.1 Pro / Flash │    │ - GPT-5 / Claude Sonnet  │
│ - Thiết kế Kiến trúc     │    │ - Viết Code Hệ thống     │
│ - Multimodal (Ảnh, KG)   │    │ - Build, Compile, Fix    │
│ - Spec, Data Pipeline    │    │ - Memory, OS, C#, Rust   │
└─────────────┬────────────┘    └─────────────┬────────────┘
              │                               │
              └───────────────┬───────────────┘
                              ▼
        ┌───────────────────────────────────┐
        │ TẦNG 3: FAST QUALITY & HALT GATE  │
        │ - Laya NOUL: Check an toàn & lỗi  │  Thời gian: 25ms
        │ - Halt Gate: Ngắt vòng lặp sớm    │  Tiết kiệm 80-90% token
        └─────────────────┬─────────────────┘
                          │
                          ▼
                  [KẾT QUẢ CUỐI CÙNG]
```

### 3.1 Bốn Chiến Lược "Cầm Cương" Thực Chiến

#### Chiến lược 1: "Zero-Token Expert Dispatching" (Dùng Kev trị 40 Chuyên gia)
- **Vấn đề**: Bản quy chế Q247 có 40 chuyên gia cho Codex, 40 chuyên gia cho Gemini. Nếu nhét cả 40 mô tả này vào System Prompt của LLM, mỗi turn chat tốn thêm 2.000 token vô ích.
- **Giải pháp**: Giao danh sách 40 chuyên gia cho **Kev Pointer Head**. Kev đọc task của Tổng Giám Đốc, trong 20ms chỉ định chính xác: `"PGS. Le Thi B - Chuyên gia Tối ưu RAM và Garbage Collection C#"`.
- **Kết quả**: System prompt gửi sang Codex CLI chỉ cần 1 dòng vai trò duy nhất. **Tiết kiệm 80% chi phí token đầu vào!**

#### Chiến lược 2: "Asymmetric Bipolar Pairing" (Phân công Bất Đối Xứng Sở Trường)
- **Antigravity CLI (Gemini 3.1 Pro)**: Sở hữu cửa sổ ngữ cảnh khổng lồ (2M token) và khả năng xử lý Multimodal (ảnh côn trùng, sơ đồ kiến trúc, tài liệu khoa học). Dùng Antigravity để:
  + Lập kế hoạch dự án tổng thể (`/plan`).
  + Thiết kế Knowledge Graph và Pipeline dữ liệu.
  + Đọc tài liệu tham khảo lớn (Doc / PDF / Specs).
- **Codex CLI (GPT-5 / Claude Sonnet)**: Khả năng coding chuẩn xác, dứt khoát trong môi trường Terminal. Dùng Codex để:
  + Triển khai chi tiết từng file mã nguồn theo thiết kế của Antigravity.
  + Chạy lệnh `dotnet build`, `pytest`, sửa lỗi compiler tại chỗ.

#### Chiến lược 3: "Single-Pass Fast Gate" (Laya NOUL chặn đứng Hallucination)
- Trước khi thực thi các lệnh nguy hiểm (xóa database, drop table, ghi đè file lớn), Laya NOUL đánh giá xác suất rủi ro trong 10ms. Nếu rủi ro > 0.85, chặn lại và cảnh báo Tổng Giám Đốc.
- Không để LLM tự phán đoán an toàn vì LLM có thể bị "lú" (hallucination).

#### Chiến lược 4: "Early Termination Halt Gate" (Ngắt vòng lặp tự động)
- Khi cho Agent tự động chạy sửa lỗi (Auto-loop), thay vì sau mỗi bước lại gọi LLM tốn 3-5 giây để hỏi "Đã xong chưa?", hãy nạp output biên dịch vào Laya với câu hỏi NOUL: `"has_task_completed_successfully"`.
- Khi độ tin cậy > 0.90, ngắt vòng lặp ngay lập tức $\rightarrow$ **Cắt giảm hoàn toàn 2 - 3 turn chat dư thừa, tăng tốc độ hoàn thành dự án gấp 3 lần.**

---

## 4. Hướng Dẫn Vận Hành Hằng Ngày Cho Tổng Giám Đốc

### Cách 1: Chạy Tự Động Toàn Diện Qua Bộ Điều Phối
Tổng Giám Đốc chỉ cần mở PowerShell và gõ:
```powershell
.\team_agent_orchestrator.ps1 "Viết service tối ưu bộ nhớ MemoryPool trong C# và chạy test"
```
Hệ thống sẽ:
1. Laya phân loại task $\rightarrow$ giao cho Codex CLI.
2. Kev chọn chuyên gia Tối ưu RAM C#.
3. Codex CLI mở terminal code và build.
4. Laya NOUL kiểm tra chất lượng kết quả.

### Cách 2: Phối Hợp Song Mã Bằng Tay Khi Cần Độ Tỉ Mỉ
1. **Bước 1 (Antigravity CLI)**: Yêu cầu Giám Đốc B thiết kế kiến trúc:
   ```
   agy -p "Thiết kế kiến trúc Knowledge Graph Multimodal côn trùng và lưu vào THIET_KE.md"
   ```
2. **Bước 2 (Codex CLI)**: Yêu cầu Codex CLI code dựa trên file thiết kế:
   ```
   codex -p "Đọc THIET_KE.md do Antigravity thiết kế, triển khai code Python backend và chạy test"
   ```
3. **Bước 3 (Laya/Kev)**: Gọi tool `system1_guardrail` để audit toàn diện trong 30ms.
