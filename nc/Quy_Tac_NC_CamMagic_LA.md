> Trích từ `skills/nc-sheetmetal/SKILL.md` (bỏ phần frontmatter và script). Nguồn chuẩn là file SKILL.md.

# NC_SheetMetal – Tạo NC cắt laser cho CamMagic LA

Định dạng lấy theo các file NC đúng của người dùng. Script bên dưới đã kiểm tra tạo ra file giống hệt từng byte với:
- 26-059090.nc (SPHCP t3.2, vát C, lỗ tròn)
- 26-059097.nc (SPHCP t1.6, góc bo R0.5)
- 26-057027.nc (SPHCP t1.6, lỗ tròn + lỗ dài ngang 16×8, góc bo R0.5)
- 26-057140.nc (SPHCP t1.6, biên chữ thập bo R0.5 + 4 khe cắt ở góc trong)
- 26-057101.nc (SPHCP t3.2, phôi dài hẹp 1199.5×99.5, 5 lỗ φ9, piercing ngoài tại marker LJ/J)
- 26-058543.nc, 26-058544.nc (SPHCP t6, vạch dấu/kegaki đường hở + lỗ dài đứng R4.5)

## Đầu vào
- File DXF trải phôi (đầu ra của skill dxf-sheetmetal): biên ngoài LINE + ARC (laser R), lỗ tròn CIRCLE, lỗ dài/lỗ hình là vòng kín LINE + ARC bên trong, khe cắt (スリット) là đường hở LINE + ARC, vạch dấu/kegaki là LINE + ARC màu đỏ (ACI color 62 = `1`), đường uốn là LINE linetype DASHED (không cắt). Marker TEXT "J" / "LJ" (kèm POINT) chỉ có ở phôi dài hẹp, không cắt.
- File PDF yêu cầu (CAD依頼書 + bản vẽ).

## Quy trình
1. Nếu người dùng nêu số đơn / folder: đường dẫn thật là `C:\Users\ADMIN\OneDrive\Desktop\SheetMetal\AI_Work\<số đơn>` ("Máy tính" là tên hiển thị của `Desktop`). Xin quyền folder, liệt kê, stage các file `26-*.dxf` và `<số đơn>.pdf`.
2. Đọc trang CAD依頼書 trong PDF (bản scan: `pdftoppm -r 100 -f 1 -l 1` rồi xem ảnh). Với **từng mã riêng biệt**: lấy **vật liệu**, **độ dày (cột 板厚)**, mã khách hàng, kích thước X/Y để đối chiếu khung bao DXF. **Không dùng chung một độ dày mặc định cho cả đơn** — mỗi mã trong CAD依頼書 có thể khác độ dày (đã từng sai ở đơn 521006-13: dùng t≤3.2 cho toàn bộ 11 mã trong khi thực tế là 4.5/6/9mm theo từng mã).
3. Vật liệu chỉ dùng để xác định **nhóm**: tra mục 7 của `Quy_tac_laser_kim_loai_tam.md` (VD: SPHCP = No205, SS400 → SS; 2B, HL, 304 → SUS; A5052 → AL; đồng/đồng thau → CU). "SPHCP t3.2" = nhóm SS, dày 3.2.
4. Xem trang bản vẽ của mã khi cần kiểm tra lỗ (taro M6 → lỗ mồi φ5, lỗ khoan φ...), vát C, ghi chú. Đọc `DS_khach_hang_yeu_cau_dac_biet.md` theo mã khách hàng; áp dụng nếu có quy định ảnh hưởng tới NC (joint, lỗ trắng, vạch dấu, bo R, xác nhận khi DXF khác bản vẽ...). VD khách 851 (合資会社メイナン): cho phép biến dạng lỗ do uốn, các vị trí hàn cần kẻ dấu ngay cả khi không có chỉ thị — giải thích vì sao DXF có đường vạch dấu/kegaki.
5. Kiểm tra DXF trước khi chạy:
   - Khung bao khớp X/Y của CAD依頼書 (lệch thì báo; khách yêu cầu xác nhận thì không tự làm).
   - Laser R: góc ngoài ≤ 90° phải đã được bo theo bảng (góc vát C 135° và góc lõm không bo). DXF thiếu R bắt buộc thì báo người dùng.
   - **Hướng hình**: phải có cạnh thẳng đứng sát Oy (X = 0) và cạnh sát Ox (Y = 0). Nếu tại X nhỏ nhất chỉ là một đỉnh (VD 26-057025: cánh chéo chĩa sang trái) thì xoay DXF 90°/180°/270° (không lật) cho cạnh thẳng nằm bên trái, tịnh tiến về 0,0, lưu lại DXF (chỉ sửa tọa độ trong ENTITIES; ARC cộng góc 50/51) rồi mới tạo NC. Script dừng với thông báo nếu không có cạnh trái thẳng đứng.
