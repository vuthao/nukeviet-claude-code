# Claude Code Setup — NukeViet 4.5 Team

Bộ cấu hình Claude Code cho nhóm 10 người phát triển NukeViet 4.5.

## Cài đặt Claude Code

```bash
# Yêu cầu: Node.js 18+
npm install -g @anthropic-ai/claude-code

# Đăng nhập (lần đầu)
claude login

# Khởi động trong thư mục dự án
cd /var/www/nukeviet
claude
```

## Cài đặt cho dự án

Copy toàn bộ thư mục vào root dự án:

```
nukeviet-project/
├── CLAUDE.md                          ← Claude đọc mỗi session
└── .claude/
    ├── skills/
    │   ├── nukeviet-module/SKILL.md   ← Tạo module chuẩn
    │   ├── nukeviet-security/SKILL.md ← Review bảo mật
    │   └── nukeviet-mysql/SKILL.md    ← Query MySQL
    ├── commands/
    │   ├── new-module.md              ← /new-module
    │   ├── review-mr.md               ← /review-mr
    │   └── security-audit.md         ← /security-audit
    └── agents/
        └── nukeviet-reviewer.md       ← Agent review độc lập
```

## Slash Commands

| Lệnh | Dùng khi |
|---|---|
| `/new-module [tên module]` | Tạo module mới đầy đủ |
| `/review-mr [tên MR hoặc path]` | Review Merge Request |
| `/security-audit modules/[tên]` | Audit bảo mật module |

**Ví dụ:**
```
/new-module tin tức có danh mục, bình luận, phân quyền
/review-mr feature/add-payment-module
/security-audit modules/shop
```

## Skills (Claude tự load)

Skills được load tự động khi Claude nhận ra context phù hợp:

- Gõ *"tạo module"* → load `nukeviet-module`
- Gõ *"review bảo mật"* → load `nukeviet-security`
- Gõ *"viết query"* → load `nukeviet-mysql`

## Quy trình nhóm 10 người

```
Developer viết code
      ↓
claude> /review-mr [branch]    ← Claude review tự động
      ↓
Fix vấn đề bảo mật 🔴
      ↓
Tạo Merge Request trên GitLab
      ↓
Peer review (1 người)
      ↓
Merge vào develop
```

## Cấu hình cá nhân (gitignored)

Mỗi developer có thể tạo `.claude/settings.local.json` để tuỳ chỉnh cá nhân (file này được gitignore):

```json
{
  "model": "claude-opus-4-5",
  "theme": "dark"
}
```

## Lưu ý bảo mật

- `CLAUDE.md` được commit vào git — **KHÔNG** để password, API key trong này
- Dùng biến môi trường cho thông tin nhạy cảm
- File `.claude/settings.local.json` đã được gitignore

## Yêu cầu

- Node.js 18+
- Tài khoản Anthropic (có API key hoặc Claude.ai Pro)
- Git, PHP 8.1+
