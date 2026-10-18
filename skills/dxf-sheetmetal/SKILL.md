---
name: "dxf-sheetmetal"
description: "Tạo bản vẽ triển khai kim loại tấm từ hình vẽ trong file PDF hay dữ liệu DXF."
---

## Yêu cầu đầu ra
- Đầu ra là file DXF đặt tên theo cú pháp: "26-" + "mã yêu cầu" (mã yêu cầu là 6 số dưới mã vạch). Ví dụ: 26-059090.dxf.
- Lưu file DXF vào chính thư mục chứa PDF yêu cầu.
- Công thức tính chiều dài phôi trước uốn: Kích thước ngoài + Kích thước ngoài - Độ giãn dài.
- Độ chính xác: tính đủ chữ số, không làm tròn giữa chừng; kết quả giữ độ chính xác tới ít nhất 2 chữ số thập phân.
- Độ giãn luôn là số dương: nếu bản vẽ ghi độ giãn âm (ví dụ "-0.48") thì hiểu độ giãn là 0.48 và vẫn trừ vào tổng kích thước ngoài theo công thức trên.
- Hình vẽ phải được đặt tại gốc tọa độ (góc dưới trái = 0,0), chiều dài hơn của tấm phôi nằm dọc theo trục X.
- Hướng xoay: sau khi đặt cạnh dài theo Ox (chỉ xoay bội số 90°, không lật), chọn hướng sao cho các cạnh thẳng của biên dạng nằm trên cả Ox và Oy (tiếp xúc bằng cạnh, không phải chỉ một đỉnh chạm trục). Ví dụ 26-057025: khối chữ nhật nằm sát góc dưới trái, cánh nghiêng hướng lên phải. Nếu có 2 hướng (xoay 180°) đều có cạnh áp trục thì chọn hướng có cạnh áp trục dài hơn (ví dụ 26-057027: cạnh thẳng dài nhất nằm trên Ox).
- Phôi dài và hẹp (chiều theo Oy ≤ 100 và chiều theo Ox ≥ 1000): thêm các POINT nằm trên mép trên của phôi (y = Oy lớn nhất), mỗi điểm kèm 1 TEXT tham khảo cao 10, đặt đúng tại điểm đó (layer 0, style Standard). Số điểm J theo chiều dài L theo Ox:
  - 1000 ≤ L ≤ 1500: 1 J + 1 LJ.
  - 1500 < L ≤ 2000: 2 J + 1 LJ.
  - 2000 < L ≤ 2500: 3 J + 1 LJ.
  - Vị trí: điểm đầu tiên (chữ "J") ở x = 30, điểm cuối cùng (chữ "LJ") ở x = L - 30; tất cả các điểm (các J và LJ) chia đều giữa 2 vị trí này, các J ở bên trái, LJ ở ngoài cùng bên phải. Với N = số J + 1 điểm, khoảng cách = (L - 60) / (N - 1). Ví dụ 2J + LJ: x = 30, L/2, L - 30; 3J + LJ: x = 30, 30 + (L - 60)/3, 30 + 2(L - 60)/3, L - 30.
  - Ví dụ 26-057101 (1199.5 x 99.5): POINT (30; 99.5) + TEXT J cao 10, POINT (1169.5; 99.5) + TEXT LJ cao 10.
