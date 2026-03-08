# Hướng Dẫn Module NukeViet 4.x

## Cấu trúc file bắt buộc

```
modules/ten-module/
├── version.php           # BẮT BUỘC — phiên bản dạng X.Y.ZZ (vd: 4.0.00)
├── functions.php         # BẮT BUỘC — không xóa dù rỗng; define NV_IS_MOD_*
├── admin.functions.php   # BẮT BUỘC
├── admin.menu.php        # BẮT BUỘC — $submenu (và $allow_func nếu module phức tạp)
├── action_mysql.php      # $sql_create_module + $sql_drop_module
├── global.functions.php  # tùy chọn — hàm dùng chung cả frontend lẫn admin
├── theme.php             # hàm giao diện ngoài site
├── funcs/main.php        # func mặc định ngoài site
├── admin/main.php        # func mặc định admin
├── Shared/               # PSR-4 classes (namespace NukeViet\Module\{name}\Shared\)
└── language/vi.php · en.php · admin_vi.php · admin_en.php
```

Template `.tpl` → `themes/[theme]/modules/[module]/` — **KHÔNG** trong `modules/`

---

## Template code

### version.php
```php
<?php
if (!defined('NV_ADMIN') or !defined('NV_MAINFILE')) {
    exit('Stop!!!');
}
$module_version = [
    'name'        => 'Tên module',
    'modfuncs'    => 'main',           // danh sách func, cách nhau bằng dấu phẩy
    'is_sysmod'   => 0,
    'virtual'     => 1,
    'version'     => '4.0.00',
    'date'        => 'Mon, 1 Jan 2025 00:00:00 GMT',
    'author'      => 'Tác giả',
    'note'        => '',
    'uploads_dir' => [$module_upload],  // dùng $module_upload, không phải $module_name

    // --- Các key tùy chọn (dùng khi cần) ---

    // Func hỗ trợ tái tạo alias — danh sách con của modfuncs
    // 'change_alias' => 'func1,func2',

    // Func hiển thị dưới dạng submenu trong admin (thay vì menu chính)
    // 'submenu'      => 'func1,func2',

    // Thư mục uploads phụ (ngoài thư mục gốc)
    // 'uploads_dir'  => [$module_upload, $module_upload . '/source', $module_upload . '/thumb'],

    // Thư mục files/ (khác với uploads/) — vd: chứa file cấu hình, dữ liệu tĩnh
    // 'files_dir'    => [$module_upload . '/topics'],
];
```

### functions.php

Module đơn giản — chỉ define hằng:
```php
<?php
if (!defined('NV_SYSTEM')) {
    exit('Stop!!!');
}
define('NV_IS_MOD_TENMODULE', true); // tên hằng riêng cho từng module
```

Module phức tạp — có thể thêm logic khởi tạo:
```php
<?php
if (!defined('NV_SYSTEM')) {
    exit('Stop!!!');
}

// Định nghĩa có điều kiện (vd: không define khi ở một số func đặc biệt)
if (!in_array($op, ['viewcat', 'detail'], true)) {
    define('NV_IS_MOD_TENMODULE', true);
}

// Load shared helper dùng chung cả frontend lẫn admin
require_once NV_ROOTDIR . '/modules/' . $module_file . '/global.functions.php';

// Có thể khởi tạo dữ liệu dùng chung ($global_array_cat, menu dọc, v.v.)
```

### global.functions.php

File hàm dùng chung cho cả frontend (`functions.php`) lẫn admin (`admin.functions.php`). Không bắt buộc nhưng là pattern chuẩn cho module phức tạp.

```php
<?php
// Guard: NV_MAINFILE (giống block của module)
if (!defined('NV_MAINFILE')) {
    exit('Stop!!!');
}

// Có thể dùng PSR-4 class từ thư mục Shared/
use NukeViet\Module\tenmodule\Shared\Helper;

/**
 * Hàm helper dùng chung cả frontend lẫn admin
 */
function nv_tenmodule_get_item($id)
{
    global $db_slave;
    $sql = 'SELECT * FROM ' . NV_PREFIXLANG . '_tenmodule WHERE id = ' . (int) $id;
    return $db_slave->query($sql)->fetch();
}
```

### admin.functions.php

**Pattern 1 — module đơn giản (như page):** `$allow_func` khai báo tại đây.

```php
<?php
if (!defined('NV_ADMIN') or !defined('NV_MAINFILE') or !defined('NV_IS_MODADMIN')) {
    exit('Stop!!!');
}

// Danh sách func admin được phép
$allow_func = ['main', 'content', 'del', 'edit'];
define('NV_IS_FILE_ADMIN', true);

// Func chỉ dành riêng cho super admin
if (defined('NV_IS_SPADMIN')) {
    $allow_func[] = 'config';
}

// Load shared helper (nếu có global.functions.php)
// require_once NV_ROOTDIR . '/modules/' . $module_file . '/global.functions.php';
```

