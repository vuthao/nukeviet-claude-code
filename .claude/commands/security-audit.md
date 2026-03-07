# Security Audit Module NukeViet 4.5

Thực hiện audit bảo mật đầy đủ cho: $ARGUMENTS

## Các bước thực hiện

1. **Scan tự động tìm pattern nguy hiểm**
```bash
# SQL injection
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" $ARGUMENTS --include="*.php" | grep -v "dbescape\|sanitize\|nv_\|(int)\|(float)"

# XSS
grep -rn "echo \$\|print \$" $ARGUMENTS --include="*.php" | grep -v "htmlspecialchars\|nv_html\|intval"

# Thiếu CSRF
grep -rn "REQUEST_METHOD.*POST" $ARGUMENTS --include="*.php" -A3 | grep -v "formtoken"

# Hardcode credential
grep -rn "password\|passwd\|secret\|api_key" $ARGUMENTS --include="*.php" | grep -v "//\|#\|\$_POST\|\$config"
```

2. **Review thủ công các file quan trọng**
   - Tất cả file trong `admin/funcs/`
   - File xử lý upload
   - File xử lý thanh toán (nếu có)
   - File API endpoint

3. **Kiểm tra phân quyền**
   - Mọi action admin có check `NV_IS_ADMIN`
   - Mọi thao tác ghi có check quyền module

4. **Kiểm tra upload file** (nếu có)
   - Kiểm tra MIME type thật (finfo, không tin $_FILES['type'])
   - Đổi tên file, không dùng tên gốc
   - Lưu ngoài webroot hoặc có .htaccess chặn execute

## Output format

```
## Security Audit: [tên module]
## Ngày audit: [ngày]

### Thống kê nhanh
- Tổng file PHP: X
- File có vấn đề: X
- Lỗ hổng nghiêm trọng: X

### 🔴 Lỗ hổng nghiêm trọng (fix ngay)
### 🟡 Rủi ro trung bình
### 💡 Cải thiện khuyến nghị
### ✅ Những điểm bảo mật tốt
```
