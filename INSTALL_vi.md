# Hướng dẫn cài đặt WPS Skills

[English](INSTALL.md) | [Tiếng Việt](INSTALL_vi.md) | [中文](INSTALL_zh.md)

> **Tài liệu này dành cho trợ lý lập trình AI** (Claude Code, Cursor, Codex CLI, v.v.) cũng như người dùng. Sau khi đọc tài liệu này, bạn có thể nắm bắt toàn bộ quy trình cài đặt và thực hiện tuần tự từng bước mà không bỏ sót bước nào.

---

## Kiểm tra trạng thái trước khi cài đặt

Trước khi tiến hành cài đặt, hãy xác định xem hệ thống đã từng được cài đặt hay chưa bằng cách kiểm tra lần lượt:

```bash
# 1. Kiểm tra MCP Server đã đăng ký chưa
claude mcp list 2>/dev/null | grep -i wps

# 2. Kiểm tra Skills đã đăng ký chưa
ls ~/.claude/skills/wps-excel/SKILL.md 2>/dev/null

# 3. Kiểm tra file build dist có tồn tại không
ls wps-office-mcp/dist/index.js 2>/dev/null
```

**Logic đánh giá**:
- Cả 3 mục đều tồn tại → Đã cài đặt hoàn chỉnh, không cần cài lại. Thông báo cho người dùng: "WPS Skills đã được cài đặt sẵn. Hãy khởi động lại Claude Code và WPS Office để sử dụng."
- Tồn tại một phần → Tiếp tục cài đặt bổ sung từ bước còn thiếu.
- Không tồn tại mục nào → Thực hiện quy trình cài đặt đầy đủ từ đầu.

---

## Cài đặt trên macOS

### Bước 1: Kiểm tra điều kiện tiên quyết

```bash
# Kiểm tra hệ điều hành (bắt buộc phải là Darwin)
uname -s

# Kiểm tra WPS Office đã được cài đặt chưa
ls /Applications/wpsoffice.app

# Kiểm tra phiên bản Node.js (phải >= 18.0.0)
node -v
```

**Nếu không đáp ứng điều kiện**:
- Chưa có WPS Office → Hướng dẫn người dùng cài đặt: https://www.wps.com/
- Chưa có Node.js hoặc phiên bản < 18 → Hướng dẫn cài đặt: https://nodejs.org/
- Dừng quy trình cài đặt lại nếu điều kiện tiên quyết chưa đạt.

### Bước 2: Chạy script cài đặt tự động 1 chạm

```bash
# Chuyển vào thư mục gốc của dự án
cd <THU_MUC_DU_AN>

# Cấp quyền thực thi và chạy script
chmod +x scripts/auto-install-mac.sh
./scripts/auto-install-mac.sh
```

Script sẽ tự động thực hiện toàn bộ các bước sau:
- Kiểm tra lại điều kiện tiên quyết (macOS, WPS Office, Node.js 18+)
- Tạo thư mục WPS add-on (nếu chưa có)
- Cài đặt các file add-on vào `~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/` (dấu gạch dưới `_` ở cuối tên thư mục là bắt buộc)
- Cập nhật file `publish.xml` để đăng ký add-on
- Chạy `npm install` để cài đặt dependencies
- Chạy `npm run build` để biên dịch TypeScript
- Chạy lệnh `claude mcp add wps-office` để đăng ký MCP Server
- Tạo symlink đăng ký Skills vào thư mục `~/.claude/skills/`

### Bước 3: Xác thực sau khi cài đặt

Thực hiện kiểm tra từng mục sau, tất cả phải thành công:

```bash
# 1. MCP Server đã được đăng ký (kết quả hiển thị wps-office)
claude mcp list

# 2. Skills đã được đăng ký (hiển thị đủ 4 thư mục/link: wps-excel, wps-word, wps-ppt, wps-office)
ls ~/.claude/skills/

# 3. File biên dịch đã tồn tại
ls wps-office-mcp/dist/index.js

# 4. Add-on WPS đã cài đặt (thư mục có đuôi _)
ls ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/

# 5. publish.xml đã ghi nhận add-on
grep "claude-assistant" ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/publish.xml
```

