# CLAUDE.md — NukeViet 4.x

Đọc `docs/ai-context.md` trước. Chi tiết kỹ thuật ở từng file trong `docs/`.

## Quy trình làm việc

Dev mô tả task → Claude vào Plan Mode phân tích codebase + lên kế hoạch → dev duyệt plan → Claude tự code → dev review kết quả.

**Nguyên tắc: dev mô tả — Claude Code thực thi. Dev chỉ can thiệp code trực tiếp khi thực sự cần thiết.**

Khi nhận task:
1. Luôn vào Plan Mode trước — không code ngay
2. Đọc code hiện tại liên quan trước khi đề xuất plan
3. Plan phải rõ: sửa file nào, thay đổi gì, lý do tại sao
4. Chờ dev duyệt plan trước khi bắt tay code

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
- `mysql-guide.md`   — $db/$db_slave methods, patterns, schema
- `upgrade-guide.md` — lộ trình nâng cấp module/theme
- `upgrade/module/`  — breaking changes từng phiên bản
- `upgrade/theme/`   — breaking changes theme từng phiên bản

## Slash commands
`/new-module` · `/new-theme` · `/add-func` · `/upgrade-module` · `/upgrade-theme` · `/review-mr` · `/security-audit`
