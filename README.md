# laya_kev_Agent

> **TeamAgent Orchestrator**: Hệ thống điều phối đa Agent song mã kết hợp **Codex CLI** và **Antigravity CLI**, ứng dụng hai System-1 Decision Engines là **Laya** và **Kev** cùng bộ **Context Compiler** tinh gọn nhằm giảm tối đa token quota và tăng tốc độ tự động hóa phát triển phần mềm.

---

## 🚀 Điểm Nhấn Kiến Trúc (Architecture Highlights)

1. **Dual System-1 Gate (Laya + Kev)**:
   - **Laya** (ModernBERT/mmBERT): Đóng vai trò First-Responder (always-on), phân loại ý định, độ phức tạp, và ngân sách ngữ cảnh trong $\le 35$ms.
   - **Kev** (Qwen 3.5 LoRA + Pointer Head): Đóng vai trò Trọng tài phân xử (Decision Adjudicator) khi độ tin cậy của Laya $< 0.90$ hoặc không gian nhãn lớn ($> 20$ options).
   - **Không sinh văn bản tự do**: Quyết định dạng kiểu gõ (Typed Decisions: `choice`, `score`, `noul`) trong 1 forward pass duy nhất, loại bỏ hoàn toàn nguy cơ ảo giác cú pháp.

2. **Quy Tắc Thực Thi Bất Đối Xứng (Codex OR Antigravity)**:
   - Mỗi tác vụ chỉ kích hoạt đúng 1 Worker lớn:
     - **Codex CLI**: Chuyên trách sửa code, build solution, fix lỗi biên dịch và tối ưu hệ thống.
     - **Antigravity CLI**: Chuyên trách kiến trúc, nghiên cứu sâu, multimodal (ảnh/đồ họa/KG), và spec dữ liệu.
   - Worker thứ 2 chỉ đóng vai trò Reviewer trong các trường hợp rủi ro cao hoặc test thất bại liên tiếp $\ge 2$ lần.

3. **Deterministic Context Compiler**:
   - Sử dụng AST, tree-sitter, ripgrep, git diff để trích xuất chính xác phạm vi liên quan.
   - Đóng gói thành `context_pack.json` cô đọng từ **1K đến 4K tokens**, thay vì nạp toàn bộ mã nguồn dự án (30K - 100K tokens).

4. **Unified MCP Server (`lk-context`)**:
   - Cung cấp 5-7 công cụ cốt lõi tích hợp trực tiếp vào Antigravity CLI và Codex CLI mà không làm phình tool surface của LLM.

---

## 📂 Cấu Trúc Thư Mục

```
laya_kev_Agent/
├── .agents/                    # Cấu hình đa agent và tiến trình Teamwork
├── kev/                        # Mã nguồn và checkpoint của Kev Engine
├── laya/                       # Mã nguồn và checkpoint của Laya Engine
├── Lich_Su_Truy_Van/           # Nhật ký truy vấn và lịch sử quyết định
├── laya_kev_gateway.py         # Cổng dịch vụ System 1 nội bộ
├── laya_kev_mcp.py             # Model Context Protocol (MCP) Server
├── team_agent_orchestrator.ps1 # Kịch bản điều phối song mã thực chiến
├── KIEN_TRUC_TEAM_AGENT_LAYA_KEV.md # Cẩm nang thiết kế kiến trúc toàn diện
└── README.md                   # Tài liệu dự án
```

---

## 🛠️ Hướng Dẫn Vận Hành

### Chạy Tự Động Toàn Trình (Orchestrator):
```powershell
.\team_agent_orchestrator.ps1 "Viết service tối ưu bộ nhớ trong C# và chạy test"
```

### Cấu Hình MCP Cho Antigravity CLI & Codex CLI:
Thêm vào cấu hình MCP (`settings.json` hoặc `config.json`):
```json
{
  "mcpServers": {
    "lk-context": {
      "command": "python",
      "args": ["D:\\KhoaLuan\\laya_kev_Agent\\laya_kev_mcp.py"]
    }
  }
}
```

---

*Phát triển bởi: MinhTriTM - Trường Đại Học Đồng Tháp (DThU)*
