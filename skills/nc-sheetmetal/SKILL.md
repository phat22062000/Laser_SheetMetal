---
name: "nc-sheetmetal"
description: "Tạo chương trình NC cắt laser định dạng CamMagic LA (Mitsubishi) từ file DXF trải phôi kim loại tấm, tra piercing/joint theo nhóm vật liệu và độ dày."
---

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

## Script cammagic_nc.py
```python
"""Tạo NC cắt laser định dạng CamMagic LA (Mitsubishi) từ DXF trải phôi.
Hỗ trợ biên ngoài LINE + ARC (laser R); lỗ CIRCLE, lỗ dài/lỗ hình (vòng kín LINE + ARC, ngang/đứng), khe cắt (đường hở).
Phôi Y<=100 & X>=1000 có marker TEXT "J" và "LJ" trên biên: đục ngoài tại LJ và J, micro joint tại 2 vị trí này.
Vạch dấu/kegaki: LINE/ARC màu đỏ (ACI color 62='1'), cắt M98P9017, không bù.
Dùng: python3 cammagic_nc.py <file.dxf> <nhóm SS|SUS|AL|CU> <độ dày> [file_ra.nc]
"""
import math, sys, os

def fmt(v):
    v = round(v, 3)
    if abs(v) < 1e-9: v = 0.0
    s = ("%.3f" % v).rstrip('0')          # 21.000 -> "21." ; 46.800 -> "46.8"
    return s

MARKS = {}
KEGAKI = []
KEGAKI_OFFSET = 3.0   # khoảng cách đục ra ngoài biên cho kegaki hở (giả định, xem Quy_tac_NC_CamMagic_LA.md)

def read_dxf(path):
    raw = open(path, 'rb').read().decode('cp932', errors='replace').replace('\r\n', '\n').split('\n')
    pairs = [(raw[i].strip(), raw[i+1].strip()) for i in range(0, len(raw) - 1, 2)]
    ents, cur, on = [], None, False
    for i, (c, v) in enumerate(pairs):
        if c == '2' and v == 'ENTITIES' and pairs[i-1] == ('0', 'SECTION'):
            on = True; continue
        if not on: continue
        if c == '0':
            if cur: ents.append(cur)
            if v == 'ENDSEC': break
            cur = {'type': v}
        elif cur is not None:
            cur.setdefault(c, v)
    segs = [{'t': 'L', 'p': (float(e['10']), float(e['20'])), 'q': (float(e['11']), float(e['21']))}
            for e in ents if e['type'] == 'LINE' and e.get('6', '').upper() != 'DASHED' and e.get('62', '') != '1']
    global KEGAKI   # đường màu đỏ (ACI 62=1) = đường vạch dấu (kegaki, M98P9017), không bù G41
    KEGAKI = [{'t': 'L', 'p': (float(e['10']), float(e['20'])), 'q': (float(e['11']), float(e['21']))}
              for e in ents if e['type'] == 'LINE' and e.get('62', '') == '1']
    for e in ents:   # ARC (laser R): ngược chiều kim đồng hồ từ góc 50 đến góc 51
        if e['type'] == 'ARC':
            cx, cy, r = float(e['10']), float(e['20']), float(e['40'])
            a0, a1 = math.radians(float(e['50'])), math.radians(float(e['51']))
            arc = {'t': 'A', 'c': (cx, cy), 'r': r,
                   'p': (cx + r*math.cos(a0), cy + r*math.sin(a0)),
                   'q': (cx + r*math.cos(a1), cy + r*math.sin(a1)),
                   'sweep': (a1 - a0) % (2*math.pi)}
            if e.get('62', '') == '1': KEGAKI.append(arc)
            else: segs.append(arc)
    circles = [(float(e['10']), float(e['20']), float(e['40'])) for e in ents if e['type'] == 'CIRCLE']
    global MARKS   # TEXT 'J' / 'LJ' (micro joint / piercing ngoài cho phôi dài, hẹp)
    MARKS = {e['1'].strip().upper(): (float(e['10']), float(e['20'])) for e in ents
             if e['type'] == 'TEXT' and e.get('1', '').strip().upper() in ('J', 'LJ')}
    if any(e['type'] in ('LWPOLYLINE', 'POLYLINE', 'SPLINE') for e in ents):
        sys.exit("Biên dạng có POLYLINE/SPLINE: script hiện chỉ hỗ trợ LINE + ARC + CIRCLE")
    return segs, circles

def chain(segs):
    """Nối LINE/ARC thành biên kín, trả về danh sách đoạn có hướng (CW):
    {'t','p','q'(+ 'c','r','ccw' cho ARC)}"""
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    def orient(sg, rev):
        d = dict(sg)
        if rev:
            d['p'], d['q'] = sg['q'], sg['p']
            if sg['t'] == 'A': d['ccw'] = False
        elif sg['t'] == 'A': d['ccw'] = True
        return d
    out, used = [orient(segs[0], False)], {0}
    while len(used) < len(segs):
        for i, sg in enumerate(segs):
            if i in used: continue
            if key(sg['p']) == key(out[-1]['q']): out.append(orient(sg, False)); used.add(i); break
            if key(sg['q']) == key(out[-1]['q']): out.append(orient(sg, True)); used.add(i); break
        else:
            sys.exit("Biên dạng ngoài bị hở")
    if key(out[0]['p']) != key(out[-1]['q']): sys.exit("Biên dạng ngoài không kín")
    def poly(sg):
        if sg['t'] == 'L': return [sg['p']]
        cx, cy = sg['c']; a0 = math.atan2(sg['p'][1]-cy, sg['p'][0]-cx)
        sw = sg['sweep'] if sg['ccw'] else -sg['sweep']
        return [(cx + sg['r']*math.cos(a0 + sw*k/4), cy + sg['r']*math.sin(a0 + sw*k/4)) for k in range(4)]
    pts = [q for sg in out for q in poly(sg)]
    area = sum(pts[i][0]*pts[(i+1) % len(pts)][1] - pts[(i+1) % len(pts)][0]*pts[i][1] for i in range(len(pts))) / 2
    if area > 0:   # đảo sang chiều kim đồng hồ
        out = [orient({**sg, 'p': sg['p'], 'q': sg['q']}, True) if sg['t'] == 'L' else
               {**sg, 'p': sg['q'], 'q': sg['p'], 'ccw': not sg['ccw']} for sg in reversed(out)]
        out = [dict(sg, p=sg['p'], q=sg['q']) for sg in out]
    return out

# (piercing ngoài, joint ngoài, piercing lỗ) theo Quy_tac_laser_kim_loai_tam.md mục 2
TABLE = {
    'SS':  [(3.2, (3, 0.8, 6)), (5, (6, 1, 6)), (9, (6, 1, 6)), (16, (10, 1, 10))],
    'SUS': [(8, (6, 0.7, 6)), (16, (10, 1, 10))],
    'AL':  [(8, (6, 0.7, 6)), (18, (10, 1, 10))],
    'CU':  [(99, (6, 0.7, 6))],
}

def params(group, t):
    for tmax, p in TABLE[group.upper()]:
        if t <= tmax + 1e-9: return p
    sys.exit("Độ dày %s của nhóm %s dùng piercing cong (≥19t): script chưa hỗ trợ" % (t, group))

def group_loops(segs):
    """Tách các đoạn LINE/ARC thành các vòng kín riêng (biên ngoài + lỗ dài/lỗ hình)."""
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    par = list(range(len(segs)))
    def f(i):
        while par[i] != i: par[i] = par[par[i]]; i = par[i]
        return i
    first = {}
    for i, g in enumerate(segs):
        for k in (key(g['p']), key(g['q'])):
            if k in first: par[f(i)] = f(first[k])
            else: first[k] = i
    grp = {}
    for i in range(len(segs)): grp.setdefault(f(i), []).append(segs[i])
    return list(grp.values())

def is_open(L):
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    deg = {}
    for g in L:
        for k in (key(g['p']), key(g['q'])): deg[k] = deg.get(k, 0) + 1
    return any(v == 1 for v in deg.values())

def inside(pt, loop):
    """Điểm nằm trong biên ngoài (đa giác xấp xỉ, cung lấy thêm điểm giữa)."""
    pts = []
    for g in loop:
        pts.append(g['p'])
        if g['t'] == 'A':
            cx, cy = g['c']; a0 = math.atan2(g['p'][1]-cy, g['p'][0]-cx)
            sw = g['sweep'] if g['ccw'] else -g['sweep']
            pts.append((cx + g['r']*math.cos(a0+sw/2), cy + g['r']*math.sin(a0+sw/2)))
    x, y = pt; c = False
    for i in range(len(pts)):
        (x1, y1), (x2, y2) = pts[i], pts[(i+1) % len(pts)]
        if (y1 > y) != (y2 > y) and x < x1 + (y-y1)*(x2-x1)/(y2-y1): c = not c
    return c

def order_open(L, outer):
    """Khe cắt (đường hở): bắt đầu ở đầu nằm ngoài chi tiết (phía phế liệu), trả về dãy đoạn có hướng."""
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    deg = {}
    for g in L:
        for p in (g['p'], g['q']): deg.setdefault(key(p), []).append(p)
    ends = [v[0] for v in deg.values() if len(v) == 1]
    if len(ends) != 2: sys.exit("Khe cắt không phải 1 đường hở đơn")
    out_e = [e for e in ends if not inside(e, outer)]
    start = out_e[0] if len(out_e) == 1 else ends[0]
    seq, cur, rest = [], start, list(L)
    while rest:
        for g in rest:
            if key(g['p']) == key(cur):
                seq.append(dict(g, ccw=True) if g['t'] == 'A' else dict(g)); cur = g['q']; rest.remove(g); break
            if key(g['q']) == key(cur):
                d = dict(g, p=g['q'], q=g['p'])
                if g['t'] == 'A': d['ccw'] = False
                seq.append(d); cur = g['p']; rest.remove(g); break
        else:
            sys.exit("Khe cắt bị đứt")
    return start, seq

def rev(loop):
    return [dict(g, p=g['q'], q=g['p'], **({'ccw': not g['ccw']} if g['t'] == 'A' else {})) for g in reversed(loop)]

def loop_area(loop):
    """Diện tích có dấu (dương = CCW), tính chính xác cả phần cung."""
    s = 0.0
    for g in loop:
        s += g['p'][0]*g['q'][1] - g['q'][0]*g['p'][1]
    s /= 2
    for g in loop:
        if g['t'] == 'A':
            seg = g['r']**2 / 2 * (g['sweep'] - math.sin(g['sweep']))
            s += seg if g['ccw'] else -seg
    return s

def split_at_left(loop):
    """Xoay vòng (CCW) để bắt đầu tại điểm vào (trái nhất nếu lỗ nằm ngang, thấp nhất nếu lỗ
    đứng - theo trục dài của lỗ); cắt đôi cung/đoạn nếu cần.
    Trả về (danh sách đoạn, điểm bắt đầu, bán kính cung tại điểm đầu hoặc None, trục 0=X/1=Y)."""
    xs = [v for g in loop for v in (g['p'][0], g['q'][0])]
    ys = [v for g in loop for v in (g['p'][1], g['q'][1])]
    ax = 0 if (max(xs) - min(xs)) >= (max(ys) - min(ys)) else 1   # 0=X (ngang), 1=Y (đứng)
    target = math.pi if ax == 0 else -math.pi/2   # trái nhất (ax=0) hoặc thấp nhất (ax=1)
    best = None
    for i, g in enumerate(loop):
        cand = [(g['p'], None)]
        if g['t'] == 'A':
            cx, cy = g['c']; ap = math.atan2(g['p'][1]-cy, g['p'][0]-cx)
            d = (target - ap) % (2*math.pi) if g['ccw'] else (ap - target) % (2*math.pi)
            if 1e-9 < d < g['sweep'] - 1e-9:
                pt = (cx + g['r']*math.cos(target), cy + g['r']*math.sin(target))
                cand.append((pt, d))
        elif ax == 0 and abs(g['p'][0]-g['q'][0]) < 1e-9:
            cand.append((((g['p'][0]+g['q'][0])/2, (g['p'][1]+g['q'][1])/2), 'mid'))
        elif ax == 1 and abs(g['p'][1]-g['q'][1]) < 1e-9:
            cand.append((((g['p'][0]+g['q'][0])/2, (g['p'][1]+g['q'][1])/2), 'mid'))
        for pt, extra in cand:
            k = (round(pt[ax], 6), 0 if extra is not None else 1)
            if best is None or k < best[0]: best = (k, i, pt, extra)
    _, i, pt, extra = best
    g = loop[i]
    if extra is None:
        out = loop[i:] + loop[:i]; r = None
    elif g['t'] == 'A':
        g1 = dict(g, q=pt, sweep=extra); g2 = dict(g, p=pt, sweep=g['sweep']-extra)
        out = [g2] + loop[i+1:] + loop[:i] + [g1]; r = g['r']
    else:
        out = [dict(g, p=pt)] + loop[i+1:] + loop[:i] + [dict(g, q=pt)]; r = None
    return out, pt, r, ax

def on_seg(g, pt, tol=1e-3):
    if g['t'] != 'L': return False
    (x1, y1), (x2, y2) = g['p'], g['q']
    L = math.dist(g['p'], g['q'])
    if L < 1e-9: return False
    t = ((pt[0]-x1)*(x2-x1) + (pt[1]-y1)*(y2-y1)) / L**2
    cx, cy = x1 + t*(x2-x1), y1 + t*(y2-y1)
    return -1e-9 <= t <= 1 + 1e-9 and math.dist(pt, (cx, cy)) < tol

def split_path(sg, M, E, joint, pierce_out):
    """Phoi dai hep (Y<=100, X>=1000) co marker LJ/J tren bien: bien ngoai (CW) tach thanh 2 duong,
    moi duong bat dau tai marker M (duc ngoai, vuong goc voi canh) va dung truoc marker E mot doan joint.
    Tra ve (diem duc, day doan tu M toi diem dung)."""
    iM = next(i for i, g in enumerate(sg) if on_seg(g, M))
    g = sg[iM]
    dx, dy = g['q'][0]-g['p'][0], g['q'][1]-g['p'][1]; ln = math.hypot(dx, dy)
    nrm = (-dy/ln, dx/ln)                                  # phap tuyen ngoai (CW -> ben trai huong chay)
    seq = [dict(g, p=M)] + sg[iM+1:] + sg[:iM] + [dict(g, q=M)]
    seq = [x for x in seq if not (x['t'] == 'L' and math.dist(x['p'], x['q']) < 1e-6)]
    out = []
    for x in seq:                                          # cat toi E
        if on_seg(x, E):
            out.append(dict(x, q=E)); break
        out.append(x)
    last = out[-1]
    if last['t'] != 'L' or math.dist(last['p'], last['q']) <= joint:
        sys.exit("Marker J/LJ: doan cuoi truoc joint khong phai doan thang du dai")
    ux, uy = (last['q'][0]-last['p'][0]), (last['q'][1]-last['p'][1]); l2 = math.hypot(ux, uy)
    last['q'] = (E[0] - ux/l2*joint, E[1] - uy/l2*joint)
    return (M[0] + nrm[0]*pierce_out, M[1] + nrm[1]*pierce_out), out

def chain_open_from(L, start_pt):
    """Nối các đoạn của 1 đường hở thành dãy có hướng, bắt đầu từ start_pt (1 trong 2 đầu hở)."""
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    seq, cur, rest = [], start_pt, list(L)
    while rest:
        for g in rest:
            if key(g['p']) == key(cur):
                seq.append(dict(g, ccw=True) if g['t'] == 'A' else dict(g)); cur = g['q']; rest.remove(g); break
            if key(g['q']) == key(cur):
                d = dict(g, p=g['q'], q=g['p'])
                if g['t'] == 'A': d['ccw'] = False
                seq.append(d); cur = g['p']; rest.remove(g); break
        else:
            sys.exit("Đường vạch dấu (kegaki) bị đứt")
    return seq

def build_kegaki(bbox):
    """Nhóm các đoạn kegaki (màu đỏ) thành từng đường, xác định điểm đục.
    Vòng kín: đục tại đỉnh gần gốc (0,0) nhất, chạy theo chiều kim đồng hồ, không bù, khép kín.
    Đường hở: đục ngoài biên (lệch KEGAKI_OFFSET khỏi cạnh ngoài tại đầu hở), chọn đầu có điểm đục gần gốc (0,0) hơn.
    Trả về list (điểm đục, dãy đoạn để emit(), không bù G41), đã sắp theo khoảng cách điểm đục tới gốc."""
    xmin, xmax, ymin, ymax = bbox
    key = lambda p: (round(p[0], 3), round(p[1], 3))
    out = []
    for L in group_loops(KEGAKI):
        if not is_open(L):
            seq = chain(L)                       # CW, khép kín
            best = min(range(len(seq)), key=lambda i: math.hypot(*seq[i]['p']))
            seq = seq[best:] + seq[:best]
            start = seq[0]['p']
            out.append((start, seq))
        else:
            deg = {}
            for g in L:
                for p in (g['p'], g['q']): deg.setdefault(key(p), []).append(p)
            ends = [v[0] for v in deg.values() if len(v) == 1]
            if len(ends) != 2: sys.exit("Đường vạch dấu (kegaki) không phải 1 đường hở đơn")
            cands = []
            for e in ends:
                if abs(e[0] - xmin) < 1e-6: ext = (xmin - KEGAKI_OFFSET, e[1])
                elif abs(e[0] - xmax) < 1e-6: ext = (xmax + KEGAKI_OFFSET, e[1])
                elif abs(e[1] - ymin) < 1e-6: ext = (e[0], ymin - KEGAKI_OFFSET)
                elif abs(e[1] - ymax) < 1e-6: ext = (e[0], ymax + KEGAKI_OFFSET)
                else: ext = e   # đầu hở không chạm biên ngoài: không rõ hướng đục, không mở rộng
                cands.append((ext, e))
            ext, e0 = min(cands, key=lambda c: math.hypot(*c[0]))
            lead = {'t': 'L', 'p': ext, 'q': e0}
            seq = [lead] + chain_open_from(L, e0)
            out.append((ext, seq))
    out.sort(key=lambda z: math.hypot(*z[0]))
    return out

def emit(segs, prev, body):
    """Ghi G01/G02/G03 cho dãy đoạn có hướng; trả về (độ dài cắt, điểm cuối)."""
    cut = 0.0
    for g in segs:
        q = g['q']
        if g['t'] == 'L':
            body.append("G01X%sY%s" % (fmt(q[0]), fmt(q[1]))); cut += math.dist(prev, q)
        else:   # ARC: G03 ngược chiều kim đồng hồ, G02 cùng chiều; I/J = tâm - điểm đầu (bỏ số 0)
            i_, j_ = g['c'][0]-prev[0], g['c'][1]-prev[1]
            w = ("G03" if g['ccw'] else "G02") + "X%sY%s" % (fmt(q[0]), fmt(q[1]))
            if abs(round(i_, 3)) > 1e-9: w += "I" + fmt(i_)
            if abs(round(j_, 3)) > 1e-9: w += "J" + fmt(j_)
            body.append(w); cut += g['r'] * g['sweep']
        prev = q
    return cut, prev

def build(dxf, group, t):
    pierce_out, joint_out, pierce_hole = params(group, t)
    segs, circles = read_dxf(dxf)
    if group.upper() != 'SS' and any(2*c[2] >= 50 for c in circles):
        sys.exit("Có lỗ ≥ φ50 trên SUS/AL/CU cần joint lỗ: script chưa hỗ trợ")
    loops = group_loops(segs)
    bb = lambda L: (max(max(g['p'][0], g['q'][0]) for g in L) - min(min(g['p'][0], g['q'][0]) for g in L)) * \
                   (max(max(g['p'][1], g['q'][1]) for g in L) - min(min(g['p'][1], g['q'][1]) for g in L))
    opens = [L for L in loops if is_open(L)]          # khe cắt (スリット): đường hở
    loops = [L for L in loops if not is_open(L)]
    loops.sort(key=bb, reverse=True)
    sg = chain(loops[0])                       # biên ngoài (CW)
    slits = [order_open(L, sg) for L in opens]
    holes = []                                 # mỗi lỗ: (diện tích, điểm đục, điểm vào, dãy đoạn CCW)
    for cx, cy, r in circles:
        px = cx if r < pierce_hole else cx - r + pierce_hole   # bán kính < piercing -> đục tâm
        holes.append((math.pi*r*r, (px, cy), (cx-r, cy),
                      [{'t': 'A', 'c': (cx, cy), 'r': r, 'p': (cx-r, cy), 'q': (cx-r, cy), 'sweep': 2*math.pi, 'ccw': True}]))
    for L in loops[1:]:                        # lỗ dài / lỗ hình (LINE + ARC)
        L = chain(L)                           # CW
        L = rev(L)                             # lỗ cắt CCW (G03)
        A = loop_area(L)
        L, st, r, ax = split_at_left(L)
        w = max(max(g['p'][ax], g['q'][ax]) for g in L) - st[ax]
        dpi = r if (r is not None and r < pierce_hole) else min(pierce_hole, w / 2)
        pp = (st[0] + dpi, st[1]) if ax == 0 else (st[0], st[1] + dpi)
        holes.append((abs(A), pp, st, L))

    split_mode = 'J' in MARKS and 'LJ' in MARKS
    _bx = [g['p'][0] for g in sg] + [g['q'][0] for g in sg]; _by = [g['p'][1] for g in sg] + [g['q'][1] for g in sg]
    if not split_mode and max(_by)-min(_by) <= 100 and max(_bx)-min(_bx) >= 1000:
        print("CANH BAO: phoi Y<=100, X>=1000 nhung DXF khong co marker J/LJ - dang dung piercing canh trai thuong", file=sys.stderr)
    xmin = min(min(g['p'][0], g['q'][0]) for g in sg if g['t'] == 'L')
    # điểm vào biên ngoài: trên đoạn thẳng đứng dài nhất nằm ở X nhỏ nhất (cạnh trái)
    cand = [(abs(g['q'][1]-g['p'][1]), i) for i, g in enumerate(sg)
            if g['t'] == 'L' and abs(g['p'][0]-xmin) < 1e-6 and abs(g['q'][0]-xmin) < 1e-6]
    if split_mode: cand = cand or [(0, 0)]
    if not cand:
        sys.exit("Không có cạnh thẳng đứng ở X nhỏ nhất: xoay DXF (90/180/270, không lật) trước khi tạo NC")
    _, i0 = max(cand)
    a, b = sg[i0]['p'], sg[i0]['q']
    d = 1 if b[1] > a[1] else -1                 # hướng chạy trên cạnh trái (CW: đi lên)
    # vị trí piercing ngoài theo chiều dài cạnh trái L:
    #   L <= 100 -> giữa cạnh ; L > 100 -> cách đầu dưới của cạnh 40
    #   L < 10 (quy tắc 14/10/2026) -> đục tại cạnh nằm trên Ox, cách đầu trái của đoạn thẳng 40
    Lr = abs(b[1] - a[1])
    bottom = False
    if Lr < 10 and not split_mode:
        ymin_ = min(g['p'][1] for g in sg)
        cb = [(abs(g['q'][0]-g['p'][0]), k) for k, g in enumerate(sg)
              if g['t'] == 'L' and abs(g['p'][1]-ymin_) < 1e-6 and abs(g['q'][1]-ymin_) < 1e-6]
        if not cb: sys.exit("Không có cạnh thẳng trên Ox để đục piercing ngoài")
        Lb, i0 = max(cb)
        a, b = sg[i0]['p'], sg[i0]['q']
        xl = min(a[0], b[0]); off = min(40.0, Lb / 2)
        start = (xl + off, ymin_); bottom = True
        dx_ = (b[0]-a[0]) / Lb
        end = (start[0] - dx_*joint_out, ymin_)
    else:
        sy = (a[1] + b[1]) / 2 if Lr <= 100 + 1e-9 else min(a[1], b[1]) + 40
        start = (xmin, sy)
        end = (xmin, sy - d*joint_out)
    seq = [sg[(i0+k) % len(sg)] for k in range(1, len(sg))]   # các đoạn sau cạnh vào, kết thúc tại a
    ys = [g['p'][1] for g in sg] + [g['q'][1] for g in sg]
    xs0 = [g['p'][0] for g in sg] + [g['q'][0] for g in sg]

    cut, rapid, beam = 0.0, 0.0, 0
    # lỗ: diện tích nhỏ nhất -> lớn nhất; cùng diện tích thì chọn lỗ có điểm đục gần nhất
    rem = sorted(holes, key=lambda h: h[0])
    order, cur = [], (0.0, 0.0)
    while rem:
        amin = rem[0][0]
        grp = [h for h in rem if abs(h[0] - amin) < 1e-3]
        h = min(grp, key=lambda h: math.dist(cur, h[1]))
        order.append(h); rem.remove(h); cur = h[2]
    body, pos = [], (0.0, 0.0)
    # kegaki (vạch dấu, màu đỏ) trước tiên: không bù, mã M98P9017
    kegaki_list = build_kegaki((min(xs0), max(xs0), min(ys), max(ys))) if KEGAKI else []
    for pp_, seq_ in kegaki_list:
        rapid += math.dist(pos, pp_)
        body += ["G00X%sY%sG40" % (fmt(pp_[0]), fmt(pp_[1])), "M98P9017"]
        c_, end_ = emit(seq_, pp_, body); cut += c_
        body += ["M121", "M199"]
        pos = end_; beam += 1
        xs0.append(pp_[0]); ys.append(pp_[1])
    # khe cắt trước: đục tại đầu ngoài phía phế liệu, không bù, không dẫn vào; gần nhất trước
    rem_s = list(slits); cur_s = (0.0, 0.0)
    while rem_s:
        st_, seq_ = min(rem_s, key=lambda z: math.dist(cur_s, z[0])); rem_s.remove((st_, seq_))
        rapid += math.dist(pos, st_)
        body += ["G00X%sY%sG40" % (fmt(st_[0]), fmt(st_[1])), "M98P9010"]
        c_, end_ = emit(seq_, st_, body); cut += c_
        body += ["M121", "M199"]
        pos = cur_s = end_; beam += 1
    for _, pp_, st, L in order:
        rapid += math.dist(pos, pp_)
        body += ["G00X%sY%sG40" % (fmt(pp_[0]), fmt(pp_[1])), "M98P9010",
                 "G01G41X%sY%s" % (fmt(st[0]), fmt(st[1]))]
        cut += math.dist(pp_, st)
        if len(L) == 1 and L[0]['sweep'] >= 2*math.pi - 1e-9:      # lỗ tròn
            r = L[0]['r']
            body.append("G03X%sY%sI%s" % (fmt(st[0]), fmt(st[1]), fmt(r))); cut += 2*math.pi*r
        else:
            c_, _ = emit(L, st, body); cut += c_
        body += ["M121", "M199"]
        pos = st; beam += 1
    if split_mode:
        paths = [split_path(sg, MARKS['LJ'], MARKS['J'], joint_out, pierce_out),
                 split_path(sg, MARKS['J'], MARKS['LJ'], joint_out, pierce_out)]
        pps = []
        for k, (pp_, seq_) in enumerate(paths):
            M = seq_[0]['p']
            rapid += math.dist(pos, pp_)
            body += ["G00X%sY%sG40" % (fmt(pp_[0]), fmt(pp_[1])), "M98P9010",
                     "G01G41X%sY%s" % (fmt(M[0]), fmt(M[1]))]
            cut += pierce_out
            c_, prev = emit(seq_, M, body); cut += c_
            body += ["M121", "M199"]
            if k == 1: body += ["G01G40", "M30"]
            pos = prev; beam += 1; pps.append(pp_)
        pp = pps[0]
        xs0 = xs0 + [p[0] for p in pps]; ys = ys + [p[1] for p in pps]
    else:
        pp = (start[0], start[1] - pierce_out) if bottom else (xmin - pierce_out, sy)
        if bottom: ys.append(pp[1])
        rapid += math.dist(pos, pp)
        body += ["G00X%sY%sG40" % (fmt(pp[0]), fmt(pp[1])), "M98P9010",
                 "G01G41X%sY%s" % (fmt(start[0]), fmt(start[1]))]
        cut += pierce_out
        body.append("G01X%sY%s" % (fmt(b[0]), fmt(b[1]))); cut += math.dist(start, b)
        c_, prev = emit(seq, b, body); cut += c_
        body.append("G01X%sY%s" % (fmt(end[0]), fmt(end[1]))); cut += math.dist(prev, end)
        body += ["M121", "M199", "G01G40", "M30"]
        beam += 1

    xs = xs0 + ([pp[0]] if not split_mode else [])
    name = os.path.basename(dxf)
    code = name.split('_')[-1].rsplit('.', 1)[0] if '_' in name else name.rsplit('.', 1)[0]
    head = ["(#_)%(#_)",
            "(%s.DXF / %.1f X %.1f)" % (code, max(xs)-min(xs), max(ys)-min(ys)),
            "#501=106", "M66", "G90", "G92X0.Y0."]
    tail = ["(CUTTING_LENGTH = %8.1fMM  RAPID_LENGTH = %8.1fMM)" % (cut, rapid),
            "(TOTAL_LENGTH = %8.1fMM)" % (cut + rapid),
            "(BEAM_ON = %d  KEGAKI_ON = 0  PREPIERCE_ON = 0)" % beam,
            "(#_)%(#_)"]
    return code, "\r\n".join(head + body + tail)

if __name__ == "__main__":
    dxf, group, t = sys.argv[1], sys.argv[2], float(sys.argv[3])
    code, txt = build(dxf, group, t)
    outp = sys.argv[4] if len(sys.argv) > 4 else code + ".nc"
    open(outp, "w", newline="").write(txt)
    print(outp)
```