**Pattern 2 — module phức tạp (như news):** `$allow_func` khai báo trong `admin.menu.php`, file này chỉ chứa hàm hiển thị + định nghĩa hằng.

```php
<?php
if (!defined('NV_ADMIN') or !defined('NV_MAINFILE') or !defined('NV_IS_MODADMIN')) {
    exit('Stop!!!');
}

define('NV_IS_FILE_ADMIN', true);

// Load shared helper
require_once NV_ROOTDIR . '/modules/' . $module_file . '/global.functions.php';

// Hàm hiển thị dùng trong nhiều trang admin (vd: dropdown danh mục)
function nv_tenmodule_show_cat_list($selected = 0)
{
    // ...
}
```

### admin.menu.php

**Pattern 1 — module đơn giản (như page):** chỉ khai báo `$submenu`.

```php
<?php
if (!defined('NV_ADMIN')) {
    exit('Stop!!!');
}

// Chỉ khai báo $submenu (menu hiển thị trên UI)
$submenu['content'] = $lang_module['menu_content']; // key tương ứng với tên func

// Config menu chỉ hiển thị với super admin
if (defined('NV_IS_SPADMIN')) {
    $submenu['config'] = $lang_module['menu_config'];
}

// $allow_func được khai báo trong admin.functions.php
```

**Pattern 2 — module phức tạp (như news):** khai báo cả `$allow_func` và `$submenu`.

```php
<?php
if (!defined('NV_ADMIN')) {
    exit('Stop!!!');
}

// Module phức tạp có thể khai báo $allow_func và $submenu cùng nhau ở đây
$allow_func = ['main', 'content', 'edit', 'del'];

// Submenu dạng đơn giản
$submenu['content'] = $lang_module['menu_content'];

// Submenu dạng lồng nhau (nested)
$submenu['setting'] = [
    'title'   => $lang_module['menu_setting'],
    'submenu' => [
        'voices' => $lang_module['menu_voices'],
        'config' => $lang_module['menu_config'],
    ]
];

// Thêm func theo quyền
if (defined('NV_IS_SPADMIN')) {
    $allow_func[] = 'system';
    $submenu['system'] = $lang_module['menu_system'];
}
```

### action_mysql.php
```php
<?php
if (!defined('NV_IS_FILE_MODULES')) {
    exit('Stop!!!');
}

$sql_drop_module = [];
$sql_drop_module[] = 'DROP TABLE IF EXISTS ' . $db_config['prefix'] . '_' . $lang . '_' . $module_data;
$sql_drop_module[] = 'DROP TABLE IF EXISTS ' . $db_config['prefix'] . '_' . $lang . '_' . $module_data . '_config';

$sql_create_module = $sql_drop_module;

// Bảng chính
$sql_create_module[] = 'CREATE TABLE ' . $db_config['prefix'] . '_' . $lang . '_' . $module_data . ' (
 id mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
 title varchar(250) NOT NULL,
 alias varchar(250) NOT NULL,
 status tinyint(1) unsigned NOT NULL DEFAULT \'0\',
 weight smallint(4) NOT NULL DEFAULT \'0\',
 admin_id mediumint(8) unsigned NOT NULL DEFAULT \'0\',
 add_time int(11) NOT NULL DEFAULT \'0\',
 edit_time int(11) NOT NULL DEFAULT \'0\',
 PRIMARY KEY (id),
 UNIQUE KEY alias (alias)
) ENGINE=MyISAM';

// Bảng config module (pattern chuẩn — hầu hết module có bảng _config riêng)
$sql_create_module[] = 'CREATE TABLE ' . $db_config['prefix'] . '_' . $lang . '_' . $module_data . '_config (
 config_name varchar(30) NOT NULL,
 config_value varchar(255) NOT NULL,
 UNIQUE KEY config_name (config_name)
) ENGINE=MyISAM';

// Insert giá trị mặc định cho config
$sql_create_module[] = "INSERT INTO " . $db_config['prefix'] . '_' . $lang . '_' . $module_data . "_config VALUES
('per_page', '20'),
('status_default', '1')";

// Config toàn cục (NV_CONFIG_GLOBALTABLE) — dùng khi tích hợp comment hoặc config hệ thống
// $sql_create_module[] = "INSERT INTO " . NV_CONFIG_GLOBALTABLE . " (lang, module, config_name, config_value)
//     VALUES ('" . $lang . "', '" . $module_name . "', 'allowed_comm', '-1')";
```

