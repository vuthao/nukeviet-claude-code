# Hướng Dẫn Bảo Mật NukeViet 4.x

## Scan nhanh

```bash
# Input không qua $nv_Request
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" . --include="*.php" | grep -v "nv_Request\|dbescape\|(int)\|(float)"

# Output không escape
grep -rn "echo \$\|print \$" . --include="*.php" | grep -v "htmlspecialchars\|nv_html\|intval"

# is_file với path từ user
grep -rn "is_file\|file_exists" . --include="*.php" | grep -v "nv_is_file\|NV_ROOTDIR"

# Open Redirect qua selfurl
grep -rn "client_info\['selfurl'\]" . --include="*.php" | grep "redirect\|location\|header"
```

---

## Lỗi phổ biến và cách fix

### Input — PHẢI qua $nv_Request

```php
// ❌ Sai
$id = $_GET['id'];

// ✅ Đúng
$id    = $nv_Request->get_int('id', 'get', 0);
$title = $nv_Request->get_title('title', 'post', '');
$title = nv_substr($title, 0, 255);
$body  = $nv_Request->get_editor('body', '', NV_ALLOWED_HTML_TAGS);
$desc  = $nv_Request->get_textarea('desc', '', NV_ALLOWED_HTML_TAGS);
```

### SQL — dbescape tự bao nháy đơn

```php
// ❌ Sai — nối chuỗi trực tiếp
$sql = "WHERE id = " . $_GET['id'];

// ✅ Đúng — số dùng (int)
$id  = $nv_Request->get_int('id', 'get', 0);
$sql = 'SELECT * FROM ' . NV_PREFIXLANG . '_news WHERE id = ' . $id;

// ✅ Đúng — chuỗi dùng dbescape (tự có nháy đơn, không thêm nháy trong SQL)
$kw  = $nv_Request->get_title('kw', 'get', '');
$sql = '... WHERE title LIKE ' . $db->dbescape('%' . $kw . '%');
```

### XSS output

```php
// ❌ Sai
echo $row['title'];

// ✅ Đúng
echo nv_htmlspecialchars($row['title']);
```

### Kiểm tra file — dùng nv_is_file

```php
// ❌ Sai — path từ user, không kiểm soát
is_file(NV_DOCUMENT_ROOT . $path_from_user);

// ✅ Đúng
nv_is_file($path_from_user, $uploads_dir_user);
```

### Open Redirect — không dùng selfurl trực tiếp

```php
// ❌ Sai
nv_redirect_location($client_info['selfurl']);

// ✅ Đúng
nv_redirect_location($page_url);

// Nếu cần truyền URL qua form: nv_redirect_encrypt() / nv_get_redirect()
// Nếu hiển thị ra HTML: nv_htmlspecialchars($client_info['selfurl'])
```

### Upload file

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime  = finfo_file($finfo, $_FILES['f']['tmp_name']);
finfo_close($finfo);
if (!in_array($mime, ['image/jpeg', 'image/png', 'image/gif', 'image/webp'], true)) {
    // từ chối upload
}
$ext      = nv_getextension($_FILES['f']['name']);
$new_name = md5(uniqid(mt_rand(), true)) . '.' . strtolower($ext);
```

### Phân quyền admin

```php
function xoaItem($id) {
    if (!defined('NV_IS_ADMIN')) die('Stop!!!');
    global $db;
    $db->query('DELETE FROM ' . NV_PREFIXLANG . '_items WHERE id = ' . (int) $id);
}
```

---

## Bảng tra nhanh — hàm bảo mật

| Hàm | Dùng khi |
|---|---|
| `$nv_Request->get_int/float` | Input số |
| `$nv_Request->get_title` | Input chuỗi/text ngắn |
| `$nv_Request->get_editor` | Nội dung rich editor |
| `$nv_Request->get_textarea` | Nội dung textarea |
| `nv_htmlspecialchars()` | Escape HTML output |
| `$db->dbescape()` | Escape vào SQL (tự có nháy đơn) |
| `nv_is_file()` | Kiểm tra file an toàn |
| `nv_redirect_encrypt/decrypt` | Redirect an toàn |
| `nv_check_valid_email()` | Validate email |

---

## Mức độ báo cáo khi review

- 🔴 **CHẶN MERGE** — SQLi, XSS rõ ràng, thiếu kiểm tra phân quyền
- 🟡 **NÊN FIX** — Open Redirect, dùng `is_file` với path từ user
- 💡 **GỢI Ý** — cải thiện thêm, không bắt buộc