- Kích thước phôi X/Y trong bảng CAD依頼書 chỉ là tham khảo; kích thước tạo theo bản vẽ mới là chính xác.
- Vị trí lỗ và chi tiết: PHẢI tính từ chuỗi kích thước ghi trên bản vẽ. Với bản vẽ lắp (lỗ được ghi kích thước tới tấm khác), quy đổi qua kích thước các tấm liên quan (ví dụ 26-058546: tâm φ19 = 68 - 50 = 18 từ mép tấm C, không phải 17.5 ở giữa). Chỉ dùng số đo pixel trên ảnh scan để kiểm tra (ảnh scan có thể nghiêng/méo), không lấy làm kích thước. Không tự giả định "nằm giữa"/"đối xứng" khi bản vẽ không ghi; nếu không tìm được chuỗi kích thước thì ghi rõ trong báo cáo. Khi một nhóm lỗ không được ghi kích thước theo phương vuông góc (ví dụ 6-M4 chỉ có kích thước dọc, không có khoảng cách tới mép), KHÔNG được suy từ hàng lỗ khác nằm cạnh (ví dụ lỗ φ9 cùng cánh); phải lấy tọa độ từ file DXF gốc của khách (nếu có, lưu ý tỉ lệ hình, ví dụ 1:3) hoặc đo kỹ trên hình chiếu rồi nêu rõ trong báo cáo. Ví dụ 26-059946: 6-M4 ở cánh dài cách mép tự do 7.5 (không phải 11.5 như lỗ φ9).
- Quy tắc đường nét đứt ở hình bằng (平面図) của chi tiết uốn: đường nét đứt cách mép ngoài đúng bằng độ dày t (KHÔNG phải độ giãn/2). Các kích thước ghi từ đường nét đứt này (ví dụ "8" tới lỗ đầu tiên, "50" tới đáy rãnh) phải tính từ đúng đường đó. Quy đổi sang phôi: với điểm nằm trên phần phẳng của tấm, u (vị trí trên phôi tính từ mép tự do của cánh đầu) = y (vị trí trên hình bằng tính từ mặt ngoài cánh đầu) + (kích thước ngoài của cánh - độ giãn). Ví dụ 26-060232 (t3.2, mũ 13 + 102 + 13, giãn 5.5, phôi 117): u = y + 7.5; nét đứt phía cánh xa ở y = 102 - 3.2 = 98.8; lỗ φ3.5 ở y = 98.8 - 8 - 20k nên u = 98.3, 78.3, 58.3, 38.3, 18.3; đáy rãnh ở y = 98.8 - 50 nên u = 56.3; kích thước ghi từ mặt ngoài cánh đầu (ví dụ 30.2) thì u = 30.2 + 7.5 = 37.7. Lỗ trên cánh vẫn đo từ mép tự do của cánh (ví dụ lỗ M3 cách mép 4.65).
- Mặt của hình vẽ thể hiện là mặt ngoài của phần uốn. Nếu hình chiếu đứng cho thấy mặt trong thì lật trái–phải (không đổi chiều dọc) để vẽ mặt ngoài. Nếu bản vẽ đánh dấu mặt 表 (mặt trước) thì vẽ mặt 表 theo hình chiếu đứng. Chi tiết uốn ngược chiều (chữ Z/Ω/mũ) không có mặt ngoài rõ ràng và bản vẽ không đánh dấu 表 thì vẽ theo hình chiếu bằng (平面図, hình nhìn từ trên xuống) đúng như trên bản vẽ, không lật (ví dụ 26-057412: hình Ω 4 chỗ uốn, vẽ như hình chiếu bằng, đầu cánh trái ở x = 0). Chi tiết phẳng không uốn thì giữ nguyên hướng hình chiếu đứng.
- Kegaki (vạch dấu, ケガキ): khi có chỉ thị KEGAKI/ケガキ, hoặc khách hàng yêu cầu vạch dấu vị trí hàn (ví dụ khách 851), vẽ vết của tấm hàn lên mặt tấm (footprint) bằng LINE màu Đỏ (mã màu DXF 62 = 1), layer 0, theo `Quy_tac_laser_gon.md` mục 5.
  - Chỉ vẽ các đoạn của vết nằm bên trong phôi (đoạn trùng mép phôi thì bỏ). Kegaki đặt trên tấm nhận (tấm có mặt bị tấm khác hàn lên); tấm chỉ hàn bằng cạnh vào tấm khác thì không có kegaki.
  - Chi tiết có kegaki: mặt bản vẽ là mặt kegaki (ưu tiên hơn quy tắc mặt ngoài/hình chiếu đứng). Nếu hình chiếu đứng cho thấy mặt đối diện thì lật trái–phải (không đổi chiều dọc).
  - Chi tiết 本図/対称 (R/L): hình dạng và kegaki của 対称 là ảnh gương của 本図.
  - Ví dụ đơn 521006-13: 26-058546 (tấm C) vạch 2 đường tại vị trí tấm B ở mặt sau nên lật trái–phải so với hình chiếu đứng; 26-058551 (tấm B t9) vạch hình chữ nhật 9 x 39.32 (đầu vát xiên của tấm A).
