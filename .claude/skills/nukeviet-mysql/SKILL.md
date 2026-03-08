---
name: nukeviet-mysql
description: Query MySQL chuẩn NukeViet 4.x — prefix, $db methods, pattern phân trang, INSERT/UPDATE an toàn, schema bảng. Load khi viết query hoặc thiết kế schema.
allowed-tools: Read, Write, Bash, Grep
---

# Skill: MySQL NukeViet 4.x

## Prefix bảng
```php
NV_PREFIXLANG  . '_ten_bang'   // đa ngôn ngữ  → nv4_vi_news
NV_TABLEPREFIX . '_ten_bang'   // dùng chung   → nv4_users
// KHÔNG hardcode: 'nv4_vi_news'
```

## $db — method hay dùng
```php
$db->query($sql)           // chạy query
$db->fetch_assoc($result)  // lấy 1 dòng
$db->num_rows($result)     // đếm dòng
$db->insert_id()           // ID vừa INSERT
$db->dbescape($val)        // escape + bao nháy đơn → 'value'
$db->result($result, 0)    // ô đầu tiên
```
> `dbescape()` trả về `'value'` — KHÔNG thêm nháy trong SQL

## Pattern: phân trang
```php
$where = ' WHERE status = 1';
$total = (int) $db->result(
    $db->query('SELECT COUNT(*) FROM ' . NV_PREFIXLANG . '_items' . $where), 0
);
if ($total > 0) {
    $offset = ($page - 1) * $perPage;
    $sql    = 'SELECT id, title, alias, created_at'
            . ' FROM ' . NV_PREFIXLANG . '_items' . $where
            . ' ORDER BY created_at DESC LIMIT ' . $perPage . ' OFFSET ' . $offset;
    $result = $db->query($sql);
    while ($row = $db->fetch_assoc($result)) { $items[] = $row; }
}
```

## Pattern: INSERT / UPDATE an toàn
```php
// Input đã qua $nv_Request trước khi vào đây
$title   = $db->dbescape($data['title']);    // trả về 'Tiêu đề'
$content = $db->dbescape($data['content']);
$status  = (int)($data['status'] ?? 1);

// INSERT
$sql = 'INSERT INTO ' . NV_PREFIXLANG . '_items (title, content, status, created_at)'
     . ' VALUES (' . $title . ', ' . $content . ', ' . $status . ', ' . NV_CURRENTTIME . ')';
$db->query($sql);
$new_id = $db->insert_id();

// UPDATE
$sql = 'UPDATE ' . NV_PREFIXLANG . '_items'
     . ' SET title=' . $title . ', content=' . $content . ', status=' . $status
     . ', updated_at=' . NV_CURRENTTIME
     . ' WHERE id=' . $id;
$db->query($sql);
```

## Schema bảng chuẩn (dùng trong action_mysql.php)
```php
'CREATE TABLE ' . $prefix . '_' . $lang . '_' . $module_data . '_items ('
. ' `id` MEDIUMINT(8) UNSIGNED NOT NULL AUTO_INCREMENT,'
. ' `title` VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
. ' `alias` VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT \'\',' 
. ' `content` MEDIUMTEXT CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
. ' `status` TINYINT(1) NOT NULL DEFAULT \'1\','
. ' `order` SMALLINT(5) UNSIGNED NOT NULL DEFAULT \'0\','
. ' `created_at` INT(11) UNSIGNED NOT NULL DEFAULT \'0\','
. ' `updated_at` INT(11) UNSIGNED NOT NULL DEFAULT \'0\','
. ' `author_id` INT(11) UNSIGNED NOT NULL DEFAULT \'0\','
. ' PRIMARY KEY (`id`),'
. ' KEY `idx_status` (`status`, `created_at`),'
. ' UNIQUE KEY `uq_alias` (`alias`)'
. ') ENGINE=MyISAM DEFAULT CHARSET=utf8'
// utf8mb4 nếu cần emoji — đồng bộ với $db_config['charset']
```

## Tối ưu — dấu hiệu cần xử lý
| Vấn đề | Fix |
|---|---|
| `LIKE '%kw%'` | FULLTEXT INDEX |
| Query trong vòng lặp | `IN (id1,id2,...)` |
| `SELECT *` | Chỉ SELECT cột cần |
| Không có INDEX trên WHERE | Thêm index |