6. Ghi script bên dưới ra file rồi chạy:
   `python3 cammagic_nc.py <file.dxf> <nhóm SS|SUS|AL|CU> <độ dày> [file_ra.nc]`
   VD: `python3 cammagic_nc.py 26-059090.dxf SS 3.2`. Script tự tra piercing ngoài / joint ngoài / piercing lỗ theo bảng.
7. Tên file ra `<mã>.nc` (VD `26-059090.nc`), CRLF. Copy vào `/mnt/user-data/outputs/` rồi ghi vào folder của đơn bằng device_commit_files (nếu bị từ chối "fetch or write failed" thì thử lại một lần; "open and locked" thì nhờ người dùng đóng file). Không có folder thì gửi file cho người dùng.
8. Báo cáo ngắn dạng bảng: mã, vật liệu/dày, phôi, piercing/joint, vị trí piercing ngoài; liệt kê mã chưa làm được và lý do (thiếu DXF, lệch kích thước, cần mẫu NC), giả định cần xác nhận.

## Định dạng NC CamMagic LA (bắt buộc)
- ASCII, xuống dòng CRLF, dòng cuối không có CRLF.
- Số bỏ 0 thừa, số nguyên vẫn có dấu chấm: `21.`, `0.`, `-3.`, `46.8`. Không có dấu cách giữa các từ lệnh: `G01G41X18.5Y46.8`.
- `#501=106` luôn giữ cố định, không cần quan tâm.

```
(#_)%(#_)
(<mã>.DXF / <rộng X kể cả piercing, 1 số lẻ> X <cao Y, 1 số lẻ>)
#501=106
M66
G90
G92X0.Y0.
--- mỗi đường cắt ---
G00X..Y..G40          ; điểm đục
M98P9010              ; mã cắt (lỗ trắng 9016, vạch dấu 9017)
G01G41X..Y..          ; dẫn vào (khe cắt / vạch dấu: không có dòng này, không bù G41)
... G01 / G02 / G03 ...  ; cắt
M121
M199
--- kết thúc ---
G01G40
M30
(CUTTING_LENGTH = %8.1fMM  RAPID_LENGTH = %8.1fMM)
(TOTAL_LENGTH = %8.1fMM)
(BEAM_ON = n  KEGAKI_ON = 0  PREPIERCE_ON = 0)
(#_)%(#_)
```

## Quy tắc đường chạy
- Thứ tự: **vạch dấu (kegaki) → khe cắt → lỗ → biên ngoài sau cùng**. Chi tiết vừa có khe vừa có lỗ thì luôn cắt hết khe trước rồi mới cắt lỗ (người dùng đã xác nhận).
- **Thứ tự lỗ: theo diện tích từ nhỏ nhất đến lớn nhất**, lỗ tròn và lỗ dài xếp chung (lỗ dài = πr² + 2r × đoạn thẳng). Cùng diện tích thì chọn lỗ có điểm đục gần vị trí hiện tại nhất, bắt đầu từ gốc (0,0).
- **Lỗ tròn**: dẫn vào điểm bên trái lỗ (cx−r, cy), cắt G03 một vòng, chỉ ghi `I` = r.
  - Bán kính < piercing lỗ → đục tại tâm (chỉ áp dụng cho lỗ tròn).
  - Ngược lại → điểm đục nằm trong lỗ, cách điểm vào đúng chiều dài piercing lỗ.