- Mỗi chỗ uốn vẽ bằng 2 đường DASHED thể hiện độ giãn dài (không vẽ 1 đường ở giữa):
  - Đường 1 cách mép phôi bên này đúng bằng kích thước ngoài của cánh bên này.
  - Đường 2 cách mép phôi bên kia đúng bằng kích thước ngoài của cánh bên kia.
  - Khoảng cách giữa 2 đường = độ giãn của chỗ uốn đó. Ví dụ chữ L 37 + 60, giãn 5.2, phôi 91.8: đường tại 37 và 31.8 (= 91.8 - 60).
  - Với nhiều chỗ uốn (chữ U...): mỗi chỗ uốn là cặp đường đặt tại (vị trí giữa ± độ giãn/2), vị trí giữa = kích thước ngoài của cánh - độ giãn/2 tính từ mép xa.
- Chỗ uốn góc ≤ 85° (cánh nghiêng): không dùng kích thước danh nghĩa (ví dụ 80/150) làm kích thước ngoài.
  - Bán kính trong R phụ thuộc độ dày t: t < 6 giữ R trong = 0.6; t ≥ 6 thì R trong = 6 (thay cho 0.6).
  - Dựng R trong theo trên, offset ra đúng bằng độ dày t để được R ngoài (t3.2: 0.6 + 3.2 = 3.8; t ≥ 6: 6 + t), rồi lấy kích thước ngoài hiệu chỉnh của mỗi cánh từ hình học thực (cánh nghiêng đo dọc mặt ngoài).
  - Giá trị hiệu chỉnh thay đổi theo từng bản vẽ, không phải hằng số. Ví dụ t3.2, R0.6: 80 → 79.65, 150 → 149.65 (lệch 0.35 chỉ là ví dụ).
  - Phôi = ngoài hiệu chỉnh + ngoài hiệu chỉnh - độ giãn. Độ giãn do người dùng cung cấp theo từng yêu cầu, không có quy tắc chung (không tự tra theo góc).
  - Góc > 90°: dùng kích thước danh nghĩa như bình thường (80 + 150 - độ giãn).
  - Áp dụng cho cả Asahi và AICOKU ALPHA. Nếu chưa có độ giãn hoặc không dựng được kích thước hiệu chỉnh thì hỏi lại, không tự suy đoán.
- Hình hộp mở bốn cánh (khay, hộp có 4 chỗ uốn): phôi hình chữ thập, khoét 4 góc. Đặt a = (kích thước ngoài của cánh) - độ giãn, t = độ dày tấm.
  - Khoét góc theo X: bằng a (cánh ở 2 cạnh theo phương Oy giữ nguyên bề rộng đầy đủ, không lùi).
  - Khoét góc theo Y: bằng a + t (2 cánh nằm ở hai đầu theo phương Ox được lùi một đoạn bằng t ở mỗi đầu).
  - Thêm 4 đường rạch ở 4 góc khoét, vẽ màu Green (mã màu DXF 62 = 3), hướng từ ngoài vào:
    - Điểm bắt đầu nằm trong phần khoét (ngoài phôi), cách mỗi mép khoét (2 cạnh tạo góc) một khoảng bằng Piercing ngoài theo bảng `Quy_Tac_Laser.md` mục 2 (ví dụ thép ≤3.2 → 3; thép 4–9 → 6...).
    - Đi chéo 45° tới góc khoét, bo R0.5 tại chỗ chuyển hướng, rồi rạch thẳng dọc theo mép khoét theo phương Ox (mép của cánh đã lùi) vào phía đáy, sâu t + 0.2 tính từ góc khoét.
    - Gồm 3 entity: LINE (điểm bắt đầu → tiếp điểm), ARC R0.5, LINE (tiếp điểm → cuối đường rạch); LINE vẽ từ ngoài vào.
  - Các góc lồi bo theo quy tắc Laser R; góc lõm không bo. Đường uốn DASHED chạy trong phạm vi mỗi cánh.
  - Chi tiết có vách bên (2 vách đối nhau uốn từ hai mép tấm, mỗi vách có chiều dài ngắn hơn chiều dài tấm): cũng rạch góc Green ở 4 góc nối vách–tấm, cùng cấu trúc 3 entity (LINE chéo từ điểm cách 2 mép bằng Piercing ngoài, ARC R0.5, LINE vào tấm sâu t + 0.2 theo phương ngang của mép vách). Ví dụ 26-060234 (t3.2): góc (137.7; 37.7) có LINE (140.7; 34.7) → (137.846; 37.554), ARC tâm (137.493; 37.2) R0.5 từ 45° đến 90°, LINE (137.493; 37.7) → (134.3; 37.7).
  - Ví dụ 26-057140 (SPHC-P t1.6, giãn 2.6, cánh 121.6, phôi 491.2 x 441.2): khoét 119.0 theo X, 120.6 theo Y; đường rạch góc dưới trái: (116, 117.6) → (118.854, 120.454), cung tâm (119.207, 120.1) R0.5 từ 90° đến 135°, (119.207, 120.6) → (120.8, 120.6).
  - Vách đơn (chỉ 1 vách uốn từ một mép tấm, ngắn hơn chiều dài tấm): cũng PHẢI rạch Green ở 2 góc lõm nối vách–tấm, cùng cấu trúc 3 entity (LINE chéo từ điểm cách 2 mép bằng Piercing ngoài, ARC R0.5, LINE vào tấm sâu t + 0.2 dọc theo mép bên của vách, không nhất thiết theo Ox). Mọi góc lõm nơi vách gặp tấm đều phải kiểm tra có rạch hay chưa. Ví dụ 26-060393 (SUS t2, vách 630 trên tấm 700, vách cách 2 đầu 35): góc (35; 60): LINE (29; 66) → (34.854; 60.146), ARC tâm (34.5; 59.793) R0.5 từ 0° đến 45°, LINE (35; 59.793) → (35; 57.8); góc (665; 60): LINE (671; 66) → (665.146; 60.146), ARC tâm (665.5; 59.793) R0.5 từ 135° đến 180°, LINE (665; 59.793) → (665; 57.8).
