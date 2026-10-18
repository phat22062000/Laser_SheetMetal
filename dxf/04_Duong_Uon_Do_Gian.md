# Đường uốn, độ giãn, cánh nghiêng ≤ 85°

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 29–40). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

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
