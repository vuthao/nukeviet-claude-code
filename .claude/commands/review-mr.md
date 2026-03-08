# Review Merge Request NukeViet 4.5

Thực hiện review đầy đủ các file thay đổi trong Merge Request: $ARGUMENTS

## Các bước thực hiện

1. **Xác định phạm vi thay đổi**
   - Liệt kê tất cả file PHP, SQL, template đã thay đổi
   - Ưu tiên review file xử lý input người dùng và database

2. **Review bảo mật** (Chặn merge nếu phát hiện)
   - Tìm SQL nối chuỗi trực tiếp với input
   - Tìm output HTML không qua `nv_htmlspecialchars()`
   - Tìm POST handler thiếu `nv_check_formtoken()`
   - Tìm thao tác ghi thiếu kiểm tra quyền

3. **Review code quality**
   - Tuân thủ PSR-2
   - Comment PHPDoc tiếng Việt đầy đủ
   - Không có debug code (`var_dump`, `print_r`, `die()` thừa)
   - Không có magic number không giải thích

4. **Review NukeViet conventions**
   - Dùng `NV_PREFIXLANG`/`NV_TABLEPREFIX` đúng chỗ
   - Dùng `NV_CURRENTTIME`, `NV_ROOTDIR`
   - File bắt đầu bằng kiểm tra hằng đúng (`NV_IS_FILE_ADMIN`, `NV_SYSTEM`, v.v.)
   - Lấy input qua `$nv_Request` (không dùng `$_GET/$_POST` trực tiếp)
   - Có file language cho mọi chuỗi hiển thị

5. **Review database** (nếu có thay đổi SQL)
   - `action_mysql.php` có đủ `$sql_drop_module` và `$sql_create_module`
   - Có index phù hợp
   - Charset `utf8` hoặc `utf8mb4` nhất quán
   - Không dùng từ khóa MySQL làm tên cột

## Output format

Trả về báo cáo theo format:

```
## Kết quả Review MR: [tên MR]

### 🔴 CHẶN MERGE (phải fix ngay)
- [file:dòng] Mô tả vấn đề + cách fix

### 🟡 NÊN FIX trước khi lên production
- [file:dòng] Mô tả vấn đề + gợi ý fix

### 💡 GỢI Ý cải thiện (không bắt buộc)
- [file:dòng] Gợi ý

### ✅ Tổng kết
- Số vấn đề nghiêm trọng: X
- Số vấn đề nên fix: X
- Quyết định: [APPROVE / REQUEST CHANGES / CHẶN MERGE]
```
