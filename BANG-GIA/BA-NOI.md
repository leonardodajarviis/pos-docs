1. Mô tả 

Bảng giá nguyên liệu là tập hợp giá mua vào và bán ra áp dụng tại một thời điểm cho một loại hàng hóa (ví dụ: Vàng ta, Vàng tây, Bạc). 

 

2. Trạng thái bảng giá 

Bảng giá có các trạng thái: 

    Draft (Nháp) 

    Active (Hoạt động) 

    Inactive (Lưu trữ) 

Luồng trạng thái: 

Draft → Active → Inactive 

Ràng buộc: 

    Không được phép chuyển từ Inactive về Active. 

    Một loại bảng giá chỉ được phép có duy nhất một bảng giá Active tại một thời điểm. 

    Khi Activate bảng giá mới cùng loại: 

    Hệ thống tự động chuyển bảng giá Active cũ sang Inactive. 

 

3. Cơ chế tính giá Derived 

Trong trạng thái Draft: 

    Khi nhập hoặc thay đổi giá của Mã Giá gốc: 

    Các Mã Giá kế thừa sẽ tự động tính lại theo công thức: 

Giá mua vào Derived = Giá mua vào Base × Hệ số mua 
Giá bán ra Derived = Giá bán ra Base × Hệ số bán 

    Nếu có nhiều cấp kế thừa: 

    Giá được tính theo chuỗi kế thừa từ gốc đến cấp cuối. 

    Khi sao chép bảng giá: 

    Giá của Mã Giá kế thừa được tính lại theo hệ số hiện tại, không giữ snapshot của bảng giá cũ. 

 

4. Tại thời điểm Activate 

    Hệ thống snapshot toàn bộ giá tại thời điểm Activate. 

    Sau khi Activate: 

    Không được chỉnh sửa. 

    Không phụ thuộc vào thay đổi hệ số trong tương lai. 

 

5. Cho phép thiếu giá 

Hệ thống cho phép lưu bảng giá trong trường hợp một số Mã Giá chưa có giá. 

Tuy nhiên: 

    Khi lưu, hệ thống hiển thị danh sách cụ thể các Mã Giá thiếu giá mua hoặc giá bán. 

    User phải xác nhận tiếp tục lưu. 

    Hệ thống không cho phép Activate nếu không thỏa mãn điều kiện kiểm tra đầy đủ (tùy cấu hình doanh nghiệp – có thể quy định bắt buộc đủ giá trước khi Activate). 