### Bước 4: Hướng dẫn người dùng

Sau khi hoàn tất cài đặt, hãy nhắc người dùng:
1. **Khởi động lại Claude Code** (Bắt buộc! Skills chỉ được nạp khi khởi động)
2. Khởi động lại WPS Office
3. Mở một tài liệu Excel/Word/PPT bất kỳ
4. Kiểm tra tab "Claude助手" (Claude Assistant) trên thanh công cụ WPS và xác nhận trạng thái hiển thị "轮询中" (Polling)

### ⚠️ Lưu ý đã biết

**Hộp thoại xin quyền truy cập thư mục Sandbox trên macOS**:
- Hiện tượng: Khi lần đầu chạy lệnh xác thực truy cập `~/Library/Containers/com.kingsoft.wpsoffice.mac/...`, macOS sẽ hiện popup hỏi quyền truy cập file.
- Xử lý: Chỉ cần nhấn "Cho phép" (Allow), hộp thoại sẽ không xuất hiện lại nữa.
- Ảnh hưởng: Cơ chế bảo mật Sandbox bình thường của macOS, không ảnh hưởng cài đặt.

---

## Cài đặt trên Linux

### Bước 1: Kiểm tra điều kiện tiên quyết

```bash
# Kiểm tra hệ điều hành (phải là Linux)
uname -s

# Kiểm tra WPS Office
which wps || ls /opt/kingsoft/wps-office

# Kiểm tra phiên bản Node.js (phải >= 18.0.0)
node -v
```

**Nếu không đáp ứng điều kiện**:
- Chưa có WPS Office -> Hướng dẫn cài đặt: https://linux.wps.com
- Chưa có Node.js hoặc version < 18 -> Hướng dẫn cài đặt: https://nodejs.org/

### Bước 2: Cài đặt thủ công

```bash
# Chuyển vào thư mục gốc của dự án
cd <THU_MUC_DU_AN>

# Cài đặt dependencies và build
cd wps-office-mcp
npm install
rm -rf dist
npm run build
cd ..

# Sao chép add-on vào thư mục WPS (tên thư mục bắt buộc có dấu _ ở đuôi)
mkdir -p ~/.local/share/Kingsoft/wps/jsaddons
cp -R wps-claude-assistant ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_

# Tạo publish.xml
# Lưu ý: enable="enable_dev" là chế độ phát triển (mặc định); nếu không nạp được có thể đổi thành enable="true"
cat > ~/.local/share/Kingsoft/wps/jsaddons/publish.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<jsplugins>
  <jsplugin name="claude-assistant" type="wps,et,wpp" url="claude-assistant_/" enable="enable_dev"/>
</jsplugins>
EOF

# Đăng ký MCP Server
claude mcp add wps-office node $(pwd)/wps-office-mcp/dist/index.js

# Đăng ký Skills
mkdir -p ~/.claude/skills
ln -sf $(pwd)/skills/wps-excel ~/.claude/skills/wps-excel
ln -sf $(pwd)/skills/wps-word ~/.claude/skills/wps-word
ln -sf $(pwd)/skills/wps-ppt ~/.claude/skills/wps-ppt
ln -sf $(pwd)/skills/wps-office ~/.claude/skills/wps-office
```

### Bước 3: Xác thực sau khi cài đặt

```bash
# 1. MCP Server đã đăng ký
claude mcp list

# 2. Skills đã đăng ký
ls ~/.claude/skills/

# 3. File biên dịch đã tồn tại
ls wps-office-mcp/dist/index.js

# 4. Add-on WPS đã cài đặt (thư mục đuôi _)
ls ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_/

# 5. publish.xml đã ghi nhận
grep "claude-assistant" ~/.local/share/Kingsoft/wps/jsaddons/publish.xml
```

### Bước 4: Hướng dẫn người dùng

1. **Khởi động lại Claude Code** (Bắt buộc!)
2. Khởi động lại WPS Office
3. Mở tài liệu bất kỳ và xem tab "Claude助手" (Claude Assistant)

### ⚠️ Thứ tự khởi động trên Linux (Rất quan trọng)

