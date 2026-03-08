# Hướng Dẫn Theme NukeViet 4.x

## Cấu trúc thư mục

```
themes/ten-theme/
├── config.ini            # tên theme, layoutdefault, <positions>
├── theme.php             # hàm PHP của theme
├── default.jpg           # ảnh mô tả (800×600px)
├── css/
│   ├── style.css · style.responsive.css · bootstrap.min.css
│   ├── custom.css            ← viết CSS tùy chỉnh vào đây
│   └── ten-module.css        ← tự load khi module chạy
├── js/
│   ├── main.js · bootstrap.min.js
│   └── custom.js             ← viết JS tùy chỉnh vào đây
├── layout/
│   ├── layout.TENL.tpl       # layout file
│   └── block.default.tpl     # BẮT BUỘC — không xóa
├── blocks/
│   └── global.TEN.php        # block global của theme
└── modules/ten-module/       # override tpl module — chỉ copy khi thực sự cần sửa
```

---

## Layout Bootstrap 24 cột

NukeViet dùng **24 cột** (không phải 12 cột chuẩn Bootstrap).

| Layout name | Cột |
|---|---|
| `main` | 24 |
| `main-right` | 18-6 |
| `left-main` | 6-18 |
| `left-main-right` | 5-13-6 |
| `main-left-right` | 13-6-5 |
| `left-right-main` | 5-6-13 |

---

## Tạo theme mới từ default

```bash
cp -r themes/default themes/ten-theme-moi
```

**Dọn dẹp sau khi copy:**
- `blocks/` → xóa hết, giữ `index.html`
- `css/` → xóa module css thừa; giữ `admin.css, bootstrap*.css, custom.css, style*.css`
- `images/` → giữ `icons/, index.html, no_image.gif`
- `js/` → giữ `bootstrap.min.js, custom.js, main.js`
- `layout/` → giữ `block.default.tpl`; xóa block/layout không dùng
- `modules/` → xóa hết (copy lại từng module khi cần sửa)

---

## config.ini — cấu hình cơ bản

```xml
<theme>
  <info>
    <n>Tên giao diện</n>
    <author>Tác giả</author>
    <version>1.0</version>
    <thumbnail>default.jpg</thumbnail>
  </info>
  <layoutdefault>main</layoutdefault>
  <positions>
    <position><n>Header</n><tag>[HEADER]</tag></position>
    <position><n>Left</n><tag>[LEFT]</tag></position>
    <position><n>Right</n><tag>[RIGHT]</tag></position>
    <position><n>Footer</n><tag>[FOOTER]</tag></position>
  </positions>
</theme>
```

> Sau khi sửa `config.ini`: **Admin → Công cụ web → Làm sạch cache**

---

## Thêm block position mới

**Bước 1** — Khai báo trong `config.ini`:
```xml
<position><n>Tên hiển thị</n><tag>[TEN_KHOI]</tag></position>
```
Tag: **in hoa**, chỉ dùng chữ/số/gạch dưới — vd: `[BOTTOM_CONTENT]`, `[BANNER_TOP]`

**Bước 2** — Đặt tag vào file layout `.tpl`:
```html
<div class="container">[BOTTOM_CONTENT]</div>
```

**Bước 3** — Xóa cache → Admin → kéo thả block vào position mới để kiểm tra.

---

## XTemplate — cú pháp .tpl

```html
{BIEN_DON}              <!-- biến đơn -->
{MANG.key}              <!-- mảng -->

<!-- BEGIN: main.loop -->
  <li>{ITEM.title}</li>
<!-- END: main.loop -->

<!-- BEGIN: main.co_anh -->
  <img src="{ROW.image}">
<!-- END: main.co_anh -->
```

```php
$xtpl->assign('BIEN', $value);          // biến đơn
$xtpl->assign('ROW', $row);             // mảng → {ROW.field}
foreach ($items as $item) {
    $xtpl->assign('ITEM', $item);
    $xtpl->parse('main.loop');          // lặp
}
if (!empty($row['image'])) {
    $xtpl->parse('main.co_anh');        // điều kiện
}
$xtpl->parse('main');
return $xtpl->text('main');
```

---

## Block global của theme

```php
// File: themes/ten-theme/blocks/global.ten-block.php
if (!defined('NV_IS_BLOCK_THEME')) die('Stop!!!');
$content = '...'; // $content là output trả về
```

---

## Quy tắc tùy biến — ưu tiên theo thứ tự

1. **CSS thuần** vào `custom.css` — nhanh nhất, ít rủi ro nhất
2. **CSS pseudo-elements** (`:before`, `:after`, `:first-child`)
3. **Copy `.tpl`** vào `modules/ten-module/` — chỉ khi CSS không đủ

> Không copy `.tpl` chỉ để đổi màu hay khoảng cách — dùng CSS trước.

---

## Checklist theme mới

- [ ] `block.default.tpl` tồn tại — không xóa
- [ ] `config.ini` có `<layoutdefault>` hợp lệ
- [ ] CSS tùy chỉnh → `custom.css` | JS tùy chỉnh → `custom.js`
- [ ] Xóa layout/template không dùng
- [ ] Xóa cache sau khi thay đổi `config.ini`
- [ ] Test giao diện desktop + mobile
