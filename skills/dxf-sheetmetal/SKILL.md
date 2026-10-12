---
name: "dxf-sheetmetal"
description: "Tạo bản vẽ triển khai kim loại tấm từ hình vẽ trong file PDF hay dữ liệu DXF."
---

## Yêu cầu đầu ra
- Đầu ra là file DXF đặt tên theo cú pháp: "26-" + "mã yêu cầu" (mã yêu cầu là 6 số dưới mã vạch). Ví dụ: 26-059090.dxf.
- Lưu file DXF vào chính thư mục chứa PDF yêu cầu.
- Công thức tính chiều dài phôi trước uốn: Kích thước ngoài + Kích thước ngoài - Độ giãn dài.
- Hình vẽ phải được đặt tại gốc tọa độ (góc dưới trái = 0,0), chiều dài hơn của tấm phôi nằm dọc theo trục X.
- Hướng xoay: sau khi đặt cạnh dài theo Ox (chỉ xoay bội số 90°, không lật), chọn hướng sao cho các cạnh thẳng của biên dạng nằm trên cả Ox và Oy (tiếp xúc bằng cạnh, không phải chỉ một đỉnh chạm trục). Ví dụ 26-057025: khối chữ nhật nằm sát góc dưới trái, cánh nghiêng hướng lên phải. Nếu có 2 hướng (xoay 180°) đều có cạnh áp trục thì chọn hướng có cạnh áp trục dài hơn (ví dụ 26-057027: cạnh thẳng dài nhất nằm trên Ox).
- Phôi dài và hẹp (chiều theo Oy ≤ 100 và chiều theo Ox ≥ 1000): thêm các POINT nằm trên mép trên của phôi (y = Oy lớn nhất), mỗi điểm kèm 1 TEXT tham khảo cao 10, đặt đúng tại điểm đó (layer 0, style Standard). Số điểm J theo chiều dài L theo Ox:
  - 1000 ≤ L ≤ 1500: 1 J + 1 LJ.
  - 1500 < L ≤ 2000: 2 J + 1 LJ.
  - 2000 < L ≤ 2500: 3 J + 1 LJ.
  - Vị trí: điểm đầu tiên (chữ "J") ở x = 30, điểm cuối cùng (chữ "LJ") ở x = L - 30; tất cả các điểm (các J và LJ) chia đều giữa 2 vị trí này, các J ở bên trái, LJ ở ngoài cùng bên phải. Với N = số J + 1 điểm, khoảng cách = (L - 60) / (N - 1). Ví dụ 2J + LJ: x = 30, L/2, L - 30; 3J + LJ: x = 30, 30 + (L - 60)/3, 30 + 2(L - 60)/3, L - 30.
  - Ví dụ 26-057101 (1199.5 x 99.5): POINT (30; 99.5) + TEXT J cao 10, POINT (1169.5; 99.5) + TEXT LJ cao 10.
- Kích thước phôi X/Y trong bảng CAD依頼書 chỉ là tham khảo; kích thước tạo theo bản vẽ mới là chính xác.
- Vị trí lỗ và chi tiết: PHẢI tính từ chuỗi kích thước ghi trên bản vẽ. Với bản vẽ lắp (lỗ được ghi kích thước tới tấm khác), quy đổi qua kích thước các tấm liên quan (ví dụ 26-058546: tâm φ19 = 68 - 50 = 18 từ mép tấm C, không phải 17.5 ở giữa). Chỉ dùng số đo pixel trên ảnh scan để kiểm tra (ảnh scan có thể nghiêng/méo), không lấy làm kích thước. Không tự giả định "nằm giữa"/"đối xứng" khi bản vẽ không ghi; nếu không tìm được chuỗi kích thước thì ghi rõ trong báo cáo.
- Mặt của hình vẽ thể hiện là mặt ngoài của phần uốn. Nếu hình chiếu đứng cho thấy mặt trong thì lật trái–phải (không đổi chiều dọc) để vẽ mặt ngoài. Nếu bản vẽ đánh dấu mặt 表 (mặt trước) thì vẽ mặt 表 theo hình chiếu đứng. Chi tiết uốn ngược chiều (chữ Z/Ω/mũ) không có mặt ngoài rõ ràng và bản vẽ không đánh dấu 表 thì vẽ theo hình chiếu bằng (平面図, hình nhìn từ trên xuống) đúng như trên bản vẽ, không lật (ví dụ 26-057412: hình Ω 4 chỗ uốn, vẽ như hình chiếu bằng, đầu cánh trái ở x = 0). Chi tiết phẳng không uốn thì giữ nguyên hướng hình chiếu đứng.
- Kegaki (vạch dấu, ケガキ): khi có chỉ thị KEGAKI/ケガキ, hoặc khách hàng yêu cầu vạch dấu vị trí hàn (ví dụ khách 851), vẽ vết của tấm hàn lên mặt tấm (footprint) bằng LINE màu Đỏ (mã màu DXF 62 = 1), layer 0, theo `Quy_Tac_Laser.md` mục 5.
  - Chỉ vẽ các đoạn của vết nằm bên trong phôi (đoạn trùng mép phôi thì bỏ). Kegaki đặt trên tấm nhận (tấm có mặt bị tấm khác hàn lên); tấm chỉ hàn bằng cạnh vào tấm khác thì không có kegaki.
  - Chi tiết có kegaki: mặt bản vẽ là mặt kegaki (ưu tiên hơn quy tắc mặt ngoài/hình chiếu đứng). Nếu hình chiếu đứng cho thấy mặt đối diện thì lật trái–phải (không đổi chiều dọc).
  - Chi tiết 本図/対称 (R/L): hình dạng và kegaki của 対称 là ảnh gương của 本図.
  - Ví dụ đơn 521006-13: 26-058546 (tấm C) vạch 2 đường tại vị trí tấm B ở mặt sau nên lật trái–phải so với hình chiếu đứng; 26-058551 (tấm B t9) vạch hình chữ nhật 9 x 39.32 (đầu vát xiên của tấm A).
