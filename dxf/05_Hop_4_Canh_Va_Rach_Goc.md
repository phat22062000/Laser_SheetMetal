# Hộp mở bốn cánh, vách bên và đường rạch góc Green

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 41–51). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

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
