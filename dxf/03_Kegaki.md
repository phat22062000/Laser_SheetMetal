# Kegaki (vạch dấu vị trí hàn)

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 24–28). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

- Kegaki (vạch dấu, ケガキ): khi có chỉ thị KEGAKI/ケガキ, hoặc khách hàng yêu cầu vạch dấu vị trí hàn (ví dụ khách 851), vẽ vết của tấm hàn lên mặt tấm (footprint) bằng LINE màu Đỏ (mã màu DXF 62 = 1), layer 0, theo `Quy_tac_laser_gon.md` mục 5.
  - Chỉ vẽ các đoạn của vết nằm bên trong phôi (đoạn trùng mép phôi thì bỏ). Kegaki đặt trên tấm nhận (tấm có mặt bị tấm khác hàn lên); tấm chỉ hàn bằng cạnh vào tấm khác thì không có kegaki.
  - Chi tiết có kegaki: mặt bản vẽ là mặt kegaki (ưu tiên hơn quy tắc mặt ngoài/hình chiếu đứng). Nếu hình chiếu đứng cho thấy mặt đối diện thì lật trái–phải (không đổi chiều dọc).
  - Chi tiết 本図/対称 (R/L): hình dạng và kegaki của 対称 là ảnh gương của 本図.
  - Ví dụ đơn 521006-13: 26-058546 (tấm C) vạch 2 đường tại vị trí tấm B ở mặt sau nên lật trái–phải so với hình chiếu đứng; 26-058551 (tấm B t9) vạch hình chữ nhật 9 x 39.32 (đầu vát xiên của tấm A).
