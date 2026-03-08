# /upgrade-module — Nâng Cấp Module NukeViet

**Yêu cầu:** $ARGUMENTS  
_(vd: `modules/news từ 4.4 lên 4.5.07`)_

---

## Thực hiện

### 1. Xác định thông tin

- **Module path:** đọc từ yêu cầu
- **Phiên bản hiện tại:** đọc từ yêu cầu, hoặc tự đọc `version.php` nếu không có
- **Phiên bản đích:** đọc từ yêu cầu

### 2. Đọc tài liệu theo đúng thứ tự lộ trình

Mở tuần tự từng file trong `docs/upgrade/module/`:

| Lộ trình | File cần đọc (theo thứ tự) |
|---|---|
| 4.4 → 4.5.07 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.4 → 4.5.02 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` |
| 4.5.00 → 4.5.07 | `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.5.06 → 4.5.07 | `NV-4.5.06-len-4.5.07.md` |

### 3. Quét và sửa từng breaking change

```bash
grep -rn "PATTERN" modules/[ten-module]/ --include="*.php"
```
Sửa → `php -l [file]` ngay sau mỗi lần sửa.

### 4. Cập nhật version.php

```php
'version' => '[phien-ban-dich]',
```

### 5. Kiểm tra cuối

```bash
find modules/[ten-module]/ -name "*.php" -exec php -l {} \;
phpcs --standard=PSR2 modules/[ten-module]/
```

### 6. Báo cáo

Tóm tắt: số file đã sửa · thay đổi đã áp dụng · vấn đề còn tồn đọng (nếu có).
