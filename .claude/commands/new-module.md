# Tạo Module NukeViet 4.x Mới

Tạo đầy đủ cấu trúc module NukeViet 4.x cho: $ARGUMENTS

## Quy trình

**Bước 0 — Plan Mode trước, không code ngay:**
- Phân tích yêu cầu, đọc module mẫu liên quan trong `modules/`
- Lên danh sách file cần tạo, cấu trúc database, các func cần có
- Trình bày plan → chờ dev duyệt → mới bắt đầu tạo file

## Các bước thực hiện

1. **Phân tích yêu cầu**
   - Xác định tên module (chữ cái thường, số, dấu gạch ngang — không dùng gạch dưới)
   - Xác định các funcs cần có (main, danh sách, form, CRUD...)
   - Xác định cấu trúc database cần thiết

2. **Đọc module mẫu hiện có**
   - Đọc 1-2 module trong `modules/` để hiểu pattern của dự án
   - Áp dụng đúng convention đang dùng

3. **Tạo đầy đủ các file theo chuẩn NukeViet**
   ```
   modules/[ten_module]/
   ├── version.php             # BẮT BUỘC — khai báo tên, funcs, phiên bản
   ├── functions.php           # BẮT BUỘC — hàm ngoài site (không được xóa dù rỗng)
   ├── admin.functions.php     # BẮT BUỘC — hàm admin
   ├── admin.menu.php          # BẮT BUỘC — $submenu và $allow_func
   ├── action_mysql.php        # cài đặt CSDL ($sql_create_module, $sql_drop_module)
   ├── theme.php               # hàm giao diện ngoài site
   ├── comment.php             # tích hợp comment (không bắt buộc)
   ├── notification.php        # tích hợp notification (không bắt buộc)
   ├── search.php              # tích hợp tìm kiếm toàn site (không bắt buộc)
   ├── siteinfo.php            # thông tin module trong admin (không bắt buộc)
   ├── menu.php                # xuất thông tin cho module menu (không bắt buộc)
   ├── blocks/                 # các block của module
   ├── funcs/
   │   └── main.php            # func mặc định ngoài site
   ├── admin/
   │   └── main.php            # func mặc định trong admin
   └── language/
       ├── vi.php              # ngôn ngữ ngoài site tiếng Việt
       ├── en.php              # ngôn ngữ ngoài site tiếng Anh
       ├── admin_vi.php        # ngôn ngữ admin tiếng Việt
       └── admin_en.php        # ngôn ngữ admin tiếng Anh
   ```

   > **Lưu ý quan trọng:** Template `.tpl` đặt trong `themes/[theme]/modules/[ten_module]/`
   > KHÔNG đặt trong thư mục module.

4. **Kiểm tra bảo mật trước khi hoàn thành**
   - Mọi input lấy qua `$nv_Request->get_int()`, `get_title()`, `get_editor()`, v.v.
   - Mọi output HTML qua `nv_htmlspecialchars()`
   - SQL dùng `$db->prepare()` + `bindParam()` cho chuỗi từ user — số nguyên dùng `(int)` trực tiếp
   - Kiểm tra `defined('NV_IS_ADMIN')` trước thao tác ghi trong admin
   - Kiểm tra `defined('NV_IS_FILE_ADMIN')` trong `admin/main.php`

5. **Tóm tắt những gì đã tạo**
   - Liệt kê file đã tạo
   - Hướng dẫn cài đặt module qua khu vực quản trị
   - Lưu ý về file `action_mysql.php` (hệ thống tự chạy khi cài module)