- Kích thước ghi dạng cộng/trừ (ví dụ "20" và "3.2" cạnh nhau ở hình nghiêng) phải đọc hết từng số rồi mới cộng, không lấy số lớn nhất làm chiều cao (26-060234: chiều cao vách = 20 + 3.2 = 23.2, không phải 30). Sau khi dựng phôi, đối chiếu tổng chiều rộng/chiều dài với công thức "ngoài + ngoài - độ giãn".
- Kích thước trong ngoặc ( ) trên bản vẽ chỉ là tham khảo, không nhất thiết phải chính xác; chỉ cần tuân thủ các kích thước không nằm trong ngoặc. Nếu có file DXF của khách thì hình dạng lấy theo DXF; không cần hỏi lại khi chỉ lệch với số trong ngoặc hoặc bảng CAD依頼書 do số đó.
- Lỗ/gia công bị gạch (bút đỏ hoặc gạch chéo, ví dụ 2-M8, φ12 H7, M5) thì không vẽ; lỗ có ghi 「あける」 thì vẽ.
- Lỗ nhỏ hơn mức tối thiểu của máy laser: so đường kính lỗ (kể cả lỗ mồi taro) với cột "Lỗ nhỏ nhất" trong `Quy_Tac_Laser.md` mục 2 theo nhóm vật liệu và độ dày (ví dụ thép 12t: tối thiểu φ6 nên φ5 không khoét được; thép 16t trở lên: tối thiểu bằng độ dày). Lỗ nhỏ hơn mức này, hoặc bản vẽ ghi PIERCING/ピアス, thì KHÔNG vẽ CIRCLE mà vẽ một POINT (piercing) đặt đúng tâm lỗ, layer 0 (POINT chỉ cần tọa độ 10/20/30). Ví dụ 26-059668/059669 (SS400 t12, 6-M6, ghi PIERCING): 6 POINT tại tâm 6 lỗ mồi φ5, không vẽ vòng tròn. Nếu khách hàng có quy định riêng trong `DS_Khach_Hang.md` (ví dụ khách 875: lỗ nhỏ hơn độ dày làm lỗ trắng) thì theo khách hàng; không tự suy đoán cách biểu diễn khác, nếu chưa rõ thì hỏi.
- Laser R (bo góc ngoài): với hình không có chỉ thị riêng (bản vẽ và danh sách khách hàng đặc biệt không nói gì), PHẢI bo các góc ngoài (góc lồi) có góc ≤ 90° theo bảng `Quy_Tac_Laser.md` (mục 2) theo nhóm vật liệu và độ dày:
  - Thép (SS: SPHC, SPCC, SS400...): dày ≤ 5 → R0.5; 6–9 → R2; 10–16 → không bo; ≥ 19 → R3 cả góc trong và ngoài.
  - Inox/khác: ≤ 6 → R0.5; 7 trở lên → không bo. Nhôm: ≤ 6 → R0.5. Đồng/đồng thau: ≤ 5 → R0.5.
  - Góc đã vát C (135°) và góc lõm (góc trong) không bo (trừ thép ≥ 19t). Nếu khách hàng có quy định riêng (không bo, bo R3...) thì theo khách hàng; bản vẽ có ghi R thì theo bản vẽ.
  - Ghi chú dạng "n-Rx" (ví dụ 2-R3) có thể xuất hiện ở NHIỀU hình chiếu khác nhau, mỗi nơi là n góc riêng. Phải liệt kê mọi ghi chú "n-Rx" trên tất cả hình chiếu của chi tiết, cộng số góc, rồi đối chiếu với số góc ngoài thực tế của biên dạng. Góc nào đã có ghi chú thì bo theo bản vẽ; chỉ góc chưa có ghi chú nào mới áp bảng laser mặc định. Ghi chú "(n-Rx)" trong ngoặc ở hình mặt cắt thường là R uốn, không phải R góc phôi. Ví dụ 26-059946 (SS400 t3.2): 2-R3 ở hình chiếu cánh thứ nhất + 2-R3 ở hình chiếu cánh thứ hai = cả 4 góc phôi R3 (trước đây chỉ bo 2 góc R3, 2 góc còn lại để R0.5 là sai).
  - Vẽ bằng ARC (tròn tiếp tuyến với 2 cạnh), cắt ngắn 2 cạnh đi tại điểm tiếp tuyến. Kích thước ghi trên bản vẽ là kích thước chưa bo.

