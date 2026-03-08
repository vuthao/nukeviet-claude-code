# /add-func — Thêm Func Mới Vào Module NukeViet

**Yêu cầu:** $ARGUMENTS  
_(vd: `modules/contact/admin/support.php` hoặc `modules/news/funcs/tag.php`)_

---

## Quy trình

**Bước 0 — Plan Mode trước, không code ngay:**
- Đọc toàn bộ cấu trúc module hiện tại
- Xác định loại func (frontend `funcs/` hay admin `admin/`)
- Liệt kê các file cần tạo + file cần cập nhật
- Trình bày plan → chờ dev duyệt → mới bắt đầu

---

## Các bước thực hiện

### 1. Đọc context module hiện tại

```bash
# Xem cấu trúc module
find modules/[ten-module]/ -type f | sort

# Đọc các file quan trọng
cat modules/[ten-module]/version.php
cat modules/[ten-module]/admin.menu.php      # nếu là func admin
cat modules/[ten-module]/functions.php
cat modules/[ten-module]/language/admin_vi.php  # nếu là func admin
cat modules/[ten-module]/language/vi.php        # nếu là func frontend
```

### 2. Xác định loại func

| Loại | Đặt ở | Hằng kiểm tra | Truy cập |
|---|---|---|---|
| Frontend | `funcs/ten-func.php` | `NV_IS_MOD_*` | `?nv=module&op=ten-func` |
| Admin | `admin/ten-func.php` | `NV_IS_FILE_ADMIN` | Admin panel |

### 3. Cập nhật các file liên quan

**Nếu là func admin** — cập nhật `admin.menu.php`:
```php
$submenu['ten-func'] = $lang_module['menu_ten_func'];
$allow_func[] = 'ten-func';
```

**Nếu là func frontend** — cập nhật `version.php`:
```php
'modfuncs' => 'main,ten-func',   // thêm tên func vào đây
```

### 4. Tạo file func mới

**Template func admin** (`admin/ten-func.php`):
```php
<?php
if (!defined('NV_IS_FILE_ADMIN')) {
    exit('Stop!!!');
}

// Lấy input
$id = $nv_Request->get_int('id', 'get', 0);

// Xử lý POST
if ($nv_Request->isset_request('submit', 'post')) {
    $title = $nv_Request->get_title('title', 'post', '');
    // ... xử lý ...
}

// Lấy dữ liệu hiển thị
$sql  = 'SELECT * FROM ' . NV_PREFIXLANG . '_' . $module_data . '_items WHERE status = 1';
$rows = $db_slave->query($sql)->fetchAll();

// Render
$xtpl = new XTemplate('ten-func.tpl', NV_ROOTDIR . '/themes/' . $global_config['module_theme'] . '/modules/' . $module_file);
$xtpl->assign('LANG', $lang_module);
$xtpl->assign('ROWS', $rows);
$xtpl->parse('main');
$contents = $xtpl->text('main');

include NV_ROOTDIR . '/includes/header.php';
echo nv_admin_theme($contents);
include NV_ROOTDIR . '/includes/footer.php';
```

**Template func frontend** (`funcs/ten-func.php`):
```php
<?php
if (!defined('NV_IS_MOD_TENMODULE')) {
    exit('Stop!!!');
}

// Lấy input
$id = $nv_Request->get_int('id', 'get', 0);

// Lấy dữ liệu
$sql = 'SELECT * FROM ' . NV_PREFIXLANG . '_' . $module_data . '_items WHERE id = ' . $id . ' LIMIT 1';
$row = $db_slave->query($sql)->fetch();

if (empty($row)) {
    nv_redirect_location($page_url);
}

// Render
$contents = nv_tenmodule_tenfunc($row);

include NV_ROOTDIR . '/includes/header.php';
echo nv_site_theme($contents);
include NV_ROOTDIR . '/includes/footer.php';
```

### 5. Thêm language key

**`language/admin_vi.php`** (func admin):
```php
$lang_module['menu_ten_func'] = 'Tên hiển thị menu';
$lang_module['ten_func_title'] = 'Tiêu đề trang';
```

**`language/vi.php`** (func frontend):
```php
$lang_module['ten_func_title'] = 'Tiêu đề trang';
```

> Thêm key tương ứng vào `admin_en.php` / `en.php`

### 6. Kiểm tra cuối

```bash
php -l modules/[ten-module]/admin/ten-func.php
phpcs --standard=PSR2 modules/[ten-module]/admin/ten-func.php
```

- [ ] File func mới có kiểm tra hằng đầu file
- [ ] `admin.menu.php` đã cập nhật `$submenu` + `$allow_func` (func admin)
- [ ] `version.php` đã cập nhật `modfuncs` (func frontend)
- [ ] Language key đã thêm đủ vi + en
- [ ] Input qua `$nv_Request`, output qua `nv_htmlspecialchars()`
- [ ] SELECT dùng `$db_slave`, INSERT/UPDATE dùng `$db`
