# Hướng Dẫn Module NukeViet 4.x

## Cấu trúc file bắt buộc

```
modules/ten-module/
├── version.php           # BẮT BUỘC — phiên bản dạng X.Y.ZZ (vd: 4.0.00)
├── functions.php         # BẮT BUỘC — không xóa dù rỗng; define NV_IS_MOD_*
├── admin.functions.php   # BẮT BUỘC
├── admin.menu.php        # BẮT BUỘC — $submenu, $allow_func
├── action_mysql.php      # $sql_create_module + $sql_drop_module
├── theme.php             # hàm giao diện ngoài site
├── funcs/main.php        # func mặc định ngoài site
├── admin/main.php        # func mặc định admin
└── language/vi.php · en.php · admin_vi.php · admin_en.php
```

Template `.tpl` → `themes/[theme]/modules/[module]/` — **KHÔNG** trong `modules/`

---

## Template code

### version.php
```php
<?php
if (!defined('NV_ADMIN') or !defined('NV_MAINFILE')) die('Stop!!!');
$module_version = array(
    'name'        => 'Tên module',
    'modfuncs'    => 'main',
    'is_sysmod'   => 0,
    'virtual'     => 1,
    'version'     => '4.0.00',
    'date'        => 'Mon, 1 Jan 2025 00:00:00 GMT',
    'author'      => 'Tác giả',
    'note'        => '',
    'uploads_dir' => array($module_name),
);
```

### functions.php
```php
<?php
if (!defined('NV_SYSTEM')) die('Stop!!!');
define('NV_IS_MOD_TENMODULE', true); // tên hằng riêng cho từng module
```

### admin.functions.php
```php
<?php
if (!defined('NV_ADMIN') or !defined('NV_MAINFILE') or !defined('NV_IS_MODADMIN')) die('Stop!!!');
define('NV_IS_TENMODULE_ADMIN', true);
```

### admin.menu.php
```php
<?php
if (!defined('NV_ADMIN')) die('Stop!!!');
$submenu['main']    = $lang_module['menu_main'];
$submenu['setting'] = $lang_module['menu_setting'];
$allow_func[] = 'edit';
$allow_func[] = 'delete';
$allow_func[] = 'setting';
```

### action_mysql.php
```php
<?php
if (!defined('NV_IS_FILE_MODULES')) die('Stop!!!');
$sql_drop_module   = array();
$sql_drop_module[] = 'DROP TABLE IF EXISTS '
    . $db_config['prefix'] . '_' . $lang . '_' . $module_data . '_items';
$sql_create_module   = $sql_drop_module;
$sql_create_module[] = 'CREATE TABLE '
    . $db_config['prefix'] . '_' . $lang . '_' . $module_data . '_items ('
    . ' `id` MEDIUMINT(8) UNSIGNED NOT NULL AUTO_INCREMENT,'
    . ' `title` VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
    . ' `content` MEDIUMTEXT CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
    . ' `status` TINYINT(1) NOT NULL DEFAULT \'1\','
    . ' `created_at` INT(11) UNSIGNED NOT NULL DEFAULT \'0\','
    . ' PRIMARY KEY (`id`), KEY `idx_status` (`status`)'
    . ') ENGINE=MyISAM DEFAULT CHARSET=utf8';
```

### funcs/main.php
```php
<?php
if (!defined('NV_IS_MOD_TENMODULE')) die('Stop!!!');
$contents = nv_tenmodule_main($data);
include(NV_ROOTDIR . '/includes/header.php');
echo nv_site_theme($contents);
include(NV_ROOTDIR . '/includes/footer.php');
```

### theme.php
```php
<?php
if (!defined('NV_IS_MOD_TENMODULE')) die('Stop!!!');
/**
 * Hiển thị trang chính
 * @param array $data
 * @return string
 */
function nv_tenmodule_main($data) {
    global $module_file, $lang_module, $lang_global, $module_info;
    $tpl = file_exists(NV_ROOTDIR . '/themes/' . $module_info['template'] . '/modules/' . $module_file . '/main.tpl')
        ? $module_info['template'] : 'default';
    $xtpl = new XTemplate('main.tpl', NV_ROOTDIR . '/themes/' . $tpl . '/modules/' . $module_file);
    $xtpl->assign('LANG', $lang_module);
    $xtpl->assign('GLANG', $lang_global);
    $xtpl->parse('main');
    return $xtpl->text('main');
}
```

### admin/main.php
```php
<?php
if (!defined('NV_IS_FILE_ADMIN')) die('Stop!!!');
$xtpl = new XTemplate('main.tpl', NV_ROOTDIR . '/themes/' . $global_config['module_theme'] . '/modules/' . $module_file);
$xtpl->assign('LANG', $lang_module);
$xtpl->parse('main');
$contents = $xtpl->text('main');
include(NV_ROOTDIR . '/includes/header.php');
echo nv_admin_theme($contents);
include(NV_ROOTDIR . '/includes/footer.php');
```

### Lấy input — PHẢI qua $nv_Request
```php
$id    = $nv_Request->get_int('id', 'get', 0);
$page  = $nv_Request->get_int('page', 'get', 1);
$title = $nv_Request->get_title('title', 'post', '');
$title = nv_substr($title, 0, 255);
$body  = $nv_Request->get_editor('body', '', NV_ALLOWED_HTML_TAGS);
$desc  = $nv_Request->get_textarea('desc', '', NV_ALLOWED_HTML_TAGS);
// Đưa lại vào editor/textarea sau khi lấy từ DB:
$body  = nv_htmlspecialchars(nv_editor_br2nl($row['body']));
$desc  = nv_htmlspecialchars(nv_br2nl($row['description']));
```

---

## Checklist tạo module mới

- [ ] 4 file bắt buộc có đủ
- [ ] `functions.php` tồn tại — không xóa dù rỗng
- [ ] Phiên bản dạng `X.Y.ZZ`
- [ ] Template `.tpl` đặt trong `themes/` không phải `modules/`
- [ ] Language có đủ `vi.php` và `admin_vi.php`
- [ ] `action_mysql.php` có cả `$sql_drop_module` và `$sql_create_module`
- [ ] Input qua `$nv_Request`, output qua `nv_htmlspecialchars()`