- **Lỗ dài / lỗ hình** (vòng kín LINE + ARC): cắt ngược chiều kim đồng hồ, bù G41. **Xác định trục dài của lỗ** (so bề rộng X và chiều cao Y của khung bao lỗ):
  - Lỗ nằm ngang (rộng ≥ cao): điểm vào = điểm **trái nhất** (lỗ dài nằm ngang: giữa nửa tròn trái). Điểm đục cách điểm vào theo phương ngang, vào trong lỗ.
  - Lỗ nằm đứng (cao > rộng): điểm vào = điểm **thấp nhất** (giữa nửa tròn/đoạn ngang dưới cùng). Điểm đục cách điểm vào theo phương đứng, vào trong lỗ.
  - Khoảng cách điểm đục–điểm vào: bằng bán kính cung đầu nếu nhỏ hơn piercing lỗ (đục tại tâm cung đầu), ngược lại bằng piercing lỗ (tối đa nửa kích thước lỗ theo trục dài).
  - Mẫu lỗ ngang 26-057027 (lỗ dài 16×8):
    `G00X513.5Y404.72G40` / `M98P9010` / `G01G41X509.5Y404.72` / `G03X513.5Y400.72I4.` / `G01X521.5Y400.72` / `G03X521.5Y408.72J4.` / `G01X513.5Y408.72` / `G03X509.5Y404.72J-4.` / `M121` / `M199`
  - Mẫu lỗ đứng 26-058543 (lỗ dài đứng R4.5, rộng9×cao10):
    `G00X28.Y7.G40` / `M98P9010` / `G01G41X28.Y2.5` / `G03X32.5Y7.J4.5` / `G01X32.5Y17.` / `G03X23.5Y17.I-4.5` / `G01X23.5Y7.` / `G03X28.Y2.5I4.5` / `M121` / `M199`
- **Vạch dấu / kegaki** (LINE/ARC màu đỏ, ACI color 62 = `1`): cắt bằng `M98P9017` (thay vì 9010), **không bù G41, không đoạn dẫn vào riêng** — đục ngay tại điểm bắt đầu rồi cắt thẳng. Cắt **trước tất cả** (trước khe, lỗ, biên ngoài). Nhiều cụm thì xếp theo khoảng cách từ gốc tuyệt đối (0,0) đến điểm đục, tăng dần.
  - **Vòng kín**: đục tại đỉnh gần gốc tuyệt đối (0,0) nhất, chạy chiều kim đồng hồ (giống quy ước biên ngoài), khép kín về điểm đầu.
  - **Đường hở** (2 đầu tự do): mỗi đầu được kéo dài ra ngoài biên của chi tiết một đoạn `KEGAKI_OFFSET = 3.0` (giả định cố định, đã khớp 3 mẫu có piercing_ngoai=6; **chưa xác nhận có tỉ lệ theo piercing ngoài hay không** — nếu gặp phôi khác piercing_ngoai mà kegaki trông lệch thì hỏi lại người dùng), theo hướng vuông góc với cạnh biên mà đầu đó chạm tới (trái/phải/trên/dưới khung bao biên ngoài). Chọn đầu có điểm kéo dài gần gốc tuyệt đối (0,0) hơn làm điểm đục, dẫn thẳng (không bù) tới đầu đường gốc rồi chạy tới đầu tự do còn lại.
  - Điểm đục/kéo dài của kegaki cũng tính vào khung bao `(<mã>.DXF / rộng X cao Y)` ở dòng tiêu đề.
  - Mẫu đường hở 26-058543: `G00X5.Y-3.G40` / `M98P9017` / `G01X5.Y0.` / `G01X5.Y16.5` / `G01X11.Y16.5` / `G01X11.Y0.` / `M121` / `M199`
  - Đã khớp byte-từng-byte với 4/5 mẫu thực tế (26-058543, 26-058544 khớp hoàn toàn; 26-058546 và 26-058551 khớp trừ hai lệch nhỏ không liên quan kegaki, xem mục dưới). Mẫu 26-058549 không khớp cả hình dạng lẫn kích thước khung bao so với DXF gốc của chính nó — nghi DXF/NC mẫu bị gửi nhầm, cần hỏi lại người dùng nếu gặp lại tình huống này.
