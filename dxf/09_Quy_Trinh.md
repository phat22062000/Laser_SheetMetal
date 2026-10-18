# Quy trình làm việc từng bước và nội dung báo cáo

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 83–100). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

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
