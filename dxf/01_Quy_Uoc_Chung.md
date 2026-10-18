# Quy ước chung: tên file, công thức, tọa độ, hướng xoay, phôi dài hẹp

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 7–20). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

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
