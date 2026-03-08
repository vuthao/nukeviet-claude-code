# NukeViet 4.x — AI Context

> File này là **nguồn sự thật duy nhất** cho tất cả AI tool.  
> Khi cần cập nhật convention → chỉ sửa file này.

## Stack
PHP 8.1+, MySQL 8.0, NukeViet 4.x · Ubuntu 22.04, Nginx+PHP-FPM  
GitLab: `main` (prod) | `develop` (staging)

---

## Bảo mật — KHÔNG được vi phạm

| Tình huống | Đúng |
|---|---|
| Lấy input | `$nv_Request->get_int/get_title/get_editor()` — không `$_GET/$_POST` trực tiếp |
| SQL | PDO `prepare()` + `bindParam()` cho chuỗi từ user — số nguyên dùng `(int)` trực tiếp |
| HTML output | `nv_htmlspecialchars()` |
| Kiểm tra file | `nv_is_file()` — không `is_file()` với path từ user |
| Redirect | `$page_url` hoặc `nv_redirect_encrypt()` — không `$client_info['selfurl']` trực tiếp |
| Admin ghi | Kiểm tra `defined('NV_IS_ADMIN')` trước |

---

## Conventions

```
Bảng đa ngôn ngữ : NV_PREFIXLANG  . '_ten_bang'  → nv4_vi_news
Bảng dùng chung  : NV_TABLEPREFIX . '_ten_bang'  → nv4_users
Thời gian        : NV_CURRENTTIME
Đường dẫn gốc   : NV_ROOTDIR
URL              : ?lang=vi&nv=ten-module&op=ten-func
```

**Kiểm tra hằng bắt buộc đầu mỗi file:**
```
version.php          → NV_ADMIN + NV_MAINFILE
functions.php        → NV_SYSTEM
admin.functions.php  → NV_ADMIN + NV_MAINFILE + NV_IS_MODADMIN
admin.menu.php       → NV_ADMIN
action_mysql.php     → NV_IS_FILE_MODULES
funcs/main.php       → NV_IS_MOD_TENMODULE  (define trong functions.php)
admin/main.php       → NV_IS_FILE_ADMIN
```

**Code style:** 4 spaces · camelCase biến/hàm · PascalCase class · PHPDoc + comment tiếng Việt

---

## Cấu trúc Module

```
modules/ten-module/
├── version.php           # BẮT BUỘC — phiên bản dạng X.Y.ZZ (vd: 4.0.00)
├── functions.php         # BẮT BUỘC — không xóa dù rỗng; define NV_IS_MOD_*
├── admin.functions.php
├── admin.menu.php        # $submenu, $allow_func
├── action_mysql.php      # $sql_create_module + $sql_drop_module
├── theme.php             # hàm giao diện ngoài site
├── funcs/main.php        # func mặc định ngoài site
├── admin/main.php        # func mặc định admin
└── language/vi.php · en.php · admin_vi.php · admin_en.php
```
Template `.tpl` → `themes/[theme]/modules/[module]/` — **KHÔNG** trong `modules/`

---

## Cấu trúc Theme

```
themes/ten-theme/
├── config.ini            # tên, layoutdefault, <positions>
├── theme.php
├── default.jpg           # 800×600px
├── css/custom.css        # ← CSS tùy chỉnh viết vào đây
├── js/custom.js          # ← JS tùy chỉnh viết vào đây
├── layout/block.default.tpl  # BẮT BUỘC — không xóa
└── modules/ten-module/   # override tpl — chỉ copy khi thực sự cần sửa
```

**Layout grid 24 cột** (NukeViet mở rộng từ Bootstrap v3.3, tăng từ 12 lên 24 cột): `main`=24 | `main-right`=18-6 | `left-main`=6-18 | `left-main-right`=5-13-6  
**Block position:** khai báo `<position><tag>[TEN_KHOI]</tag></position>` trong `config.ini`  
**Sau thay đổi config.ini:** Admin → Công cụ web → Làm sạch cache

---

## MySQL Patterns

```php
$db->query($sql)->fetch()         // lấy 1 dòng
$db->query($sql)->fetchAll()      // lấy tất cả dòng
$db->query($sql)->fetchColumn()   // lấy ô đầu tiên (COUNT, MAX...)
$db->prepare($sql)                // chuẩn bị prepared statement cho user input
$db->lastInsertId()               // ID vừa INSERT
```

Số nguyên và hằng hệ thống nối thẳng vào SQL — chuỗi từ user input dùng `prepare()` + `bindParam()`.  
Chi tiết: xem `docs/mysql-guide.md`

---

## Nâng cấp Module

Tài liệu chi tiết: `docs/upgrade/module/`

**Lộ trình — đọc file theo đúng thứ tự:**

| Lộ trình | File cần đọc (theo thứ tự) |
|---|---|
| 4.4 → 4.5.07 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.4 → 4.5.02 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` |
| 4.5.00 → 4.5.07 | `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.5.06 → 4.5.07 | `NV-4.5.06-len-4.5.07.md` |

---

## Nâng cấp Theme

Tài liệu chi tiết: `docs/upgrade/theme/`  
Cùng bảng lộ trình và tên file như module — chỉ khác thư mục.

---

## Git
Không push thẳng `main`/`develop` · Commit: `feat|fix|refactor|docs: mô tả [AI-assisted]` · MR cần 1 peer review
