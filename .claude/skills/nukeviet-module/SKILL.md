---
name: nukeviet-module
description: Tạo module NukeViet 4.5 mới với cấu trúc chuẩn, admin panel CRUD, phân quyền, validate input, bảo mật XSS/CSRF/SQLi, comment tiếng Việt. Dùng khi tạo module mới, scaffold chức năng, viết CRUD cho NukeViet.
allowed-tools: Read, Write, Glob, Grep, Bash
---

# Skill: Tạo Module NukeViet 4.5

## Khi nào dùng
- Tạo module mới cho NukeViet
- Scaffold CRUD, danh sách, form
- Tạo cấu trúc thư mục module

## Quy trình bắt buộc

1. Đọc các module hiện có trong `modules/` để hiểu pattern đang dùng
2. Tạo đúng cấu trúc thư mục chuẩn
3. Áp dụng đầy đủ bảo mật (escape output, CSRF, prepared query)
4. Viết comment PHPDoc tiếng Việt cho mọi function
5. Tạo file language `vi.php` và `en.php`
6. Tạo `sql/install.sql` và `sql/uninstall.sql`

## Template: module.php
```php
<?php
/**
 * Khai báo module [TEN_MODULE]
 *
 * @package   NukeViet\Modules\[TenModule]
 * @author    NukeViet Developer Team
 * @copyright © 2025 NukeViet CMS
 */

if (!defined('NV_IS_FILE_MODULES')) {
    die('Stop!');
}
```

## Template: funcs/main.php
```php
<?php
/**
 * Xử lý hiển thị frontend module [TEN_MODULE]
 */

if (!defined('NV_IS_FILE_MODULES')) {
    die('Stop!');
}

// Lấy tham số URL, cast type rõ ràng
$page = isset($array_op[0]) ? (int) $array_op[0] : 1;
$page = max(1, $page);

// Query dùng class $db — không nối chuỗi trực tiếp
$sql = 'SELECT id, title, alias, pubdate FROM ' . NV_PREFIXLANG . '_[ten_module]'
    . ' WHERE status = 1'
    . ' ORDER BY pubdate DESC'
    . ' LIMIT ' . NV_ROWS_PER_PAGE . ' OFFSET ' . (($page - 1) * NV_ROWS_PER_PAGE);
$result = $db->query($sql);

$items = [];
while ($row = $db->fetch_assoc($result)) {
    // Escape output chống XSS
    $row['title'] = nv_htmlspecialchars($row['title']);
    $items[] = $row;
}
```

## Template: admin/funcs/main.php (save action)
```php
<?php
if (!defined('NV_IS_FILE_MODULES')) {
    die('Stop!');
}

// Kiểm tra quyền admin
if (!defined('NV_IS_ADMIN')) {
    die('Không có quyền truy cập');
}

if ($op === 'save') {
    // Bắt buộc: kiểm tra CSRF token
    if (!nv_check_formtoken()) {
        nv_jsonOutput(['status' => 'error', 'message' => 'Token không hợp lệ']);
    }

    // Validate và sanitize input
    $title = isset($_POST['title']) ? nv_sanitize_string(trim($_POST['title'])) : '';

    if (empty($title)) {
        nv_jsonOutput(['status' => 'error', 'message' => 'Tiêu đề không được để trống']);
    }

    // Lưu DB dùng dbescape
    $id = (int) ($_POST['id'] ?? 0);
    if ($id > 0) {
        $sql = 'UPDATE ' . NV_PREFIXLANG . '_[ten_module]'
            . ' SET title = ' . $db->dbescape($title)
            . ', updated_at = ' . NV_CURRENTTIME
            . ' WHERE id = ' . $id;
    } else {
        $sql = 'INSERT INTO ' . NV_PREFIXLANG . '_[ten_module] (title, created_at, status)'
            . ' VALUES (' . $db->dbescape($title) . ', ' . NV_CURRENTTIME . ', 1)';
    }

    if ($db->query($sql)) {
        nv_jsonOutput(['status' => 'success', 'message' => 'Lưu thành công']);
    }
}
```

## Template: sql/install.sql
```sql
CREATE TABLE IF NOT EXISTS `NV_PREFIXLANG_[ten_module]` (
  `id`         INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `title`      VARCHAR(255) NOT NULL DEFAULT '',
  `content`    MEDIUMTEXT NOT NULL,
  `status`     TINYINT(1) NOT NULL DEFAULT '1' COMMENT '1=hiện, 0=ẩn',
  `created_at` INT(11) UNSIGNED NOT NULL DEFAULT '0',
  `updated_at` INT(11) UNSIGNED NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `idx_status` (`status`),
  KEY `idx_created_at` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Checklist sau khi tạo
- [ ] Mọi output qua `nv_htmlspecialchars()`
- [ ] Mọi POST action kiểm tra `nv_check_formtoken()`
- [ ] SQL dùng `$db->dbescape()` hoặc cast type
- [ ] Có `vi.php` và `en.php`
- [ ] Có `install.sql` và `uninstall.sql`
- [ ] PHPDoc tiếng Việt cho mọi function
