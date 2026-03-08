# CLAUDE.md — NukeViet 4.x

## Stack
PHP 8.1+, MySQL 8.0, NukeViet 4.x · Ubuntu 22.04, Nginx+PHP-FPM · GitLab: `main` (prod) | `develop` (staging)

## Lệnh thường dùng
```bash
phpcs --standard=PSR2 modules/<module>/   # lint
phpcbf --standard=PSR2 modules/<module>/  # autofix
find modules/<module>/ -name "*.php" -exec php -l {} \;  # syntax check
```

## Quy tắc bảo mật — KHÔNG được vi phạm

| Tình huống | Đúng |
|---|---|
| Lấy input | `$nv_Request->get_int/get_title/get_editor()` — không dùng `$_GET/$_POST` trực tiếp |
| SQL | `$db->dbescape()` hoặc `(int)` — không nối chuỗi input |
| HTML output | `nv_htmlspecialchars()` |
| Kiểm tra file | `nv_is_file()` — không dùng `is_file()` với path từ user |
| Redirect | `$page_url` hoặc `nv_redirect_encrypt()` — không dùng `$client_info['selfurl']` trực tiếp |
| Admin ghi | Kiểm tra `defined('NV_IS_ADMIN')` trước |
| `$db->dbescape()` | Tự bao dấu nháy đơn — không thêm nháy trong SQL |

## NukeViet conventions

```
Bảng đa ngôn ngữ : NV_PREFIXLANG . '_ten_bang'   → nv4_vi_news
Bảng dùng chung  : NV_TABLEPREFIX . '_ten_bang'   → nv4_users
Thời gian        : NV_CURRENTTIME
Đường dẫn gốc    : NV_ROOTDIR
URL              : ?lang=vi&nv=ten-module&op=ten-func
```

**Đầu mỗi file — kiểm tra hằng:**
```
version.php          → NV_ADMIN + NV_MAINFILE
functions.php        → NV_SYSTEM
admin.functions.php  → NV_ADMIN + NV_MAINFILE + NV_IS_MODADMIN
admin.menu.php       → NV_ADMIN
action_mysql.php     → NV_IS_FILE_MODULES
funcs/main.php       → NV_IS_MOD_TENMODULE  (define trong functions.php)
admin/main.php       → NV_IS_FILE_ADMIN
```

**Code style:** indent 4 spaces · camelCase biến/hàm · PascalCase class · PHPDoc + comment tiếng Việt

## Git
Không push thẳng `main`/`develop` · Commit: `feat|fix|refactor|docs: mô tả [AI-assisted]` · MR cần 1 peer review

## Skills (load khi cần — đọc file tương ứng trong `.claude/skills/`)
- `nukeviet-module`   — cấu trúc file, template code, checklist tạo module
- `nukeviet-theme`    — cấu trúc theme, layout, block position, XTemplate
- `nukeviet-security` — patterns nguy hiểm, ví dụ fix
- `nukeviet-mysql`    — prefix, $db methods, query patterns

## Slash commands
`/new-module` · `/new-theme` · `/review-mr` · `/security-audit`