- Mỗi chỗ uốn vẽ bằng 2 đường DASHED thể hiện độ giãn dài (không vẽ 1 đường ở giữa):
  - Đường 1 cách mép phôi bên này đúng bằng kích thước ngoài của cánh bên này.
  - Đường 2 cách mép phôi bên kia đúng bằng kích thước ngoài của cánh bên kia.
  - Khoảng cách giữa 2 đường = độ giãn của chỗ uốn đó. Ví dụ chữ L 37 + 60, giãn 5.2, phôi 91.8: đường tại 37 và 31.8 (= 91.8 - 60).
  - Với nhiều chỗ uốn (chữ U...): mỗi chỗ uốn là cặp đường đặt tại (vị trí giữa ± độ giãn/2), vị trí giữa = kích thước ngoài của cánh - độ giãn/2 tính từ mép xa.
- Hình hộp mở bốn cánh (khay, hộp có 4 chỗ uốn): phôi hình chữ thập, khoét 4 góc. Đặt a = (kích thước ngoài của cánh) - độ giãn, t = độ dày tấm.
  - Khoét góc theo X: bằng a (cánh ở 2 cạnh theo phương Oy giữ nguyên bề rộng đầy đủ, không lùi).
  - Khoét góc theo Y: bằng a + t (2 cánh nằm ở hai đầu theo phương Ox được lùi một đoạn bằng t ở mỗi đầu).
  - Thêm 4 đường rạch ở 4 góc khoét, vẽ màu Green (mã màu DXF 62 = 3), hướng từ ngoài vào:
    - Điểm bắt đầu nằm trong phần khoét (ngoài phôi), cách mỗi mép khoét (2 cạnh tạo góc) một khoảng bằng Piercing ngoài theo bảng `Quy_Tac_Laser.md` mục 2 (ví dụ thép ≤3.2 → 3; thép 4–9 → 6...).
    - Đi chéo 45° tới góc khoét, bo R0.5 tại chỗ chuyển hướng, rồi rạch thẳng dọc theo mép khoét theo phương Ox (mép của cánh đã lùi) vào phía đáy, sâu t + 0.2 tính từ góc khoét.
    - Gồm 3 entity: LINE (điểm bắt đầu → tiếp điểm), ARC R0.5, LINE (tiếp điểm → cuối đường rạch); LINE vẽ từ ngoài vào.
  - Các góc lồi bo theo quy tắc Laser R; góc lõm không bo. Đường uốn DASHED chạy trong phạm vi mỗi cánh.
  - Ví dụ 26-057140 (SPHC-P t1.6, giãn 2.6, cánh 121.6, phôi 491.2 x 441.2): khoét 119.0 theo X, 120.6 theo Y; đường rạch góc dưới trái: (116, 117.6) → (118.854, 120.454), cung tâm (119.207, 120.1) R0.5 từ 90° đến 135°, (119.207, 120.6) → (120.8, 120.6).
