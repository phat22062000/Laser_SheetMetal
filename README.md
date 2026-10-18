# Laser_SheetMetal

Tài liệu, quy tắc, template và script cho quy trình: **bản vẽ PDF/DXF khách → DXF trải phôi kim loại tấm → NC cắt laser CamMagic LA (Mitsubishi)**.

Tổng hợp từ 2 skill (`dxf-sheetmetal`, `nc-sheetmetal`) và tài liệu `Context_Laser` của project Create_Flatten_SheetMetal. Cập nhật lần đầu: 18/10/2026.

## Cấu trúc thư mục

| Thư mục | Nội dung | Dùng khi |
|---|---|---|
| `skills/` | Bản gốc đầy đủ của 2 skill (`dxf-sheetmetal`, `nc-sheetmetal`). **Nguồn chuẩn** để Claude chạy | Cập nhật skill, đồng bộ lên Claude |
| `dxf/` | Quy tắc tạo DXF, tách thành 9 file theo chủ đề + ghi chú lỗi | Tra cứu khi vẽ phôi |
| `dxf/template/` | `Template.dxf` (AC1024, ANSI_932, HANDSEED 25D) | Bắt buộc dùng làm khung file DXF đầu ra |
| `nc/` | Quy tắc NC CamMagic LA + `cammagic_nc.py` | Tạo file `.nc` từ DXF |
| `rules/` | Quy tắc laser theo vật liệu/độ dày, bảng lỗ mồi taro | Tra piercing, joint, Laser R, lỗ nhỏ nhất, lỗ mồi |
| `customers/` | Yêu cầu đặc biệt theo mã khách | Tra theo mã khách trên CAD依頼書 |
| `data/Bang_Do_Gian/` | Bảng độ giãn Asahi (mặc định) và AICOKU ALPHA (khách 646) | Tra độ giãn khi bản vẽ không ghi tay |

### Chi tiết `dxf/`
`01_Quy_Uoc_Chung` · `02_Doc_Ban_Ve` · `03_Kegaki` · `04_Duong_Uon_Do_Gian` · `05_Hop_4_Canh_Va_Rach_Goc` · `06_Lo_Va_Laser_R` · `07_File_ALL` · `08_Template_Va_Xuat_DXF` · `09_Quy_Trinh` · `Ghi_Chu_Loi_Tao_DXF`.

Mỗi file ghi rõ số dòng tương ứng trong `skills/dxf-sheetmetal/SKILL.md`. Nội dung được cắt nguyên văn, không viết lại.

## Quy trình tóm tắt

1. Đọc CAD依頼書: mã yêu cầu, mã khách, vật liệu, độ dày từng mã, kích thước phôi tham khảo.
2. Tra `rules/Quy_Tac_Laser.md` và `customers/DS_Khach_Hang.md` (theo mã khách).
3. Độ giãn: ưu tiên số ghi tay trên bản vẽ; không có thì tra `data/Bang_Do_Gian/`.
4. Dựng phôi, xuất DXF từ `dxf/template/Template.dxf` (quy tắc ở `dxf/`), đặt tên `26-<mã 6 số>.dxf`, thêm `<số đơn>_ALL.dxf`.
5. Tạo NC: `python3 nc/cammagic_nc.py <file.dxf> <SS|SUS|AL|CU> <độ dày> [file_ra.nc]`.

## Lưu ý đồng bộ (đã phát hiện khi gộp)

1. **Tên file tham chiếu trong skill khác tên file trong repo.**

   | Skill ghi | File trong repo |
   |---|---|
   | `Quy_tac_laser_gon.md`, `Quy_tac_laser_kim_loai_tam.md` | `rules/Quy_Tac_Laser.md` (bản gọn; bản gốc đầy đủ `Quy_tac_laser_kim_loai_tam.md` không có trong project) |
   | `DS_khach_hang_gon.md`, `DS_khach_hang_yeu_cau_dac_biet.md` | `customers/DS_Khach_Hang.md` (bản gọn) |
   | `Quy_Tac_Lo_Moi.md`, `Quy_Tac_Laser.md`, `DS_Khach_Hang.md`, `Template.dxf` | đúng tên, nằm ở `rules/`, `customers/`, `dxf/template/` |

2. **`Quy_tac_NC_Cam_LA.md` trong project cũ hơn skill `nc-sheetmetal`.** Bản project thiếu quy tắc mới 14/10/2026 (cạnh trái theo Oy < 10 thì đục piercing ngoài ở cạnh nằm trên Ox, cách đầu trái 40). Repo dùng nội dung của skill (`nc/Quy_Tac_NC_CamMagic_LA.md`) và không đưa bản cũ vào. Nên cập nhật hoặc xóa file cũ trong project để tránh dùng nhầm.
3. **`Ghi_Chu_Loi_Tao_DXF.md` ghi HANDSEED = 208, đã lỗi thời.** `Template.dxf` hiện là `25D` (hex), khớp với skill. Đã chú thích ngay trong file.
4. `nc/cammagic_nc.py` lấy từ skill `nc-sheetmetal`. Bản `cammagic_nc.py` trong project có cùng logic nhưng chưa so sánh từng byte.
5. `Template.dxf` lưu đúng như bản trong project (UTF-8, CRLF). Khi ghép entity, đọc/ghi theo cách ở `dxf/08_Template_Va_Xuat_DXF.md` và kiểm tra lại trước khi giao file.

## Git

Repo đã có `.gitattributes` giữ nguyên CRLF cho `.dxf`, `.nc` và coi `.xlsx` là nhị phân. Khởi tạo:

```
cd Laser_SheetMetal
git init
git add .
git commit -m "Khởi tạo Laser_SheetMetal"
```
