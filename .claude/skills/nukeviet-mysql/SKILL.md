---
name: nukeviet-mysql
description: Viết query MySQL chuẩn và tối ưu hiệu năng cho NukeViet 4.5. Dùng khi viết query mới, tối ưu query chậm, thiết kế schema database, xử lý dữ liệu lớn, thêm index cho NukeViet.
allowed-tools: Read, Write, Bash, Grep
---

# Skill: MySQL cho NukeViet 4.5

## Khi nào dùng
- Viết query mới cho module
- Tối ưu query chạy chậm (> 1 giây)
- Thiết kế bảng database mới
- Thêm index tối ưu

## Quy tắc prefix bảng

```php
// ĐA NGÔN NGỮ (dữ liệu riêng theo ngôn ngữ)
NV_PREFIXLANG . '_ten_bang'      // vd: nv4_vi_news

// DÙNG CHUNG (users, config, không phân ngôn ngữ)
NV_TABLEPREFIX . '_ten_bang'     // vd: nv4_users

// KHÔNG BAO GIỜ hardcode
// ❌  'SELECT * FROM nv4_vi_news'
// ✅  'SELECT * FROM ' . NV_PREFIXLANG . '_news'
```

## Class $db — các method hay dùng

```php
$result  = $db->query($sql);              // Chạy query
$row     = $db->fetch_assoc($result);     // Lấy 1 dòng
$total   = $db->num_rows($result);        // Đếm số dòng kết quả
$id      = $db->insert_id();             // ID vừa INSERT
$escaped = $db->dbescape($value);        // Escape có dấu nháy
$num     = $db->result($result, 0);      // Lấy giá trị ô đầu tiên
```

## Pattern: Danh sách có phân trang

```php
/**
 * Lấy danh sách có phân trang
 *
 * @param int $page    Trang hiện tại
 * @param int $perPage Số bản ghi mỗi trang
 * @return array ['total' => int, 'items' => array]
 */
function getList(int $page = 1, int $perPage = 20): array
{
    global $db;

    $where = ' WHERE status = 1';

    // Đếm tổng — dùng COUNT, không lấy hết dữ liệu
    $total = (int) $db->result(
        $db->query('SELECT COUNT(*) FROM ' . NV_PREFIXLANG . '_items' . $where),
        0
    );

    if ($total === 0) {
        return ['total' => 0, 'items' => []];
    }

    $offset = ($page - 1) * $perPage;

    // Chỉ SELECT cột cần thiết — KHÔNG dùng SELECT *
    $sql = 'SELECT id, title, alias, pubdate, author_name'
        . ' FROM ' . NV_PREFIXLANG . '_items'
        . $where
        . ' ORDER BY pubdate DESC'
        . ' LIMIT ' . $perPage . ' OFFSET ' . $offset;

    $result = $db->query($sql);
    $items = [];
    while ($row = $db->fetch_assoc($result)) {
        $items[] = $row;
    }

    return ['total' => $total, 'items' => $items];
}
```

## Pattern: Lưu dữ liệu an toàn

```php
/**
 * Thêm mới hoặc cập nhật bản ghi
 *
 * @param array $data Dữ liệu cần lưu
 * @param int   $id   ID cập nhật (0 = thêm mới)
 * @return int|false  ID đã lưu hoặc false
 */
function saveItem(array $data, int $id = 0)
{
    global $db;

    $set = [
        'title'      => $db->dbescape($data['title']),
        'content'    => $db->dbescape($data['content']),
        'status'     => (int) ($data['status'] ?? 1),
        'updated_at' => NV_CURRENTTIME,
    ];

    if ($id > 0) {
        $parts = array_map(fn($k, $v) => "`$k` = $v", array_keys($set), $set);
        $sql = 'UPDATE ' . NV_PREFIXLANG . '_items'
            . ' SET ' . implode(', ', $parts)
            . ' WHERE id = ' . $id;
        return $db->query($sql) ? $id : false;
    }

    $set['created_at'] = NV_CURRENTTIME;
    $sql = 'INSERT INTO ' . NV_PREFIXLANG . '_items'
        . ' (`' . implode('`, `', array_keys($set)) . '`)'
        . ' VALUES (' . implode(', ', $set) . ')';
    return $db->query($sql) ? $db->insert_id() : false;
}
```

## Template tạo bảng chuẩn

```sql
CREATE TABLE IF NOT EXISTS `NV_PREFIXLANG_ten_bang` (
  `id`          INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `title`       VARCHAR(255) NOT NULL DEFAULT '',
  `alias`       VARCHAR(255) NOT NULL DEFAULT '',
  `content`     MEDIUMTEXT NOT NULL,
  `status`      TINYINT(1) NOT NULL DEFAULT '1' COMMENT '1=hiện, 0=ẩn',
  `order`       SMALLINT(5) UNSIGNED NOT NULL DEFAULT '0',
  `created_at`  INT(11) UNSIGNED NOT NULL DEFAULT '0',
  `updated_at`  INT(11) UNSIGNED NOT NULL DEFAULT '0',
  `author_id`   INT(11) UNSIGNED NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `idx_status_created` (`status`, `created_at`),
  KEY `idx_author` (`author_id`),
  UNIQUE KEY `uq_alias` (`alias`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Dấu hiệu query cần tối ưu

| Vấn đề | Giải pháp |
|---|---|
| `LIKE '%keyword%'` | Dùng FULLTEXT INDEX |
| Query trong vòng lặp | Gộp thành `IN (id1, id2, ...)` |
| `SELECT *` | Chỉ SELECT cột cần thiết |
| Không có INDEX trên WHERE | Thêm index phù hợp |
| JOIN nhiều bảng lớn | Kiểm tra `EXPLAIN`, thêm index |
| COUNT(*) chậm | Dùng bảng summary riêng nếu > 1M rows |
