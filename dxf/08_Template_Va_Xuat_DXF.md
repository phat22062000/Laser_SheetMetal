# Dùng Template.dxf, ghép entity, kiểm tra bắt buộc trước khi giao

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 70–82). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

- File mẫu nằm trong Project docs: `Template.dxf` (đọc bằng Projects `project_read`; luôn đọc lại bản mới nhất mỗi lần làm vì người dùng có thể cập nhật). Nếu người dùng đính kèm file, dùng bản đính kèm.
- File DXF đầu ra phải giống hệt file mẫu (AC1024, ANSI_932, CLASSES, TABLES, BLOCKS, OBJECTS... gồm layer 0, Defpoints, DIM nếu có trong mẫu), CHỈ khác phần hình vẽ. Không tự dựng header/tables.
- Cách làm:
  1. Đọc template dạng bytes, decode `cp932`, tách theo `\r\n` thành cặp (mã nhóm, giá trị).
  2. Xóa toàn bộ entity giữa `2/ENTITIES` và `0/ENDSEC` của mẫu, chèn entity mới.
  3. Entity: LINE, CIRCLE, ARC, POINT, TEXT, layer `0`, owner `330 = 6F`, subclass `AcDbEntity` + `AcDbLine`/`AcDbCircle`/`AcDbCircle`+`AcDbArc` (ARC có thêm góc 50/51, ngược chiều kim đồng hồ, độ)/`AcDbPoint` (POINT chỉ cần tọa độ 10/20/30)/`AcDbText` như mẫu. Đường uốn là LINE có thêm `6 = DASHED`. Đường rạch (LINE/ARC) có thêm `62 = 3` (Green) ngay sau layer `8`. Đường kegaki (LINE) có thêm `62 = 1` (Red) ngay sau layer `8`.
  4. Handle entity mới bắt đầu từ giá trị `$HANDSEED` của mẫu (đọc từ mẫu hiện tại, bản mới nhất là 25D (hex); giá trị này là handle kế tiếp chưa dùng), tăng dần; sau đó cập nhật `$HANDSEED` = handle kế tiếp.
  5. Cập nhật `$EXTMAX` (10/20 = kích thước khung bao X/Y), VPORT `*Active`: 12/22 = tâm hình, 40 = chiều cao view (~1.1 × kích thước). Trong mẫu, EXTMIN/EXTMAX gốc là ±1e20 (không phải 0) nên chỉ ghi đè EXTMAX (10/20); các dòng EXTMIN/EXTMAX khác mẫu là bình thường.
  6. Ghi lại với `\r\n`, mã hóa `cp932`. Kiểm tra: mọi phần ngoài ENTITIES phải trùng mẫu, trừ EXTMAX, VPORT, HANDSEED.
  7. LỖI ĐÃ TỪNG GẶP (13/10/2026, đơn 521013-04, CAD báo "Press ENTER to continue" và không mở được file): khi ghép entity vào mẫu, KHÔNG được cắt bỏ dòng mã nhóm `  0` đứng ngay trước `ENDSEC` của ENTITIES. Các dòng của file là cặp (mã, giá trị): `ENDSEC` là giá trị, dòng trước nó (`  0`) là mã. Ghép đúng: `L[:i+1] + entity + L[j-1:]` (i = chỉ số `ENTITIES`, j = chỉ số `ENDSEC` sau đó). Ghép sai `L[:i+1] + entity + L[j:]` làm mất mã `0`, file hỏng.
  8. Kiểm tra bắt buộc TRƯỚC khi giao file: (a) toàn bộ file đọc theo từng cặp 2 dòng, dòng mã luôn là số nguyên; (b) phần từ `0/ENDSEC` cuối ENTITIES đến hết file trùng khớp từng dòng với mẫu (`mẫu[j-1:] == ra[l-1:]`); (c) chỉ khác mẫu ở EXTMAX, HANDSEED, VPORT; (d) không có số dạng `1e-05`, `nan`; (e) không tin báo "written" của công cụ ghi file: sau khi ghi vào thư mục người dùng phải liệt kê thư mục để xác nhận file có mặt và đúng dung lượng, nếu lệch thì gửi bằng SendUserFile lấy file_uuid rồi commit lại bằng fileUuid.
