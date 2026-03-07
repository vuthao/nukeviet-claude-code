---
name: nukeviet-security
description: Review và fix bảo mật code PHP NukeViet 4.5. Phát hiện XSS, CSRF, SQL Injection, lỗ hổng upload file, phân quyền sai. Dùng khi review bảo mật, kiểm tra code trước merge, audit lỗ hổng bảo mật NukeViet.
allowed-tools: Read, Grep, Glob, Bash
---

# Skill: Bảo Mật NukeViet 4.5

## Khi nào dùng
- Review bảo mật code PHP
- Tìm lỗ hổng XSS, CSRF, SQLi
- Kiểm tra trước khi merge MR
- Audit bảo mật định kỳ

## Bước thực hiện review

1. Dùng Grep tìm các pattern nguy hiểm:
```bash
# Tìm SQL injection nghi ngờ
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" . --include="*.php" | grep -v "dbescape\|sanitize\|nv_\|filter_var"

# Tìm output không escape
grep -rn "echo \$\|print \$" . --include="*.php" | grep -v "htmlspecialchars\|nv_html\|intval\|number_format"

# Tìm POST handler thiếu CSRF
grep -rn "REQUEST_METHOD.*POST\|_POST\[" . --include="*.php" | grep -v "formtoken\|nv_check"
```

2. Đọc từng file có vấn đề và phân tích chi tiết

3. Báo cáo theo mức độ:
   - 🔴 **CHẶN MERGE** — Lỗ hổng nghiêm trọng, phải fix ngay
   - 🟡 **NÊN FIX** — Rủi ro trung bình, fix trước khi lên production
   - 💡 **GỢI Ý** — Cải thiện tốt hơn, không bắt buộc

## Các lỗi hay gặp và cách fix

### SQL Injection
```php
// ❌ SAI
$sql = "SELECT * FROM nv_users WHERE id = " . $_GET['id'];
$sql = "SELECT * FROM nv_news WHERE title LIKE '%" . $_POST['kw'] . "%'";

// ✅ ĐÚNG
$id = (int) $_GET['id'];
$sql = 'SELECT * FROM ' . NV_TABLEPREFIX . '_users WHERE id = ' . $id;

$kw = $db->dbescape('%' . nv_sanitize_string($_POST['kw']) . '%');
$sql = 'SELECT * FROM ' . NV_PREFIXLANG . '_news WHERE title LIKE ' . $kw;
```

### XSS
```php
// ❌ SAI
echo $_GET['keyword'];
echo $row['title'];
echo $user['name'];

// ✅ ĐÚNG
echo nv_htmlspecialchars($_GET['keyword']);
echo nv_htmlspecialchars($row['title']);
echo nv_htmlspecialchars($user['name']);
```

### CSRF
```php
// ❌ SAI
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // xử lý luôn
}

// ✅ ĐÚNG
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!nv_check_formtoken()) {
        nv_jsonOutput(['status' => 'error', 'message' => 'Yêu cầu không hợp lệ']);
        exit();
    }
    // xử lý
}
```

### Phân quyền
```php
// ❌ SAI — không kiểm tra quyền
function deleteItem($id) {
    global $db;
    $db->query('DELETE FROM ' . NV_PREFIXLANG . '_items WHERE id = ' . (int)$id);
}

// ✅ ĐÚNG
function deleteItem($id) {
    global $db, $admin_info, $module_info;
    if (!defined('NV_IS_ADMIN')) { die('Không có quyền'); }
    if (empty($admin_info['module_' . $module_info['module_name']]['delete'])) {
        die('Không có quyền xóa');
    }
    $db->query('DELETE FROM ' . NV_PREFIXLANG . '_items WHERE id = ' . (int)$id);
}
```

### Upload file
```php
// ✅ ĐÚNG — kiểm tra MIME type thật
$allowed = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);

if (!in_array($mime, $allowed, true)) {
    // từ chối
}

// Đổi tên file, không dùng tên gốc
$ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));
$new_name = md5(uniqid(mt_rand(), true)) . '.' . $ext;
```

## Hàm bảo mật NukeViet quan trọng

| Hàm | Dùng khi |
|---|---|
| `nv_htmlspecialchars($str)` | Output HTML |
| `nv_sanitize_string($str)` | Làm sạch input chuỗi |
| `$db->dbescape($val)` | Giá trị vào SQL |
| `nv_check_formtoken()` | Kiểm tra CSRF |
| `nv_get_formtoken_field()` | Sinh input CSRF trong form |
| `nv_is_email($email)` | Validate email |
| `(int) $value` | Cast số nguyên |
| `(float) $value` | Cast số thực |