## File tổng hợp tất cả hình
- Ngoài các file riêng, tạo thêm 1 file DXF chứa tất cả hình, đặt tên `<số đơn>_ALL.dxf` (ví dụ 521008-11_ALL.dxf), lưu cùng thư mục. Khi người dùng yêu cầu chỉ xuất file ALL thì không tạo file riêng từng mã.
- Xếp thành 1 cột dọc, căn trái tại X = 0, các hình cách nhau 200 mm theo trục Y. Thứ tự mã tăng dần từ trên xuống dưới (mã nhỏ nhất ở trên cùng); hình ở dưới cùng đặt sát Y = 0.
- Giữ nguyên chiều quay, hình dạng và màu như file riêng (gồm cả POINT/TEXT tham khảo và kegaki nếu có), chỉ tịnh tiến.
- Mỗi hình có 1 dòng chữ TEXT ghi mã yêu cầu (6 số), cao 12 mm, đặt bên trái, chính giữa chiều cao hình: điểm chèn (đầu chữ, căn trái) tại X = -50, Y = Y đáy hình + (chiều cao hình)/2 (đáy chữ ngang tâm hình), style Standard, layer 0. Cập nhật EXTMAX theo toàn bộ hình + chữ.
- Chữ cùng layer 0 với đường cắt là chủ ý, người dùng đã biết; không cần nhắc lại.

## File mẫu Template.dxf (BẮT BUỘC dùng làm template)
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

