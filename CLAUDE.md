# CLAUDE.md — NukeViet 4.5 Project

## Dự án
- **CMS:** NukeViet 4.5
- **PHP:** 8.1+
- **Database:** MySQL 8.0
- **Server:** Ubuntu 22.04, Nginx + PHP-FPM
- **Git:** GitLab — nhánh `main` (production), `develop` (staging)

## Lệnh hay dùng

```bash
# Kiểm tra coding standard
phpcs --standard=PSR2 modules/[ten_module]/

# Tự động fix coding standard
phpcbf --standard=PSR2 modules/[ten_module]/

# Tìm lỗi bảo mật nhanh
grep -rn "\$_GET\|\$_POST" modules/ | grep -v "dbescape\|sanitize\|nv_"

# Clear cache NukeViet
php cli/clear_cache.php

# Chạy migration SQL
mysql -u root -p nukeviet < modules/[ten_module]/sql/install.sql

# Kiểm tra syntax PHP toàn module
find modules/[ten_module]/ -name "*.php" -exec php -l {} \;
```

## Quy tắc bắt buộc (KHÔNG được bỏ qua)

### Bảo mật
- Mọi SQL phải dùng `$db->dbescape()` — KHÔNG nối chuỗi với input người dùng
- Mọi output HTML phải qua `nv_htmlspecialchars()` — KHÔNG echo trực tiếp
- Mọi form POST phải kiểm tra `nv_check_formtoken()` — KHÔNG bỏ qua CSRF
- KHÔNG bao giờ để password, API key, secret trong code hoặc CLAUDE.md

### NukeViet conventions
- Prefix bảng: `NV_PREFIXLANG . '_ten_bang'` (đa ngôn ngữ) hoặc `NV_TABLEPREFIX . '_ten_bang'` (chung)
- Thời gian: dùng `NV_CURRENTTIME` thay vì `time()`
- Đường dẫn: dùng `NV_ROOTDIR` thay vì đường dẫn tuyệt đối
- Mọi file PHP bắt đầu bằng: `if (!defined('NV_IS_FILE_MODULES')) { die('Stop!'); }`

### Code style
- Indent: 4 spaces (không dùng tab)
- Comment: tiếng Việt
- Tên biến/hàm: tiếng Anh, camelCase
- PHPDoc bắt buộc cho mọi function

## Cấu trúc thư mục module chuẩn
```
modules/ten_module/
├── module.php
├── funcs/main.php
├── admin/
│   ├── index.php
│   └── funcs/main.php
├── language/
│   ├── vi.php
│   └── en.php
├── templates/main.tpl
└── sql/
    ├── install.sql
    └── uninstall.sql
```

## Git workflow
- KHÔNG push trực tiếp vào `main` hoặc `develop`
- Mọi thay đổi qua Merge Request, cần ít nhất 1 người review
- Commit message: `feat:`, `fix:`, `refactor:`, `docs:` — thêm `[AI-assisted]` nếu dùng AI

## Skills có sẵn (Claude tự load khi phù hợp)
- `nukeviet-module` — Tạo module mới đúng chuẩn
- `nukeviet-security` — Review bảo mật XSS/CSRF/SQLi
- `nukeviet-mysql` — Viết và tối ưu query MySQL
- `nukeviet-review` — Review code trước khi merge
