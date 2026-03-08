# Hướng Dẫn MySQL NukeViet 4.x

## Prefix bảng — không hardcode

```php
NV_PREFIXLANG  . '_ten_bang'   // bảng đa ngôn ngữ  → nv4_vi_news
NV_TABLEPREFIX . '_ten_bang'   // bảng dùng chung   → nv4_users
// ❌ Không viết: 'nv4_vi_news'
```

---

## $db — method hay dùng

```php
$db->query($sql)            // chạy query
$db->fetch_assoc($result)   // lấy 1 dòng dưới dạng mảng
$db->num_rows($result)      // đếm số dòng kết quả
$db->insert_id()            // ID vừa INSERT
$db->dbescape($val)         // escape + bao nháy đơn → trả về 'value'
$db->result($result, 0)     // lấy ô đầu tiên
```

> `dbescape()` **tự bao dấu nháy đơn** — không thêm nháy thủ công trong câu SQL.

---

## Pattern: đếm + phân trang

```php
$where = ' WHERE status = 1';
$total = (int) $db->result(
    $db->query('SELECT COUNT(*) FROM ' . NV_PREFIXLANG . '_items' . $where), 0
);

if ($total > 0) {
    $offset = ($page - 1) * $perPage;
    $sql    = 'SELECT id, title, alias, created_at'
            . ' FROM '   . NV_PREFIXLANG . '_items' . $where
            . ' ORDER BY created_at DESC'
            . ' LIMIT '  . $perPage . ' OFFSET ' . $offset;
    $result = $db->query($sql);
    $items  = [];
    while ($row = $db->fetch_assoc($result)) {
        $items[] = $row;
    }
}
```

---

## Pattern: INSERT / UPDATE an toàn

```php
// Input đã qua $nv_Request trước khi vào đây
$title   = $db->dbescape($data['title']);     // → 'Tiêu đề'
$content = $db->dbescape($data['content']);
$status  = (int) ($data['status'] ?? 1);

// INSERT
$sql = 'INSERT INTO ' . NV_PREFIXLANG . '_items (title, content, status, created_at)'
     . ' VALUES (' . $title . ', ' . $content . ', ' . $status . ', ' . NV_CURRENTTIME . ')';
$db->query($sql);
$new_id = $db->insert_id();

// UPDATE
$sql = 'UPDATE ' . NV_PREFIXLANG . '_items'
     . ' SET title='   . $title
     . ', content='    . $content
     . ', status='     . $status
     . ', updated_at=' . NV_CURRENTTIME
     . ' WHERE id='    . $id;
$db->query($sql);
```

---

## Schema bảng chuẩn (dùng trong action_mysql.php)

```php
'CREATE TABLE ' . $db_config['prefix'] . '_' . $lang . '_' . $module_data . '_items ('
. ' `id`         MEDIUMINT(8) UNSIGNED NOT NULL AUTO_INCREMENT,'
. ' `title`      VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
. ' `alias`      VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT \'\','
. ' `content`    MEDIUMTEXT  CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,'
. ' `status`     TINYINT(1)  NOT NULL DEFAULT \'1\','
. ' `order`      SMALLINT(5) UNSIGNED NOT NULL DEFAULT \'0\','
. ' `created_at` INT(11)     UNSIGNED NOT NULL DEFAULT \'0\','
. ' `updated_at` INT(11)     UNSIGNED NOT NULL DEFAULT \'0\','
. ' `author_id`  INT(11)     UNSIGNED NOT NULL DEFAULT \'0\','
. ' PRIMARY KEY (`id`),'
. ' KEY `idx_status` (`status`, `created_at`),'
. ' UNIQUE KEY `uq_alias` (`alias`)'
. ') ENGINE=MyISAM DEFAULT CHARSET=utf8'
// Dùng utf8mb4 nếu cần lưu emoji — đồng bộ với $db_config['charset']
```

---

## Tối ưu — dấu hiệu cần xử lý

| Vấn đề | Giải pháp |
|---|---|
| `LIKE '%keyword%'` trên cột lớn | Thêm FULLTEXT INDEX |
| Query nằm trong vòng lặp | Dùng `IN (id1, id2, ...)` một lần |
| `SELECT *` | Chỉ SELECT cột thực sự cần |
| Lọc/sort mà không có INDEX | Thêm composite index phù hợp |