Theo phản ánh từ Issue #17: Trên một số bản phân phối như Kylin Linux, nếu WPS đang chạy trước khi khởi động Claude Code (MCP Server), tiến trình WPS có thể bị thoát đột ngột.

**Thứ tự khởi động chuẩn**:
1. Khởi động **Claude Code** trước (để MCP Server sẵn sàng)
2. Sau đó mở **WPS Office** (add-on sẽ thăm dò HTTP tới cổng 58891 đã sẵn sàng)
3. Sử dụng các công cụ WPS MCP bình thường

Nếu WPS đang chạy, khuyến nghị chạy lệnh `pkill -9 wps && pkill -9 wpp && pkill -9 et` trước khi khởi động lại theo thứ tự trên.

### Bảng đường dẫn quan trọng trên Linux

| Hạng mục | Đường dẫn |
|----------|-----------|
| Thư mục gốc Add-on WPS | `~/.local/share/Kingsoft/wps/jsaddons/` |
| Thư mục cài đặt Add-on | `<Thư mục gốc>/claude-assistant_/` (bắt buộc có dấu `_`) |
| publish.xml | `<Thư mục gốc>/publish.xml` |

---

## Cài đặt trên Windows

### Bước 1: Kiểm tra điều kiện tiên quyết

```powershell
# Kiểm tra thư mục add-on WPS có tồn tại không
Test-Path "$env:APPDATA\kingsoft\wps\jsaddons"

# Kiểm tra phiên bản Node.js (phải >= 18.0.0)
node -v
```

**Nếu không đáp ứng điều kiện**:
- Thư mục add-on WPS không tồn tại → Yêu cầu người dùng cài đặt WPS Office: https://www.wps.com/
- Chưa có Node.js hoặc phiên bản < 18 → Yêu cầu người dùng cài đặt: https://nodejs.org/
- Dừng quy trình cài đặt nếu điều kiện tiên quyết chưa đạt.

### Bước 2: Chạy script cài đặt tự động 1 chạm

```powershell
# Chuyển vào thư mục gốc của dự án
cd <THU_MUC_DU_AN>

# Chạy script PowerShell cài đặt
powershell -ExecutionPolicy Bypass -File scripts/auto-install.ps1
```

Script sẽ tự động hoàn tất:
- Kiểm tra phiên bản Node.js 18+
- Chạy `npm install` cài đặt dependencies
- Chạy `npm run build` biên dịch TypeScript
- Cấu hình Claude Code MCP (ghi vào `%USERPROFILE%\.claude\settings.json`)
- Sao chép Skills vào `%USERPROFILE%\.claude\skills\`
- Cài đặt Add-on WPS vào `%APPDATA%\kingsoft\wps\jsaddons\wps-claude-addon_\` (đuôi `_` là bắt buộc)
- Cập nhật `publish.xml` để đăng ký add-on

### Bước 3: Xác thực sau cài đặt

```powershell
# 1. MCP Server đã đăng ký
claude mcp list

# 2. Skills đã được copy (hiển thị wps-excel, wps-word, wps-ppt, wps-office)
Get-ChildItem "$env:USERPROFILE\.claude\skills"

# 3. File biên dịch tồn tại
Test-Path "wps-office-mcp\dist\index.js"

# 4. Thư mục add-on WPS đã tạo
Test-Path "$env:APPDATA\kingsoft\wps\jsaddons\wps-claude-addon_"

