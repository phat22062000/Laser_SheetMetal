# File tổng hợp tất cả hình <số đơn>_ALL.dxf

> Trích từ `skills/dxf-sheetmetal/SKILL.md` (dòng 63–69). Nguồn chuẩn là file SKILL.md; sửa ở đó trước rồi cập nhật lại file này.

- Ngoài các file riêng, tạo thêm 1 file DXF chứa tất cả hình, đặt tên `<số đơn>_ALL.dxf` (ví dụ 521008-11_ALL.dxf), lưu cùng thư mục. Khi người dùng yêu cầu chỉ xuất file ALL thì không tạo file riêng từng mã.
- Xếp thành 1 cột dọc, căn trái tại X = 0, các hình cách nhau 200 mm theo trục Y. Thứ tự mã tăng dần từ trên xuống dưới (mã nhỏ nhất ở trên cùng); hình ở dưới cùng đặt sát Y = 0.
- Giữ nguyên chiều quay, hình dạng và màu như file riêng (gồm cả POINT/TEXT tham khảo và kegaki nếu có), chỉ tịnh tiến.
- Mỗi hình có 1 dòng chữ TEXT ghi mã yêu cầu (6 số), cao 12 mm, đặt bên trái, chính giữa chiều cao hình: điểm chèn (đầu chữ, căn trái) tại X = -50, Y = Y đáy hình + (chiều cao hình)/2 (đáy chữ ngang tâm hình), style Standard, layer 0. Cập nhật EXTMAX theo toàn bộ hình + chữ.
- Chữ cùng layer 0 với đường cắt là chủ ý, người dùng đã biết; không cần nhắc lại.
