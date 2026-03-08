# /upgrade-theme — Nâng Cấp Theme NukeViet

**Yêu cầu:** $ARGUMENTS  
_(vd: `themes/my-theme từ 4.4 lên 4.5.07`)_

---

## Thực hiện

### 1. Xác định thông tin

- **Theme path:** đọc từ yêu cầu
- **Phiên bản hiện tại / đích:** đọc từ yêu cầu hoặc `config.ini`

### 2. Đọc tài liệu theo đúng thứ tự lộ trình

Mở tuần tự từng file trong `docs/upgrade/theme/`:

| Lộ trình | File cần đọc (theo thứ tự) |
|---|---|
| 4.4 → 4.5.07 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.4 → 4.5.02 | `NV-4.4.02-len-4.5.00.md` → `NV-4.5.00-len-4.5.02.md` |
| 4.5.00 → 4.5.07 | `NV-4.5.00-len-4.5.02.md` → `NV-4.5.05-len-4.5.06.md` → `NV-4.5.06-len-4.5.07.md` |
| 4.5.06 → 4.5.07 | `NV-4.5.06-len-4.5.07.md` |

### 3. So sánh file tpl hệ thống với default

Với mỗi file tpl hệ thống thay đổi trong tài liệu:

```bash
diff themes/default/layout/[file].tpl themes/[ten-theme]/layout/[file].tpl
```

- Theme đang **override** tpl đó → merge thủ công
- Theme **không override** → tự cập nhật khi sync từ default

### 4. Cập nhật config.ini

Áp dụng các thay đổi `config.ini` được ghi trong tài liệu (thẻ mới, thẻ xóa, tên position).

### 5. Xóa cache

```
Admin → Công cụ web → Dọn dẹp hệ thống → Làm sạch cache
```

### 6. Kiểm tra cuối

- [ ] `block.default.tpl` vẫn tồn tại
- [ ] Giao diện ngoài site — desktop + mobile
- [ ] Kéo thả block hoạt động bình thường
- [ ] Block position mới (nếu có) hiển thị đúng vị trí

### 7. Báo cáo

Tóm tắt: file đã sửa · thay đổi đã áp dụng · vấn đề còn tồn đọng (nếu có).