# 5. publish.xml đã đăng ký
Select-String -Path "$env:APPDATA\kingsoft\wps\jsaddons\publish.xml" -Pattern "wps-claude-addon"
```

### Bước 4: Hướng dẫn người dùng

Sau khi hoàn tất, hãy nhắc người dùng:
1. **Khởi động lại Claude Code** (Bắt buộc!)
2. Khởi động lại WPS Office
3. Kiểm tra tab "Claude助手" (Claude Assistant) trên WPS

---

## Bảng tra cứu đường dẫn chính

### macOS

| Hạng mục | Đường dẫn |
|----------|-----------|
| Thư mục gốc Add-on WPS | `~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/` |
| Thư mục cài đặt Add-on | `<Thư mục gốc>/claude-assistant_/` (bắt buộc đuôi `_`) |
| publish.xml | `<Thư mục gốc>/publish.xml` |
| Thư mục đăng ký Skills | `~/.claude/skills/` (4 symlinks) |
| File chạy MCP Server | `<THU_MUC_DU_AN>/wps-office-mcp/dist/index.js` |
| Cổng HTTP Polling | `58891` |

### Windows

| Hạng mục | Đường dẫn |
|----------|-----------|
| Thư mục gốc Add-on WPS | `%APPDATA%\kingsoft\wps\jsaddons\` |
| Thư mục cài đặt Add-on | `<Thư mục gốc>\wps-claude-addon_\` (bắt buộc đuôi `_`) |
| publish.xml | `<Thư mục gốc>\publish.xml` |
| Thư mục đăng ký Skills | `%USERPROFILE%\.claude\skills\` (sao chép trực tiếp, không dùng symlink) |
| Cấu hình MCP Server | `%USERPROFILE%\.claude\settings.json` |

---

## Xử lý sự cố khi cài đặt

Tra cứu bảng sau nếu gặp lỗi:

### npm install thất bại

```bash
# Xóa cache và thử lại
cd wps-office-mcp
rm -rf node_modules package-lock.json
npm install
```

Nếu vẫn lỗi, kiểm tra phiên bản Node.js:
```bash
node -v
# Phải >= 18.0.0, nếu thấp hơn cần nâng cấp Node.js
```

### npm run build (biên dịch TypeScript) thất bại

```bash
cd wps-office-mcp
rm -rf dist node_modules
npm install
npm run build
```

Nếu báo lỗi `tsc: command not found`, kiểm tra lại `package.json` xem typescript đã có trong devDependencies chưa.

### Đăng ký MCP Server thất bại

Đăng ký thủ công bằng lệnh:
```bash
claude mcp add wps-office node <DUONG_DAN_TUYET_DOI>/wps-office-mcp/dist/index.js
```

Lưu ý: `<DUONG_DAN_TUYET_DOI>` phải là đường dẫn tuyệt đối đầy đủ, không dùng đường dẫn tương đối hay biến môi trường chưa giải phóng.

### Không tạo được symlink cho Skills

```bash
PROJECT_DIR=<DUONG_DAN_TUYET_DOI>
mkdir -p ~/.claude/skills
ln -sf "$PROJECT_DIR/skills/wps-excel" ~/.claude/skills/wps-excel
ln -sf "$PROJECT_DIR/skills/wps-word" ~/.claude/skills/wps-word
ln -sf "$PROJECT_DIR/skills/wps-ppt" ~/.claude/skills/wps-ppt
ln -sf "$PROJECT_DIR/skills/wps-office" ~/.claude/skills/wps-office
```

Kiểm tra:
```bash
ls -la ~/.claude/skills/
# Cần thấy 4 symlinks trỏ tới các thư mục con trong skills/
```

### Add-on WPS không hiển thị tab "Claude助手" (Claude Assistant)

1. Kiểm tra thư mục add-on đã được sao chép đúng và có đuôi `_` chưa:
```bash
# macOS
ls ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_/
# Cần có main.js, manifest.xml, ribbon.xml, v.v.
```

2. Kiểm tra `publish.xml` có entry đăng ký:
```bash
# macOS
cat ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/publish.xml
# Cần chứa thẻ <jsplugin name="claude-assistant" .../>
```

3. Buộc đóng và mở lại WPS:
```bash
# macOS
pkill -f wpsoffice
sleep 2
open /Applications/wpsoffice.app
```

### Cổng HTTP Polling 58891 bị chiếm dụng (macOS)

```bash
# Tìm tiến trình đang chiếm cổng
lsof -i :58891

# Buộc dừng tiến trình
kill <PID>
```

### Quyền truy cập thư mục Add-on trên macOS bị từ chối

```bash
# Tạo thư mục thủ công
mkdir -p ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons

