# Tạo Module NukeViet 4.5 Mới

Tạo đầy đủ cấu trúc module NukeViet 4.5 cho: $ARGUMENTS

## Các bước thực hiện

1. **Phân tích yêu cầu**
   - Xác định tên module (snake_case)
   - Xác định các chức năng cần có (CRUD, danh sách, form...)
   - Xác định cấu trúc database cần thiết

2. **Đọc module mẫu hiện có**
   - Đọc 1-2 module trong `modules/` để hiểu pattern của dự án
   - Áp dụng đúng convention đang dùng

3. **Tạo đầy đủ các file**
   ```
   modules/[ten_module]/
   ├── module.php
   ├── funcs/main.php
   ├── admin/
   │   ├── index.php
   │   └── funcs/main.php
   ├── language/
   │   ├── vi.php
   │   └── en.php
   ├── templates/main.tpl
   └── sql/
       ├── install.sql
       └── uninstall.sql
   ```

4. **Kiểm tra bảo mật trước khi hoàn thành**
   - Mọi output qua `nv_htmlspecialchars()`
   - Mọi POST có `nv_check_formtoken()`
   - SQL dùng `$db->dbescape()` hoặc cast type
   - Có kiểm tra quyền trong admin

5. **Tóm tắt những gì đã tạo**
   - Liệt kê file đã tạo
   - Hướng dẫn cài đặt module
   - Lệnh chạy SQL install
