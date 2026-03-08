# CLAUDE.md — NukeViet 4.x

Đọc `docs/ai-context.md` trước. Chi tiết kỹ thuật ở từng file trong `docs/`.

## Lệnh thường dùng
```bash
phpcs --standard=PSR2 modules/<module>/
phpcbf --standard=PSR2 modules/<module>/
find modules/<module>/ -name "*.php" -exec php -l {} \;
```

## Tài liệu kỹ thuật (`docs/`)
- `ai-context.md`    — tổng quan, conventions, bảo mật, MySQL
- `module-guide.md`  — cấu trúc file, template code, checklist
- `theme-guide.md`   — layout, block position, XTemplate
- `security-guide.md`— scan, lỗi phổ biến, cách fix
- `mysql-guide.md`   — $db methods, patterns, schema
- `upgrade-guide.md` — lộ trình nâng cấp module/theme
- `upgrade/module/`  — breaking changes từng phiên bản
- `upgrade/theme/`   — breaking changes theme từng phiên bản

## Slash commands
`/new-module` · `/new-theme` · `/upgrade-module` · `/upgrade-theme` · `/review-mr` · `/security-audit`