# Sửa phân quyền
chmod -R 755 ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons
```

---

## Các vấn đề thường gặp trên GitHub Issues

### Issue #6: Lỗi kết nối MCP ("Failed to connect")

**Hiện tượng**: `claude mcp list` báo `wps-office: Failed to connect`.

**Nguyên nhân**: File build cũ trong thư mục `dist` bị lỗi thời hoặc có tên tool bị trùng lặp khiến MCP Server crash ngay khi khởi động.

**Cách khắc phục**:
```bash
cd wps-office-mcp
# Xóa thư mục dist cũ và build lại (bước quan trọng!)
rm -rf dist
npm run build
# Kiểm tra khởi động (phải thấy "Server started successfully")
node dist/index.js 2>&1 | head -5
# Nhấn Ctrl+C để thoát
```

Nếu vẫn lỗi, hãy dọn dẹp toàn bộ và cài đặt lại:
```bash
cd wps-office-mcp
rm -rf dist node_modules
npm install
npm run build
```

Sau đó đăng ký lại MCP:
```bash
claude mcp remove wps-office
claude mcp add wps-office node <DUONG_DAN_TUYET_DOI>/wps-office-mcp/dist/index.js
```

### Issue #5: WPS trên Linux không tìm thấy Add-on

**Hiện tượng**: Cài đặt trên Linux (như Arch Linux) thành công nhưng không thấy tab Claude trong WPS.

**Nguyên nhân**: Script cài đặt thiếu dấu `_` ở đuôi thư mục add-on trên Linux và chưa cập nhật `publish.xml`. Quy định của WPS jsaddons yêu cầu tên thư mục phải kết thúc bằng `_` mới nhận diện được.

**Cách khắc phục**:
```bash
# 1. Xóa thư mục cài đặt cũ bị sai
rm -rf ~/.local/share/Kingsoft/wps/jsaddons/wps-claude-addon

# 2. Sao chép lại vào đường dẫn chuẩn (bắt buộc đuôi _)
cp -R <THU_MUC_DU_AN>/wps-claude-assistant ~/.local/share/Kingsoft/wps/jsaddons/claude-assistant_

# 3. Tạo file publish.xml
cat > ~/.local/share/Kingsoft/wps/jsaddons/publish.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<jsplugins>
  <jsplugin name="claude-assistant" type="wps,et,wpp" url="claude-assistant_/" enable="enable_dev"/>
</jsplugins>
EOF

# 4. Khởi động lại WPS
pkill -f wps
# Mở lại WPS Office
```

**Lưu ý**: Một số bản phân phối Linux có thể dùng đường dẫn `~/.kingsoft/wps/jsaddons/`. Nếu không chắc chắn, bạn có thể tìm bằng lệnh:
```bash
find / -path "*/Kingsoft/wps/jsaddons" -type d 2>/dev/null
find / -path "*kingsoft/wps/jsaddons" -type d 2>/dev/null
```

### Issue #4: Add-on WPS báo lỗi "arguments error" khi khởi động

**Hiện tượng**: WPS hiển thị popup `ERROR: arguments error at <anonymous>:1:89`.

**Nguyên nhân**: File `manifest.xml` thiếu khai báo thẻ `<ribbon>` và `<scripts>` khiến WPS không giải quyết được entrypoint của plugin.

**Cách khắc phục**:

File `wps-claude-assistant/manifest.xml` đã được sửa hoàn chỉnh trong bản mới. Nếu vẫn gặp:

```bash
# 1. Kiểm tra manifest.xml đã có khai báo ribbon và scripts chưa
grep -E "ribbon|scripts" <THU_MUC_CAI_DAT_ADDON>/manifest.xml

# 2. Xác nhận file ribbon.xml và main.js tồn tại
ls <THU_MUC_CAI_DAT_ADDON>/ribbon.xml
ls <THU_MUC_CAI_DAT_ADDON>/main.js

# 3. Sao chép đè lại thư mục add-on mới nhất
# macOS:
rm -rf ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_
cp -R <THU_MUC_DU_AN>/wps-claude-assistant ~/Library/Containers/com.kingsoft.wpsoffice.mac/Data/.kingsoft/wps/jsaddons/claude-assistant_

# 4. Khởi động lại WPS
pkill -f wpsoffice
sleep 2
open /Applications/wpsoffice.app
```