- Kích thước trong ngoặc ( ) trên bản vẽ chỉ là tham khảo, không nhất thiết phải chính xác; chỉ cần tuân thủ các kích thước không nằm trong ngoặc. Nếu có file DXF của khách thì hình dạng lấy theo DXF; không cần hỏi lại khi chỉ lệch với số trong ngoặc hoặc bảng CAD依頼書 do số đó.
- Lỗ/gia công bị gạch (bút đỏ hoặc gạch chéo, ví dụ 2-M8, φ12 H7, M5) thì không vẽ; lỗ có ghi 「あける」 thì vẽ.
- Laser R (bo góc ngoài): với hình không có chỉ thị riêng (bản vẽ và danh sách khách hàng đặc biệt không nói gì), PHẢI bo các góc ngoài (góc lồi) có góc ≤ 90° theo bảng `Quy_Tac_Laser.md` (mục 2) theo nhóm vật liệu và độ dày:
  - Thép (SS: SPHC, SPCC, SS400...): dày ≤ 5 → R0.5; 6–9 → R2; 10–16 → không bo; ≥ 19 → R3 cả góc trong và ngoài.
  - Inox/khác: ≤ 6 → R0.5; 7 trở lên → không bo. Nhôm: ≤ 6 → R0.5. Đồng/đồng thau: ≤ 5 → R0.5.
  - Góc đã vát C (135°) và góc lõm (góc trong) không bo (trừ thép ≥ 19t). Nếu khách hàng có quy định riêng (không bo, bo R3...) thì theo khách hàng; bản vẽ có ghi R thì theo bản vẽ.
  - Vẽ bằng ARC (tròn tiếp tuyến với 2 cạnh), cắt ngắn 2 cạnh đi tại điểm tiếp tuyến. Kích thước ghi trên bản vẽ là kích thước chưa bo.

## File tổng hợp tất cả hình
- Ngoài các file riêng, tạo thêm 1 file DXF chứa tất cả hình, đặt tên `<số đơn>_ALL.dxf` (ví dụ 521008-11_ALL.dxf), lưu cùng thư mục. Khi người dùng yêu cầu chỉ xuất file ALL thì không tạo file riêng từng mã.
- Xếp thành 1 cột dọc, căn trái tại X = 0, các hình cách nhau 100 mm theo trục Y. Thứ tự mã tăng dần từ trên xuống dưới (mã nhỏ nhất ở trên cùng); hình ở dưới cùng đặt sát Y = 0.
- Giữ nguyên chiều quay, hình dạng và màu như file riêng (gồm cả POINT/TEXT tham khảo và kegaki nếu có), chỉ tịnh tiến.
- Mỗi hình có 1 dòng chữ TEXT ghi mã yêu cầu (6 số), cao 12 mm, đặt trên cạnh trên của hình khoảng 15 mm, style Standard, layer 0. Cập nhật EXTMAX theo toàn bộ hình + chữ.
- Chữ cùng layer 0 với đường cắt là chủ ý, người dùng đã biết; không cần nhắc lại.

## File mẫu Template.dxf (BẮT BUỘC dùng làm template)
- File mẫu nằm trong Project docs: `Template.dxf` (đọc bằng Projects `project_read`; luôn đọc lại bản mới nhất mỗi lần làm vì người dùng có thể cập nhật). Nếu người dùng đính kèm file, dùng bản đính kèm.
- File DXF đầu ra phải giống hệt file mẫu (AC1024, ANSI_932, CLASSES, TABLES, BLOCKS, OBJECTS... gồm layer 0, Defpoints, DIM nếu có trong mẫu), CHỈ khác phần hình vẽ. Không tự dựng header/tables.
- Cách làm:
  1. Đọc template dạng bytes, decode `cp932`, tách theo `\r\n` thành cặp (mã nhóm, giá trị).
  2. Xóa toàn bộ entity giữa `2/ENTITIES` và `0/ENDSEC` của mẫu, chèn entity mới.
  3. Entity: LINE, CIRCLE, ARC, POINT, TEXT, layer `0`, owner `330 = 6F`, subclass `AcDbEntity` + `AcDbLine`/`AcDbCircle`/`AcDbCircle`+`AcDbArc` (ARC có thêm góc 50/51, ngược chiều kim đồng hồ, độ)/`AcDbPoint` (POINT chỉ cần tọa độ 10/20/30)/`AcDbText` như mẫu. Đường uốn là LINE có thêm `6 = DASHED`. Đường rạch (LINE/ARC) có thêm `62 = 3` (Green) ngay sau layer `8`. Đường kegaki (LINE) có thêm `62 = 1` (Red) ngay sau layer `8`.
  4. Handle entity mới bắt đầu từ giá trị `$HANDSEED` của mẫu (hiện là 15F), tăng dần; sau đó cập nhật `$HANDSEED` = handle kế tiếp.
  5. Cập nhật `$EXTMAX` (10/20 = kích thước khung bao X/Y), VPORT `*Active`: 12/22 = tâm hình, 40 = chiều cao view (~1.1 × kích thước).
  6. Ghi lại với `\r\n`, mã hóa `cp932`. Kiểm tra: mọi phần ngoài ENTITIES phải trùng mẫu, trừ EXTMAX, VPORT, HANDSEED.