### funcs/main.php
```php
<?php
if (!defined('NV_IS_MOD_TENMODULE')) {
    exit('Stop!!!');
}

// Logic xử lý dữ liệu — có thể viết inline hoặc gọi hàm từ theme.php
$contents = nv_tenmodule_main($data);

include NV_ROOTDIR . '/includes/header.php';
echo nv_site_theme($contents);
include NV_ROOTDIR . '/includes/footer.php';
```

### theme.php
```php
<?php
if (!defined('NV_IS_MOD_TENMODULE')) {
    exit('Stop!!!');
}

/**
 * Hiển thị trang chính
 * @param array $data
 * @return string
 */
function nv_tenmodule_main($data)
{
    global $lang_module, $lang_global, $module_info;

    // Dùng $module_info['module_theme'] (không phải $module_file) cho đường dẫn tpl ngoài site
    // Fallback về 'default' nếu theme hiện tại chưa có tpl
    $theme = file_exists(NV_ROOTDIR . '/themes/' . $module_info['template'] . '/modules/' . $module_info['module_theme'] . '/main.tpl')
        ? $module_info['template'] : 'default';

    $xtpl = new XTemplate('main.tpl', NV_ROOTDIR . '/themes/' . $theme . '/modules/' . $module_info['module_theme']);
    $xtpl->assign('LANG', $lang_module);
    $xtpl->assign('GLANG', $lang_global);
    $xtpl->parse('main');

    return $xtpl->text('main');
}
```

### admin/main.php
```php
<?php
if (!defined('NV_IS_FILE_ADMIN')) {
    exit('Stop!!!');
}

// Admin dùng $module_file (không phải $module_info['module_theme']) cho đường dẫn tpl
$xtpl = new XTemplate('main.tpl', NV_ROOTDIR . '/themes/' . $global_config['module_theme'] . '/modules/' . $module_file);
$xtpl->assign('LANG', $lang_module);
$xtpl->assign('GLANG', $lang_global);
$xtpl->parse('main');
$contents = $xtpl->text('main');

include NV_ROOTDIR . '/includes/header.php';
echo nv_admin_theme($contents);
include NV_ROOTDIR . '/includes/footer.php';
```

### PSR-4 Classes trong module

Module phức tạp có thể dùng PSR-4 autoloading. Đặt class trong thư mục `Shared/` (hoặc `Log/`, `Service/`...):

```
modules/ten-module/
└── Shared/
    ├── Posts.php     → namespace NukeViet\Module\tenmodule\Shared;
    └── Helper.php    → namespace NukeViet\Module\tenmodule\Shared;
```

```php
<?php
// modules/ten-module/Shared/Posts.php
namespace NukeViet\Module\tenmodule\Shared;

if (!defined('NV_MAINFILE')) {
    exit('Stop!!!');
}

class Posts
{
    const STATUS_DEACTIVE = 0;
    const STATUS_PUBLISH  = 1;
    const STATUS_PENDING  = 2;

    // ...
}
```

Dùng trong code:
```php
use NukeViet\Module\tenmodule\Shared\Posts;

if ($row['status'] === Posts::STATUS_PUBLISH) {
    // ...
}
```

### Lấy input — PHẢI qua $nv_Request

```php
// Số nguyên
$id    = $nv_Request->get_int('id', 'get', 0);
$page  = $nv_Request->get_int('page', 'get', 1);

// Số nguyên không âm (≥0)
$num   = $nv_Request->get_absint('num', 'get', 0);

// Boolean
$active = $nv_Request->get_bool('active', 'post', false);

// Chuỗi ngắn (text field, tên, tiêu đề)
$title = $nv_Request->get_title('title', 'post', '');
$title = nv_substr($title, 0, 255);

// Chuỗi đã lọc bảo mật — HTML bị strip/escape (dùng cho slug, alias, search keyword...)
// Lưu ý: get_string() KHÔNG phải raw — vẫn chạy qua security filter
$alias = $nv_Request->get_string('alias', 'post', '');

// Nội dung rich editor (WYSIWYG) — chỉ đọc từ POST, không có param $mode
$body  = $nv_Request->get_editor('body', '', NV_ALLOWED_HTML_TAGS);

// Nội dung textarea — chỉ đọc từ POST; $save=true chuyển newline → <br />
$desc  = $nv_Request->get_textarea('desc', '', NV_ALLOWED_HTML_TAGS);
$desc_save = $nv_Request->get_textarea('desc', '', '', true); // newline → <br />

// Mảng ID (ví dụ checkbox nhiều lựa chọn)
$ids   = $nv_Request->get_array('ids', 'post', []);

// Ghi session (ví dụ: đếm view không trùng lặp)
$nv_Request->set_Session('key_name', NV_CURRENTTIME);
$time_set = $nv_Request->get_int('key_name', 'session');

// Đưa lại vào editor/textarea sau khi lấy từ DB:
$body  = nv_htmlspecialchars(nv_editor_br2nl($row['body']));
$desc  = nv_htmlspecialchars(nv_br2nl($row['description']));
```