## Quy trình
1. Đọc trang danh sách (CAD依頼書) trong PDF: mã yêu cầu, khách hàng, vật liệu, độ dày, kích thước phôi X/Y. Dùng X/Y này để đối chiếu kết quả (chỉ là tham khảo, bản vẽ mới là chính xác).
2. Đọc tài liệu project bản gọn (tiết kiệm usage): `Quy_Tac_Laser.md` (đọc toàn bộ, ngắn) và `DS_Khach_Hang.md`. Với `DS_Khach_Hang.md` chỉ cần đọc phần "Tra nhanh theo nhóm quy tắc" và dòng của đúng mã khách trong đơn (dùng `project_search` theo mã khách, không đọc cả file nếu không cần). Áp dụng quy tắc theo mã khách (bao gồm quy tắc bo góc). 
3. Độ giãn: ưu tiên số ghi tay trên bản vẽ (VD: "37+60-5.2=91.8"). Nếu không có thì tra bảng `Bang_Do_Gian_Asahi.xlsx` (khách 646 dùng `Bang_Do_Gian_AICOKU ALPHA.xlsx`).
4. Với mỗi trang bản vẽ (tiết kiệm token, mỗi ảnh xem tốn nhiều): trang danh sách chỉ cần render ≤100 dpi. Trước khi render trang bản vẽ, kiểm tra hướng trang (`pdftotext` hoặc ảnh nhỏ ≤50 dpi: khung tên/chữ có nằm ngang không); nếu bản vẽ nằm xoay 90° thì xoay ngay khi render, cắt vùng chứa hình, rồi render MỘT lần ở 200 dpi và xem ảnh đó (không xem ảnh dpi thấp chưa xoay vì chữ/kích thước không đọc được, phải làm lại). Xem từng hình chiếu (三角法 - third angle) để xác định cạnh uốn, lỗ, vát C. Trong lúc xem, ghi lại mọi ghi chú "n-Rx" ở từng hình chiếu (xem quy tắc Laser R). Nếu thư mục có file DXF của khách (3D*.DXF...) thì stage về đọc để lấy hình dạng/vị trí lỗ chính xác.
5. Dựng phôi trong hệ tọa độ cục bộ (u dọc đường uốn, v vuông góc), sau đó xoay 90° (không lật) nếu cạnh v dài hơn, rồi chọn hướng theo quy tắc "Hướng xoay" ở phần Yêu cầu đầu ra.
   - Đường uốn: 2 đường DASHED theo quy tắc ở phần Yêu cầu đầu ra.
   - Hình hộp 4 cánh: khoét góc, lùi cánh và đường rạch Green theo quy tắc hình hộp.
   - Phôi Oy ≤ 100 và Ox ≥ 1000: thêm POINT + TEXT (J, LJ cao 10) trên mép trên, số J và vị trí chia đều theo chiều dài Ox (xem quy tắc ở phần Yêu cầu đầu ra).
   - Kegaki: xác định quan hệ lắp ráp (tấm nào hàn lên mặt nào) từ các hình chiếu, vẽ vết hàn màu Đỏ và chọn mặt vẽ là mặt kegaki (xem quy tắc Kegaki).
   - Tọa độ lỗ/chi tiết lấy theo chuỗi kích thước (xem quy tắc Vị trí lỗ); kiểm tra lại bằng số đo trên ảnh, lệch > 0.5 mm thì xem lại chuỗi kích thước. Nhóm lỗ không có kích thước vuông góc: lấy từ DXF gốc của khách, không suy từ hàng lỗ khác.
   - Vị trí lỗ trên cánh: giữ khoảng cách đo từ mép xa của cánh. Lỗ trên phần đáy chữ U: giữ khoảng cách so với tâm đáy.
   - Vát C ở góc: vẽ thành đoạn chéo trên phôi.
   - Lỗ taro: tra đường kính lỗ mồi trong `Quy_Tac_Lo_Moi.md` (ví dụ M6 → φ5, M8 → φ6.8). Sau đó so với "Lỗ nhỏ nhất" theo độ dày: nhỏ hơn thì vẽ POINT piercing thay vì CIRCLE (xem quy tắc "Lỗ nhỏ hơn mức tối thiểu").
   - Bo góc ngoài ≤ 90° theo quy tắc Laser R ở phần Yêu cầu đầu ra: trước hết áp các ghi chú n-Rx trên bản vẽ (đối chiếu số góc), góc còn lại mới áp bảng laser (vẽ sau khi đã dựng đủ biên dạng, trước khi xoay).
6. Xuất DXF theo template, so kích thước phôi với bảng CAD依頼書, kiểm tra cạnh thẳng áp cả Ox và Oy, kiểm tra số góc bo đủ so với ghi chú n-Rx, render preview để kiểm tra, đối chiếu số entity với dạng hợp lý (cung nửa tròn là 1 ARC, các cung tiếp tuyến cùng tâm cùng bán kính phải gộp lại, không tách thành 2 ARC), kiểm tra mọi góc lõm vách–tấm đã có đường rạch Green.
7. Tạo thêm file tổng hợp theo phần "File tổng hợp tất cả hình".
8. Báo cáo: danh sách file, kích thước phôi, các giả định (nêu rõ vị trí nào suy ra gián tiếp), kegaki đã vẽ, mã yêu cầu nào thiếu bản vẽ.