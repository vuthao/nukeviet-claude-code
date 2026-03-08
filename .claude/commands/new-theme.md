# Tạo / Chỉnh Sửa Theme NukeViet 4.x

Thực hiện tác vụ liên quan đến giao diện NukeViet: $ARGUMENTS

## Xác định loại tác vụ

Đọc yêu cầu và xác định một trong các trường hợp sau:

### A) Tạo theme mới
1. Copy `themes/default` → `themes/[ten-theme-moi]`
2. Dọn dẹp file thừa theo quy trình:
   - `blocks/` → xóa hết trừ `index.html`
   - `css/` → giữ: `admin.css, bootstrap*.css, custom.css, style*.css`
   - `images/` → giữ: thư mục `icons/`, `index.html`, `no_image.gif`
   - `js/` → giữ: `bootstrap.min.js, custom.js, main.js`
   - `layout/` → giữ `block.default.tpl`, xóa template/layout không dùng
   - `modules/` → xóa hết (copy lại khi cần chỉnh sửa từng module)
3. Cập nhật `config.ini` — tên, tác giả, layout mặc định, block positions
4. Xóa cache sau khi xong

### B) Thêm block position mới
1. Khai báo `<position>` trong `themes/[theme]/config.ini`
   - `<n>` = tên hiển thị
   - `<tag>` = `[TEN_KHOI]` — in hoa, chữ/số/gạch dưới
2. Đặt `[TEN_KHOI]` vào file tpl layout tương ứng
3. Nhắc xóa cache: Quản trị → Công cụ web → Dọn dẹp → Làm sạch cache

### C) Tạo/chỉnh sửa layout
1. Tạo file `themes/[theme]/layout/layout.[ten].tpl`
2. Dùng Bootstrap 24 cột (không phải 12 cột chuẩn)
3. Đặt tag `[POSITION_NAME]` đúng vị trí
4. Cập nhật `config.ini` nếu cần

### D) Tùy biến giao diện module
1. **Thử CSS trước** — tận dụng `custom.css` với class selector có sẵn
2. Chỉ copy tpl khi CSS không đủ:
   - Copy `themes/default/modules/[ten-module]/[file].tpl`
   - Paste vào `themes/[theme]/modules/[ten-module]/`
   - Chỉnh sửa tpl
3. Không copy nếu chỉ thay đổi màu sắc/khoảng cách

## Lưu ý quan trọng
- `block.default.tpl` — KHÔNG BAO GIỜ XÓA
- CSS tùy chỉnh → `custom.css` | JS tùy chỉnh → `custom.js`
- Giao diện dùng Bootstrap **24 cột** (khác 12 cột của Bootstrap chuẩn)
- Sau mọi thay đổi config.ini → **xóa cache**