- **Khe cắt (スリット)** (đường hở, VD khe giảm ứng suất ở góc trong khi uốn): cắt **sau vạch dấu, trước lỗ và biên ngoài**. Đục tại đầu khe nằm ngoài chi tiết (phía phế liệu), **không G41, không đoạn dẫn vào**, chạy theo đường khe tới đầu kia. Nhiều khe: chọn khe có điểm đục gần vị trí hiện tại nhất (từ gốc 0,0; sau mỗi khe tính từ điểm cuối khe). Cung trong khe ghi đủ I và J. Mẫu 26-057140:
  `G00X116.Y117.6G40` / `M98P9010` / `G01X118.854Y120.454` / `G02X119.207Y120.6I0.354J-0.354` / `G01X120.8Y120.6` / `M121` / `M199`
- **Biên ngoài**: chiều kim đồng hồ, bù G41. Điểm đục nằm ngoài biên đúng chiều dài piercing ngoài, dẫn vào vuông góc với cạnh. Cắt dừng trước điểm vào một đoạn bằng joint ngoài.
  - **Mặc định: cạnh trái** — điểm đục trên cạnh trái (đoạn thẳng đứng dài nhất ở X nhỏ nhất), dẫn vào theo phương ngang.
  - **QUY TẮC MỚI (14/10/2026, người dùng yêu cầu; ví dụ 26-060094 phôi 250 × 9):** nếu đoạn thẳng theo Oy (cạnh trái) dài < 10 (không tính vát C / bo R) thì đục piercing ngoài tại **cạnh nằm trên trục Ox** (đoạn thẳng dài nhất ở Y nhỏ nhất): điểm vào cách đầu trái của đoạn thẳng 40 (nếu đoạn thẳng ngắn hơn 80 thì lấy giữa đoạn — giả định của Claude, chưa được xác nhận), điểm đục nằm dưới Ox đúng chiều dài piercing ngoài (Y = −piercing), dẫn vào vuông góc lên cạnh, cắt về phía trái (chiều kim đồng hồ), dừng cách điểm vào một đoạn joint ngoài. Chiều cao ở dòng tiêu đề tính cả điểm đục (26-060094: 250 × 15; NC: `G00X42.Y-6.G40` / `M98P9010` / `G01G41X42.Y0.` / `G01X2.Y0.` … / `G01X43.Y0.`).
- **Phôi dài hẹp (Y ≤ 100 và X ≥ 1000) có marker J / LJ** (TEXT "J" và "LJ" + POINT nằm trên cạnh biên; mẫu 26-057101.nc, SPHCP t3.2): không dùng piercing cạnh trái. Biên ngoài tách thành **2 đường cắt**, đều theo chiều kim đồng hồ, **LJ trước, J sau**.
  - Điểm đục = marker dịch ra ngoài theo pháp tuyến cạnh một đoạn piercing ngoài (SS ≤3.2: 3 → Y102.5 khi marker ở Y99.5); dẫn vào vuông góc `G01G41` tới marker.
  - Khoảng hở (micro joint) tại mỗi marker = **joint ngoài tra theo bảng vật liệu/độ dày, không cố định 0.8** (SS ≤3.2: 0.8; SUS/AL ≤8 và CU: 0.7; SS 4–16, SUS/AL ≥9: 1).
  - Đường 1 bắt đầu tại LJ, chạy cả biên, dừng cách J một đoạn bằng joint ngoài (SS ≤3.2: X29.2 khi J ở X30). Đường 2 bắt đầu tại J, chạy dọc cạnh, dừng cách LJ một đoạn joint ngoài (SS ≤3.2: X1168.7 khi LJ ở X1169.5). Đường 2 kết thúc `G01G40` / `M30`.
  - Đầu dòng 2 ghi `1199.5 X 102.5` (chiều cao tính cả điểm đục ngoài).
  - Mẫu (SS t3.2): `G00X1169.5Y102.5G40` / `M98P9010` / `G01G41X1169.5Y99.5` / `G01X1194.5Y99.5` / `G02X1199.5Y94.5J-5.` … `G01X29.2Y99.5` / `M121` / `M199` / `G00X30.Y102.5G40` / `M98P9010` / `G01G41X30.Y99.5` / `G01X1168.7Y99.5` / `M121` / `M199` / `G01G40` / `M30`.
