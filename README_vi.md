# WPS Skills

[English](README.md) | [Tiếng Việt](README_vi.md) | [中文](README_zh.md)

> Bất kỳ cập nhật nào về tính năng hoặc kiến trúc của dự án này đều phải được đồng bộ hóa vào tài liệu liên quan sau khi hoàn tất. Đây là một phần trong quy ước của dự án.

Bộ công cụ MCP cho phép trợ lý lập trình AI điều khiển WPS Office thông qua ngôn ngữ tự nhiên.

## Định vị dự án

Dự án này là một framework kết hợp giữa MCP Server + Skills, giúp các trợ lý AI (Claude Code, Cursor, Augment, Codex CLI, Windsurf, v.v.) có thể điều khiển trực tiếp WPS Office.
- 231 công cụ MCP chuyên dụng + 12 công cụ tích hợp sẵn = tổng cộng 243 công cụ
- Hỗ trợ Excel (82 công cụ) / Word (28 công cụ) / PPT (112 công cụ) / Dùng chung (9 công cụ)
- Hỗ trợ macOS, Windows và Linux

## Yêu cầu tiên quyết

- Đã cài đặt WPS Office ([https://www.wps.com/](https://www.wps.com/) hoặc [https://www.wps.cn/](https://www.wps.cn/))
- Node.js >= 18.0.0
- Git

## Cài đặt tự động (Dành cho AI thực thi)

Các bước sau đây được thiết kế để trợ lý AI thực thi tự động. Các bước cần con người thao tác được đánh dấu ⚠️.

### Bước 1: Clone dự án

```bash
git clone https://github.com/lc2panda/wps-skills.git
cd wps-skills
```

Nếu dự án đã tồn tại trong máy, bỏ qua bước này.

### Bước 2: Cài đặt dependencies và build

```bash
cd wps-office-mcp
npm install
npm run build
cd ..
```

### Bước 3: Cấu hình MCP Server

Tùy theo công cụ AI bạn đang dùng, thêm cấu hình sau vào file tương ứng. Chú ý thay thế đường dẫn thành đường dẫn tuyệt đối thực tế của dự án.

**Claude Code** — ghi vào `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "wps-office": {
      "command": "node",
      "args": ["/DUONG_DAN_CUA_BAN/wps-skills/wps-office-mcp/dist/index.js"]
    }
  }
}
```

**Cursor** — ghi vào thư mục gốc dự án `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "wps-office": {
      "command": "node",
      "args": ["/DUONG_DAN_CUA_BAN/wps-skills/wps-office-mcp/dist/index.js"]
    }
  }
}
```

**OpenAI Codex CLI** — ghi vào `~/.codex/config.toml`:
```toml
[mcp_servers.wps-office]
command = "node"
args = ["/DUONG_DAN_CUA_BAN/wps-skills/wps-office-mcp/dist/index.js"]
```
Hoặc đăng ký bằng dòng lệnh: `codex mcp add wps-office -- node /DUONG_DAN_CUA_BAN/wps-skills/wps-office-mcp/dist/index.js`

**Augment / Các IDE tương thích MCP khác** — Tham khảo tài liệu cấu hình MCP Server của từng IDE, sử dụng cùng command và args như trên. MCP Server của dự án là triển khai stdio tiêu chuẩn (spec 2025-11-25), tương thích với mọi MCP first-class client (Claude Code / Cursor / Codex CLI / GitHub Copilot CLI / Windsurf, v.v.).

### Bước 4: Cài đặt Add-on cho WPS

⚠️ Yêu cầu thao tác thủ công (AI không thể thao tác trực tiếp vào giao diện ứng dụng WPS):

```bash
# macOS
bash scripts/auto-install-mac.sh

# Windows (PowerShell)
powershell scripts/install.ps1

# Linux
bash scripts/install.sh
```

⚠️ Sau khi cài đặt, bạn **bắt buộc phải khởi động lại WPS Office** để add-on có hiệu lực.

### Bước 5: Cài đặt Skills (Chỉ cần thiết cho Claude Code)

```bash
# Tạo thư mục skills (nếu chưa có)
mkdir -p ~/.claude/skills

# Tạo symbolic link
ln -sf "$(pwd)/skills/wps-excel" ~/.claude/skills/wps-excel
ln -sf "$(pwd)/skills/wps-word" ~/.claude/skills/wps-word
ln -sf "$(pwd)/skills/wps-ppt" ~/.claude/skills/wps-ppt
ln -sf "$(pwd)/skills/wps-office" ~/.claude/skills/wps-office
```

### Bước 6: Xác thực cài đặt

```bash
# Xác thực MCP Server có thể khởi động bình thường
node wps-office-mcp/dist/index.js &
# Bạn sẽ thấy dòng log "MCP Server started successfully"
kill %1 2>/dev/null
```

## Kiến trúc

```
Tầng Skills (hướng dẫn bằng ngôn ngữ tự nhiên trong SKILL.md)
  ↓ Claude Code gọi
Tầng MCP Server (239 công cụ)
  ↓ wpsClient.executeMethod()
Tầng thực thi (Execution Layer)
  ├── macOS: wps-claude-assistant (227 actions, HTTP polling)
  └── Windows: wps-com.ps1 (231 actions, giao diện COM)
```

## Danh mục công cụ

| Ứng dụng | Số công cụ | Khả năng chính |
|----------|------------|----------------|
| Excel | 82 | Công thức / Xử lý dữ liệu / Biểu đồ / Pivot Table / Quản lý sheet / Định dạng / Workbook / Hàng & cột / Ghi chú & bảo vệ / Xuất ảnh |
| Word | 28 | Định dạng / Nội dung / Quản lý tài liệu / Header & Footer / Chú thích / Điền biểu mẫu theo mẫu / Cấu trúc đoạn văn |
| PPT | 112 | Slide / Hình dạng (Shape) / Hình ảnh / Bảng / Làm đẹp & bố cục / Hiệu ứng hoạt họa (Animation) / Biểu đồ / 3D / Trực quan hóa dữ liệu / Xuất ảnh |
| Dùng chung (Common) | 9 | Lưu file / Kiểm tra kết nối / Chọn văn bản / Chuyển đổi định dạng |
| Tích hợp sẵn (Built-in) | 12 | Kiểm tra kết nối / Gọi hàm vạn năng (universal method call) / Bộ nhớ đệm dữ liệu (Cache) |

## Xử lý sự cố (Troubleshooting)

| Sự cố | Cách khắc phục |
|-------|----------------|
| Kết nối MCP thất bại | Đảm bảo đã chạy `npm install && npm run build`, kiểm tra file `dist/index.js` có tồn tại. |
| WPS không phản hồi | Khởi động lại WPS Office, kiểm tra add-on đã được cài đặt thành công chưa. |
| Lỗi "arguments error" | Chạy lại script cài đặt và khởi động lại WPS. |
| Không tìm thấy plugin trên Linux | Xem hướng dẫn chi tiết dành riêng cho Linux trong [INSTALL_vi.md](INSTALL_vi.md). |
| Gọi tool trả về null | Đảm bảo đã mở đúng loại tài liệu tương ứng trong WPS Office (Excel/Word/PPT). |

## Giấy phép (License)

MIT
