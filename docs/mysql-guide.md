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
$db->query($sql)                  // chạy query, trả về PDOStatement
$db->query($sql)->fetch()         // chạy query + lấy 1 dòng
$db->query($sql)->fetchAll()      // chạy query + lấy tất cả dòng
$db->query($sql)->fetchColumn()   // chạy query + lấy giá trị ô đầu tiên (COUNT, MAX...)
$db->prepare($sql)                // chuẩn bị prepared statement
$db->lastInsertId()               // ID vừa INSERT
```

---

## Pattern: SELECT

```php
// Nhiều dòng
$sql  = 'SELECT id, title FROM ' . NV_PREFIXLANG . '_items WHERE status = 1 ORDER BY weight ASC';
$rows = $db->query($sql)->fetchAll();

// 1 dòng
$sql = 'SELECT * FROM ' . NV_PREFIXLANG . '_items WHERE id = ' . (int) $id . ' LIMIT 1';
$row = $db->query($sql)->fetch();

// 1 giá trị (COUNT, MAX...)
$total = (int) $db->query('SELECT COUNT(*) FROM ' . NV_PREFIXLANG . '_items')->fetchColumn();
$max   = (int) $db->query('SELECT MAX(weight) FROM ' . NV_PREFIXLANG . '_items')->fetchColumn();
```

## Pattern: phân trang

```php
$where = ' WHERE status = 1';
$total = (int) $db->query('SELECT COUNT(*) FROM ' . NV_PREFIXLANG . '_items' . $where)->fetchColumn();

if ($total > 0) {
    $offset = ($page - 1) * $perPage;
    $sql    = 'SELECT id, title, alias, created_at'
            . ' FROM '  . NV_PREFIXLANG . '_items' . $where
            . ' ORDER BY created_at DESC'
            . ' LIMIT ' . $perPage . ' OFFSET ' . $offset;
    $rows = $db->query($sql)->fetchAll();
}
```

---

## Pattern: INSERT / UPDATE an toàn

**Quy tắc:** số nguyên và hằng hệ thống nối thẳng vào SQL — dữ liệu từ user input dùng `:named_param` + `bindParam`.

```php
// INSERT
$sql = 'INSERT INTO ' . NV_PREFIXLANG . '_items (title, content, status, created_at)
        VALUES (:title, :content, :status, ' . NV_CURRENTTIME . ')';
$sth = $db->prepare($sql);
$sth->bindParam(':title',   $row['title'],   PDO::PARAM_STR);
$sth->bindParam(':content', $row['content'], PDO::PARAM_STR, strlen($row['content']));
$sth->bindParam(':status',  $row['status'],  PDO::PARAM_INT);
$sth->execute();
$new_id = $db->lastInsertId();

// UPDATE
$sql = 'UPDATE ' . NV_PREFIXLANG . '_items
        SET title = :title, content = :content, updated_at = ' . NV_CURRENTTIME . '
        WHERE id = ' . $id;
$sth = $db->prepare($sql);
$sth->bindParam(':title',   $row['title'],   PDO::PARAM_STR);
$sth->bindParam(':content', $row['content'], PDO::PARAM_STR, strlen($row['content']));
$sth->execute();
if ($sth->rowCount()) {
    // cập nhật thành công
}

// DELETE
$db->query('DELETE FROM ' . NV_PREFIXLANG . '_items WHERE id = ' . (int) $id);
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