## Quy trình
1. Đọc trang danh sách (CAD依頼書) trong PDF: mã yêu cầu, khách hàng, vật liệu, độ dày, kích thước phôi X/Y. Dùng X/Y này để đối chiếu kết quả (chỉ là tham khảo, bản vẽ mới là chính xác).
2. Đọc tài liệu quy tắc (rules/ trong repo, hoặc Project docs): `Quy_Tac_Laser.md` (đọc toàn bộ, ngắn) và `DS_Khach_Hang.md`. Với `DS_Khach_Hang.md` chỉ cần đọc phần "Tra nhanh theo nhóm quy tắc" và dòng của đúng mã khách trong đơn (dùng `project_search` theo mã khách, không đọc cả file nếu không cần). Áp dụng quy tắc theo mã khách (bao gồm quy tắc bo góc).
3. Độ giãn: ưu tiên số ghi tay trên bản vẽ (VD: "37+60-5.2=91.8"). Nếu không có thì tra bảng `Bang_Do_Gian_Asahi.xlsx` (khách 646 dùng `Bang_Do_Gian_AICOKU ALPHA.xlsx`).
4. Với mỗi trang bản vẽ: render PDF ở 200 dpi, xem từng hình chiếu (三角法 - third angle) để xác định cạnh uốn, lỗ, vát C. Nếu thư mục có file DXF của khách (3D*.DXF...) thì stage về đọc để lấy hình dạng/vị trí lỗ chính xác.
5. Dựng phôi trong hệ tọa độ cục bộ (u dọc đường uốn, v vuông góc), sau đó xoay 90° (không lật) nếu cạnh v dài hơn, rồi chọn hướng theo quy tắc "Hướng xoay" ở phần Yêu cầu đầu ra.
   - Đường uốn: 2 đường DASHED theo quy tắc ở phần Yêu cầu đầu ra.
   - Hình hộp 4 cánh: khoét góc, lùi cánh và đường rạch Green theo quy tắc hình hộp.
   - Phôi Oy ≤ 100 và Ox ≥ 1000: thêm POINT + TEXT (J, LJ cao 10) trên mép trên, số J và vị trí chia đều theo chiều dài Ox (xem quy tắc ở phần Yêu cầu đầu ra).
   - Kegaki: xác định quan hệ lắp ráp (tấm nào hàn lên mặt nào) từ các hình chiếu, vẽ vết hàn màu Đỏ và chọn mặt vẽ là mặt kegaki (xem quy tắc Kegaki).
   - Tọa độ lỗ/chi tiết lấy theo chuỗi kích thước (xem quy tắc Vị trí lỗ); kiểm tra lại bằng số đo trên ảnh, lệch > 0.5 mm thì xem lại chuỗi kích thước.
   - Vị trí lỗ trên cánh: giữ khoảng cách đo từ mép xa của cánh. Lỗ trên phần đáy chữ U: giữ khoảng cách so với tâm đáy.
   - Vát C ở góc: vẽ thành đoạn chéo trên phôi.
   - Lỗ taro M6: vẽ lỗ mồi φ5 (cần xác nhận nếu quy định khác).
   - Bo góc ngoài ≤ 90° theo quy tắc Laser R ở phần Yêu cầu đầu ra (vẽ sau khi đã dựng đủ biên dạng, trước khi xoay).
6. Xuất DXF theo template, so kích thước phôi với bảng CAD依頼書, kiểm tra cạnh thẳng áp cả Ox và Oy, render preview để kiểm tra.
7. Tạo thêm file tổng hợp theo phần "File tổng hợp tất cả hình".
8. Báo cáo: danh sách file, kích thước phôi, các giả định (nêu rõ vị trí nào suy ra gián tiếp), kegaki đã vẽ, mã yêu cầu nào thiếu bản vẽ.