1. Mô tả 

Mã giá là thực thể trung gian dùng để xác định cách tính giá mua vào và bán ra cho các nhóm hàng hóa. 
Mã giá được liên kết với nhóm hàng nhỏ nhất trong cấu trúc nhóm hàng hóa cha – con nhằm đảm bảo việc tra cứu giá có thể thực hiện bằng khóa xác định duy nhất. 

 

2. Cấu trúc Mã Giá 

Mỗi Mã Giá bao gồm các thông tin chính: 

    Mã giá: chọn từ danh sách nhóm hàng hóa, hiển thị all leaf, nhưng disable những leaf đã được gán với mã giá 

    Mã giá gốc (có thể null) 

    Nhóm hàng nhỏ nhất: c 

    Hàm lượng vàng (nếu áp dụng)  

    Thương hiệu (nếu áp dụng) 

    Hệ số mua vào 

    Hệ số bán ra 

    Trạng thái (Active / Inactive) 

 

3. Quan hệ kế thừa giá (Price Code Hierarchy) 

Hệ thống cho phép: 

    Mã giá gốc (Base Code) 

    Mã giá kế thừa (Derived Code) 

    Kế thừa nhiều cấp (A → B → C) 

Ràng buộc: 

    Không được phép tạo vòng lặp kế thừa (ví dụ: A → B → A). 

    Hệ thống phải kiểm tra và ngăn chặn circular reference khi lưu dữ liệu. 

 

4. Nguyên tắc hệ số 

    Hệ số mua vào và bán ra được cấu hình tại Mã Giá. 

    Khi thay đổi hệ số: 

    Không ảnh hưởng đến các bảng giá đã Active. 

    Chỉ áp dụng cho các bảng giá được tạo sau thời điểm thay đổi. 

 

5. Trạng thái Inactive 

    Mã giá Inactive: 

    Không hiển thị khi tạo bảng giá mới. 

    Không hiển thị khi sử dụng chức năng sao chép bảng giá. 

    Vẫn được lưu trữ để tra cứu lịch sử. 

6. Ràng buộc toàn vẹn giữa Nhóm hàng và Mã Giá (Category–PriceCode Integrity) 

Mã giá chỉ được gắn với nhóm hàng nhỏ nhất (không có nhóm con). 

Hệ thống áp dụng các ràng buộc sau: 

    Nếu một nhóm hàng đã được gắn Mã Giá: 

    Không được phép thêm nhóm con vào nhóm hàng đó. 

    Trường hợp user cố thực hiện thay đổi cấu trúc: 

    Hệ thống hiển thị cảnh báo: 

“Category này đang được gắn với Mã Giá XXX. 
Vui lòng xóa hoặc Inactive Mã Giá trước khi thay đổi cấu trúc.” 

    Hệ thống không cho phép lưu thay đổi cấu trúc nếu Mã Giá chưa được xử lý. 