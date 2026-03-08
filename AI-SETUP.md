# NukeViet Claude Code

Bộ cấu hình AI cho nhóm phát triển NukeViet 4.x.
Hỗ trợ đồng thời: **Claude Code · GitHub Copilot · Cursor · Windsurf**

---

## Kiến trúc: 1 nguồn — nhiều AI

```
docs/ai-context.md          ← nội dung thật (conventions, bảo mật, cấu trúc, MySQL, lộ trình nâng cấp)
         ↑ pointer từ:
  CLAUDE.md  ·  .github/copilot-instructions.md  ·  .cursor/rules/nukeviet.mdc  ·  .windsurfrules
```

Cập nhật convention → **chỉ sửa 1 file**: `docs/ai-context.md`.

---

## Cấu trúc đầy đủ

```
├── docs/
│   ├── ai-context.md                    ← nguồn sự thật — AI nào cũng đọc được
│   └── upgrade/
│       ├── module/                      ← breaking changes module từng bước
│       │   ├── NV-4.4.02-len-4.5.00.md
│       │   ├── NV-4.5.00-len-4.5.02.md
│       │   ├── NV-4.5.05-len-4.5.06.md
│       │   └── NV-4.5.06-len-4.5.07.md
│       └── theme/                       ← breaking changes theme từng bước
│           └── (cùng tên file)
│
├── CLAUDE.md                            ← Claude Code entry point
├── .github/copilot-instructions.md      ← GitHub Copilot entry point
├── .cursor/rules/nukeviet.mdc           ← Cursor entry point
├── .windsurfrules                       ← Windsurf entry point
│
└── .claude/                             ← Claude Code only
    ├── skills/
    │   ├── nukeviet-module/             — template code tạo module
    │   ├── nukeviet-theme/              — template code giao diện
    │   ├── nukeviet-security/           — patterns bảo mật + fix
    │   ├── nukeviet-mysql/              — query patterns + schema
    │   └── nukeviet-upgrade/            — workflow nâng cấp module/theme
    ├── commands/
    │   ├── new-module.md                ← /new-module [tên]
    │   ├── new-theme.md                 ← /new-theme [yêu cầu]
    │   ├── upgrade-module.md            ← /upgrade-module [path] từ X lên Y
    │   ├── upgrade-theme.md             ← /upgrade-theme [path] từ X lên Y
    │   ├── review-mr.md                 ← /review-mr [branch]
    │   └── security-audit.md            ← /security-audit [path]
    ├── agents/nukeviet-reviewer.md
    └── hooks/pre-bash.sh · post-edit.sh · post-task.sh
```

---

## Cài đặt

```bash
# Copy vào root dự án NukeViet, rồi:
npm install -g @anthropic-ai/claude-code   # Node.js 18+
claude login
cd /var/www/nukeviet && claude
```

---

## Thêm tài liệu nâng cấp phiên bản mới

```
1. Tạo docs/upgrade/module/NV-X.X.XX-len-X.X.XX.md
2. Tạo docs/upgrade/theme/NV-X.X.XX-len-X.X.XX.md
3. Cập nhật bảng lộ trình trong docs/ai-context.md
```

---

## Token budget

| File | Load khi nào | Kích thước |
|---|---|---|
| `CLAUDE.md` | Mỗi session | ~900B |
| `docs/ai-context.md` | Khi Claude cần tra cứu | ~3KB |
| Skills | Theo task cụ thể | ~3–5KB/skill |
| Commands | Theo `/slash` | ~1–2KB |
| `docs/upgrade/*` | Khi nâng cấp, theo lộ trình | ~1KB/file |