- **Vị trí Y của piercing ngoài** (cạnh trái) theo chiều dài L của đoạn thẳng cạnh trái (không tính phần vát C / bo R):
  - L ≤ 100 → giữa cạnh. VD 26-059090: cạnh Y5–Y86.8 → Y45.9.
  - L > 100 → cách đầu dưới của cạnh 40. VD 26-059091: cạnh Y5–Y131 → Y45; 26-059097: cạnh Y0.5–Y347.42 → Y40.5.
  - L < 10 → dùng quy tắc mới ở trên (đục tại cạnh trên Ox).
- **Góc bo laser R (ARC)** trên biên: cung theo chiều kim đồng hồ ghi `G02`, ngược chiều ghi `G03`; `I`, `J` = tâm − điểm đầu cung, thành phần bằng 0 thì bỏ (làm tròn 3 số lẻ trước khi xét bằng 0). VD 26-059097: `G01X0.Y347.42` / `G02X0.5Y347.92I0.5` / `G01X1674.5Y347.92` / `G02X1675.Y347.42J-0.5`.
- Đường uốn (DASHED) không cắt.
- CUTTING_LENGTH tính cả đoạn dẫn vào, cung tính theo chiều dài cung. RAPID_LENGTH tính từ gốc (0,0) qua các điểm đục, không tính quay về. BEAM_ON = số lần `M98P9010` + `M98P9017`.

## Bảng piercing / joint (theo Quy_tac_laser_kim_loai_tam.md mục 2)
| Nhóm | Độ dày | Piercing ngoài | Joint ngoài | Piercing lỗ |
|---|---|---|---|---|
| SS | ≤3.2 | 3 | 0.8 | 6 |
| SS | 4–9 | 6 | 1 | 6 |
| SS | 10–16 | 10 | 1 | 10 |
| SUS | ≤8 | 6 | 0.7 | 6 |
| SUS | 9–16 | 10 | 1 | 10 |
| AL | ≤8 | 6 | 0.7 | 6 |
| AL | 9–18 | 10 | 1 | 10 |
| CU | mọi độ dày | 6 | 0.7 | 6 |
Nếu `Quy_tac_laser_kim_loai_tam.md` được cập nhật khác bảng này thì theo file quy tắc và sửa `TABLE` trong script.

## Lệch nhỏ đã biết, chưa có quy tắc tổng quát (không chặn sản xuất)
- 26-058546.nc: tâm lỗ tròn φ19 trong mẫu lệch Y +0.5mm so với tâm thực trong DXF (18. thay vì 17.5) — có vẻ là chỉnh tay riêng lẻ trong file mẫu, script vẫn dùng tâm DXF.
- 26-058551.nc: 1 tọa độ vạch dấu trong mẫu là `44.32` trong khi DXF có giá trị `44.319334` (fmt() làm tròn 3 số lẻ ra `44.319`) — chênh ~0.0007mm, không ảnh hưởng gia công.

## Giới hạn hiện tại
- Script dừng và báo khi gặp: POLYLINE/SPLINE, không có cạnh trái thẳng đứng, độ dày ≥19 (piercing cong), lỗ ≥ φ50 trên SUS/AL/CU (cần joint lỗ).
- Phôi Y ≤ 100 và X ≥ 1000 nhưng DXF không có marker J/LJ: script chỉ cảnh báo và dùng piercing cạnh trái thường → hỏi lại người dùng. Marker J/LJ nằm trên cạnh không phải cạnh dài trên: chưa có mẫu.
- Đường vạch dấu (kegaki) hở với đầu không chạm biên ngoài: chưa rõ hướng đục/kéo dài, chưa có mẫu.
- Chưa có mẫu, không tự đoán cú pháp: lỗ trắng (9016), joint lỗ, lỗ vuông → báo người dùng và xin file NC mẫu.

## Script
Xem `nc/cammagic_nc.py` (tách từ cuối `skills/nc-sheetmetal/SKILL.md`).
