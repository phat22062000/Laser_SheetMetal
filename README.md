# Laser_SheetMetal

Skill, quy tắc, script và dữ liệu cho quy trình: bản vẽ PDF/DXF khách hàng, DXF trải phôi kim loại tấm, NC cắt laser (CamMagic LA, Mitsubishi).

Repo này chứa thông tin khách hàng nội bộ, giữ ở chế độ private.

## Cấu trúc

| Thư mục | Nội dung |
|---|---|
| `skills/dxf-sheetmetal/` | Skill tạo DXF trải phôi từ PDF/DXF khách hàng |
| `skills/nc-sheetmetal/` | Skill tạo NC CamMagic LA từ DXF trải phôi (có nhúng script) |
| `rules/laser/` | Quy tắc laser theo vật liệu/độ dày (`Quy_Tac_Laser.md`), lỗ mồi taro/khoét (`Quy_Tac_Lo_Moi.md`) |
| `rules/nc/` | Quy tắc định dạng và đường chạy NC (`Quy_tac_NC_Cam_LA.md`) |
| `rules/customers/` | Danh sách khách hàng yêu cầu đặc biệt (`DS_Khach_Hang.md`) |
| `scripts/` | `cammagic_nc.py`: DXF sang NC |
| `data/bend_allowance/` | Bảng độ giãn: Asahi (mặc định), AICOKU ALPHA (khách 646) |
| `templates/` | `Template.dxf`: file mẫu bắt buộc cho đầu ra DXF (AC1024, ANSI_932) |
| `tests/samples/` | File NC mẫu đã được người dùng xác nhận đúng |

## Quan hệ với Claude Project

Skill vẫn đọc quy tắc và template từ Project docs (`Context_Laser/`). Repo là bản lưu và quản lý phiên bản, không tự đồng bộ ngược. Khi sửa một file ở đây, cập nhật cả Project docs (và ngược lại).

| Project docs | Repo |
|---|---|
| `Context_Laser/Quy_Tac_Laser.md` | `rules/laser/Quy_Tac_Laser.md` |
| `Context_Laser/Quy_Tac_Lo_Moi.md` | `rules/laser/Quy_Tac_Lo_Moi.md` |
| `Context_Laser/Quy_tac_NC_Cam_LA.md` | `rules/nc/Quy_tac_NC_Cam_LA.md` |
| `Context_Laser/DS_Khach_Hang.md` | `rules/customers/DS_Khach_Hang.md` |
| `Context_Laser/cammagic_nc.py` | `scripts/cammagic_nc.py` |
| `Context_Laser/Template.dxf` | `templates/Template.dxf` |
| `Context_Laser/Bang_Do_Gian_*.xlsx` | `data/bend_allowance/` |

## Lưu ý khi sửa

- `scripts/cammagic_nc.py` là bản gốc. Skill NC nhúng một bản sao trong `SKILL.md`; sửa script thì sửa cả bản nhúng.
- Bảng `TABLE` trong script phải khớp mục 2 của `Quy_Tac_Laser.md`.
- `Template.dxf` mã hóa ANSI_932 (cp932), kết thúc dòng CRLF. Không mở rồi lưu lại bằng trình soạn thảo tự đổi mã hóa; đặt `*.dxf binary` trong `.gitattributes` để Git không đổi xuống dòng.
- Skill chỉ nhận các tên file ở bảng trên. Nếu đổi tên file, sửa tham chiếu trong 2 `SKILL.md`.

## Việc còn thiếu

- `tests/samples/` mới có file `.nc`. Chưa có file `.dxf` đầu vào tương ứng nên chưa chạy lại được kiểm tra khớp từng byte. Cần bổ sung DXF cùng mã (26-057027, 26-057101, 26-057140, 26-058543, 26-059090, 26-059097).
- Skill NC còn nhắc các mẫu 26-058544, 26-058546, 26-058551 chưa có trong `tests/samples/`.
- Đường dẫn máy cá nhân (`C:\Users\ADMIN\...`) còn cứng trong skill NC.
