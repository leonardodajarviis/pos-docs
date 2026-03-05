# Demo Mermaid Diagrams

## Flowchart Example

<div class="mermaid">
graph TD
    A[Bắt đầu] --> B{Kiểm tra điều kiện}
    B -->|Có| C[Xử lý]
    B -->|Không| D[Bỏ qua]
    C --> E[Kết thúc]
    D --> E
</div>

## Sequence Diagram Example

<div class="mermaid">
sequenceDiagram
    participant User
    participant System
    participant Database
    
    User->>System: Tạo bảng giá
    System->>Database: Lưu thông tin
    Database-->>System: Xác nhận
    System-->>User: Thành công
</div>

## Class Diagram Example

<div class="mermaid">
classDiagram
    class BangGia {
        +String maBangGia
        +String tenBangGia
        +Date ngayHieuLuc
        +taoMoi()
        +capNhat()
    }
    
    class ChiTietBangGia {
        +String maSanPham
        +Decimal giaBan
        +capNhatGia()
    }
    
    BangGia "1" --> "*" ChiTietBangGia
</div>

## State Diagram Example

<div class="mermaid">
stateDiagram-v2
    [*] --> MoiTao
    MoiTao --> DangHieuLuc: Kích hoạt
    DangHieuLuc --> TamNgung: Tạm ngừng
    TamNgung --> DangHieuLuc: Kích hoạt lại
    TamNgung --> [*]: Xóa
    DangHieuLuc --> [*]: Xóa
</div>

---

## Cách sử dụng

Để thêm sơ đồ Mermaid vào file markdown, sử dụng cú pháp:

```html
<div class="mermaid">
graph TD
    A[Start] --> B[End]
</div>
```