---

## Block trong module

Có hai loại block:

| Loại | File | Guard |
|---|---|---|
| Block toàn site (`global.*`) | `modules/ten-module/blocks/global.TEN.php` | `NV_MAINFILE` |
| Block theo context module (`module.*`) | `modules/ten-module/blocks/module.TEN.php` | `NV_MAINFILE` |

`blocks/global.*` — hiển thị mọi nơi trên site.
`blocks/module.*` — chỉ hiển thị khi module đó đang active (dùng trong sidebar của module).

```php
<?php
// Guard: NV_MAINFILE (không phải NV_IS_BLOCK_THEME — cái đó dành cho block của theme)
if (!defined('NV_MAINFILE')) {
    exit('Stop!!!');
}

// Dùng nv_function_exists() để tránh khai báo trùng khi block được load nhiều lần
if (!nv_function_exists('nv_block_config_tenblock')) {
    /**
     * Form cấu hình block (hiển thị trong trang quản trị block)
     */
    function nv_block_config_tenblock($module, $data_block, $lang_block)
    {
        $html  = '<div class="form-group">';
        $html .= '<label>' . $lang_block['numrow'] . '</label>';
        $html .= '<input type="text" name="config_numrow" value="' . $data_block['numrow'] . '">';
        $html .= '</div>';
        return $html;
    }

    /**
     * Xử lý submit form cấu hình block
     */
    function nv_block_config_tenblock_submit($module, $lang_block)
    {
        global $nv_Request;
        return [
            'error'  => [],
            'config' => [
                'numrow' => $nv_Request->get_int('config_numrow', 'post', 5)
            ]
        ];
    }

    /**
     * Render nội dung block
     */
    function nv_tenblock($block_config)
    {
        global $nv_Cache, $db, $global_config;
        $module = $block_config['module'];

        // Dùng Query Builder + cache cho SELECT
        $db->sqlreset()
            ->select('id, title, alias')
            ->from(NV_PREFIXLANG . '_' . $block_config['module_data'])
            ->where('status = 1')
            ->order('weight ASC')
            ->limit($block_config['numrow']);
        $list = $nv_Cache->db($db->sql(), 'id', $module);

        if (empty($list)) {
            return '';
        }

        // Fallback theme 3 cấp: module_theme → site_theme → default
        if (file_exists(NV_ROOTDIR . '/themes/' . $global_config['module_theme'] . '/modules/' . $module . '/block.tenblock.tpl')) {
            $block_theme = $global_config['module_theme'];
        } elseif (file_exists(NV_ROOTDIR . '/themes/' . $global_config['site_theme'] . '/modules/' . $module . '/block.tenblock.tpl')) {
            $block_theme = $global_config['site_theme'];
        } else {
            $block_theme = 'default';
        }

        $xtpl = new XTemplate('block.tenblock.tpl', NV_ROOTDIR . '/themes/' . $block_theme . '/modules/' . $module);
        foreach ($list as $row) {
            $xtpl->assign('ROW', $row);
            $xtpl->parse('main.loop');
        }
        $xtpl->parse('main');
        return $xtpl->text('main');
    }
}

// Entry point khi block được gọi từ hệ thống
if (defined('NV_SYSTEM')) {
    $content = nv_tenblock($block_config);
}
```

> Tên hàm config block: `nv_block_config_{TEN}` và `nv_block_config_{TEN}_submit`.
> Tên hàm render: bất kỳ — thường `nv_{TEN}`.

---

## Checklist tạo module mới

- [ ] 4 file bắt buộc có đủ
- [ ] `functions.php` tồn tại — không xóa dù rỗng
- [ ] Phiên bản dạng `X.Y.ZZ`
- [ ] Template `.tpl` đặt trong `themes/` không phải `modules/`
- [ ] Language có đủ `vi.php` và `admin_vi.php`
- [ ] `action_mysql.php` có cả `$sql_drop_module` và `$sql_create_module`
- [ ] `admin.functions.php` có `define('NV_IS_FILE_ADMIN', true)`
- [ ] `$allow_func` khai báo trong `admin.functions.php` (module đơn giản) hoặc `admin.menu.php` (module phức tạp)
- [ ] Input qua `$nv_Request`, output qua `nv_htmlspecialchars()`
- [ ] theme.php dùng `$module_info['module_theme']` cho đường dẫn tpl ngoài site
- [ ] admin/*.php dùng `$module_file` cho đường dẫn tpl admin
