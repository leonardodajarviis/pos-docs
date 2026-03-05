# Use Case UC-MAGIA-06: Vô hiệu hóa Mã giá (Deactive)

---

| **Use Case ID** | **UC-MAGIA-06** |
|-----------------|-----------------||
| **Use Case Name** | Vô hiệu hóa Mã giá (Deactive) |
| **Description** | Use Case "Vô hiệu hóa Mã giá" cho phép Admin vô hiệu hóa mã giá đang Active để ngừng sử dụng trong hệ thống. |
| **Actor(s)** | Admin |
| **Priority** | Must Have |
| **Trigger** | Admin yêu cầu vô hiệu hóa một Mã giá đang Active |

---

## Input

| Tên trường | Loại | Bắt buộc | Mô tả | Ràng buộc |
|------------|------|----------|-------|-----------|
| `priceCodeId` | Số | Có | ID mã giá cần vô hiệu hóa | Mã giá phải tồn tại và đang Active |

---

## Output

### Trường hợp thành công:

| Tên trường | Loại | Mô tả |
|------------|------|-------|
| `id` | Số | ID mã giá đã vô hiệu hóa |
| `status` | Văn bản | Trạng thái mới = "Inactive" |
| `deactivatedAt` | Ngày giờ | Thời gian vô hiệu hóa |
| `deactivatedBy` | Văn bản | Người vô hiệu hóa |
| `message` | Văn bản | "Vô hiệu hóa mã giá thành công" |

### Trường hợp lỗi:

| Mã lỗi | Thông báo | Mô tả |
|--------|-----------|-------|
| `PRICE_CODE_NOT_FOUND` | "Mã giá không tồn tại" | Không tìm thấy mã giá |
| `ALREADY_INACTIVE` | "Mã giá đã ở trạng thái Inactive" | Mã giá đang Inactive |
| `HAS_ACTIVE_CHILDREN` | "Không thể vô hiệu hóa. Có mã giá con đang Active" | Có mã giá kế thừa từ mã giá này đang Active |

---

## Pre-Condition(s)

- Mã giá đã tồn tại trong hệ thống
- Mã giá đang có trạng thái Active
- Admin đã đăng nhập và có quyền vô hiệu hóa mã giá
- Không có mã giá con (child) nào đang Active

---

## Post-Condition(s)

- Mã giá chuyển sang trạng thái Inactive
- Mã giá không thể sử dụng cho bảng giá mới
- Hệ thống ghi nhận thông tin người vô hiệu hóa và thời gian vô hiệu hóa
- Audit log ghi nhận hành động DEACTIVATE
- Các bảng giá đã tồn tại không bị ảnh hưởng

---

## Basic Flow

1. Admin yêu cầu vô hiệu hóa một mã giá đang Active
2. Hệ thống kiểm tra tính hợp lệ:
   - Mã giá tồn tại
   - Mã giá đang Active
   - Không có mã giá con (children) nào đang Active
3. Hệ thống cập nhật:
   - Chuyển status từ Active → Inactive
   - Ghi nhận thời gian vô hiệu hóa (deactivatedAt)
   - Ghi nhận người vô hiệu hóa (deactivatedBy)
4. Hệ thống ghi audit log với action = DEACTIVATE
5. Hệ thống trả về kết quả thành công

Use case kết thúc.

---

## Alternative Flow

*Không có luồng thay thế*

---

## Exception Flow

### 2a. Mã giá không tồn tại

2a. Hệ thống không tìm thấy mã giá với ID được cung cấp

2a1. Hệ thống trả về lỗi: "Mã giá không tồn tại hoặc đã bị xóa."

2a2. Use case kết thúc

### 2b. Mã giá đã ở trạng thái Inactive

2b. Hệ thống phát hiện mã giá đang ở trạng thái Inactive

2b1. Hệ thống trả về lỗi: "Mã giá đã ở trạng thái Inactive. Không cần vô hiệu hóa lại."

2b2. Use case kết thúc

### 2c. Có mã giá con đang Active

2c. Hệ thống phát hiện có mã giá khác đang Active kế thừa từ mã giá này

2c1. Hệ thống lấy danh sách các mã giá con đang Active

2c2. Hệ thống trả về lỗi: "Không thể vô hiệu hóa mã giá này. Có [N] mã giá con đang Active kế thừa từ mã giá này: [Danh sách]. Vui lòng vô hiệu hóa các mã giá con trước."

2c3. Use case kết thúc

---

## Business Rules

### BR-MAGIA-033: Chỉ Admin được vô hiệu hóa

- Chỉ Admin mới có quyền vô hiệu hóa mã giá
- Nhân viên không có quyền này
- Lý do: Tránh thay đổi không kiểm soát ảnh hưởng đến toàn hệ thống

### BR-MAGIA-034: Chỉ vô hiệu hóa mã giá Active

- Chỉ có thể vô hiệu hóa mã giá đang ở trạng thái **Active**
- Nếu mã giá đã Inactive → Từ chối thao tác
- Mục đích: Tránh thao tác không cần thiết

### BR-MAGIA-035: Kiểm tra mã giá con (Children)

Trước khi vô hiệu hóa mã giá:
- **Phải kiểm tra không có mã giá con nào đang Active**
- Nếu có mã giá con Active → Từ chối vô hiệu hóa
- Lý do: Đảm bảo tính toàn vẹn của chuỗi kế thừa

**Quy trình vô hiệu hóa đúng đắn:**
```
PC-001 (gốc): Active
  ├─ PC-002 (con): Active
      └─ PC-003 (cháu): Active

Muốn Deactive PC-001:
1. Deactive PC-003 trước (cháu)
2. Deactive PC-002 sau (con)
3. Deactive PC-001 cuối cùng (gốc)

→ Vô hiệu hóa từ lá (leaf) lên gốc (root)
```

### BR-MAGIA-036: Không ảnh hưởng bảng giá đã tồn tại

Việc vô hiệu hóa mã giá **không ảnh hưởng** đến các bảng giá đã tồn tại:
- Các bảng giá đã Active trước khi mã giá bị Deactive → Vẫn hoạt động bình thường
- Chỉ các bảng giá tạo mới sau khi Deactive → Không thể sử dụng mã giá này

**Ví dụ:**
```
Thời điểm T1:
- Mã giá PC-001: Active
- Bảng giá BG-001 (sử dụng PC-001): Active

Thời điểm T2: Admin Deactive PC-001
- Mã giá PC-001: Inactive
- Bảng giá BG-001: Vẫn Active và hoạt động bình thường

Thời điểm T3: Tạo bảng giá mới
- Không thể chọn PC-001 (đã Inactive)
- Phải chọn mã giá Active khác
```

### BR-MAGIA-037: Ghi nhận audit log

Mỗi lần vô hiệu hóa mã giá, hệ thống ghi nhận đầy đủ:
- Action: DEACTIVATE
- Thời gian vô hiệu hóa (deactivatedAt)
- Người vô hiệu hóa (deactivatedBy)
- Trạng thái trước: Active
- Trạng thái sau: Inactive
- Lý do vô hiệu hóa (optional)

Mục đích: Theo dõi lịch sử thay đổi trạng thái

### BR-MAGIA-038: Có thể kích hoạt lại

- Mã giá đã bị Deactive có thể được kích hoạt lại thông qua UC05
- Dữ liệu mã giá được giữ nguyên, chỉ thay đổi status
- Không xóa mã giá khỏi hệ thống

---

## Diagrams

### 1. Use Case Diagram - UC06: Vô hiệu hóa Mã giá

```mermaid
graph LR
    Actor["👤 Admin"]
    
    UC06["UC06: Vô hiệu hóa Mã giá<br/>(Deactive)"]
    
    Actor -->|Thực hiện| UC06
    
    style UC06 fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style Actor fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

### 2. Activity Diagram - Luồng vô hiệu hóa Mã giá

```mermaid
flowchart TD
    Start([Bắt đầu]) --> Input[Admin yêu cầu vô hiệu hóa<br/>priceCodeId]
    
    Input --> CheckExists{Mã giá<br/>tồn tại?}
    
    CheckExists -->|Không| Error1[Lỗi: Mã giá không tồn tại]
    Error1 --> End
    
    CheckExists -->|Có| CheckStatus{Trạng thái<br/>Active?}
    
    CheckStatus -->|Không| Error2[Lỗi: Đã Inactive]
    Error2 --> End
    
    CheckStatus -->|Có| CheckChildren{Có mã giá con<br/>đang Active?}
    
    CheckChildren -->|Có| Error3[Lỗi: Có mã giá con Active<br/>Phải Deactive con trước]
    Error3 --> End
    
    CheckChildren -->|Không| Deactivate[Cập nhật status = Inactive<br/>Ghi nhận thời gian & người Deactive]
    
    Deactivate --> Log[Ghi audit log<br/>Action: DEACTIVATE]
    
    Log --> Success[Trả về kết quả thành công]
    
    Success --> End([Kết thúc])
    
    style Start fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style End fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style Success fill:#a5d6a7,stroke:#388e3c,stroke-width:2px
    style Error1 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error2 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error3 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
```

### 3. Sequence Diagram - Vô hiệu hóa Mã giá

```mermaid
sequenceDiagram
    actor Admin as 👤 Admin
    participant Controller as Controller
    participant Service as PriceCodeService
    participant DB as Database
    
    Admin->>Controller: 1. Yêu cầu vô hiệu hóa (priceCodeId)
    Controller->>Service: 2. Deactivate price code
    
    Service->>DB: 3. Query mã giá by ID
    DB-->>Service: Thông tin mã giá
    
    alt Mã giá không tồn tại
        Service-->>Controller: Lỗi: Mã giá không tồn tại
        Controller-->>Admin: "Mã giá không tồn tại"
    else Mã giá tồn tại
        Service->>Service: 4. Kiểm tra trạng thái
        
        alt Mã giá đã Inactive
            Service-->>Controller: Lỗi: Đã Inactive
            Controller-->>Admin: "Mã giá đã ở trạng thái Inactive"
        else Mã giá đang Active
            Service->>DB: 5. Kiểm tra mã giá con đang Active
            DB-->>Service: Danh sách mã giá con
            
            alt Có mã giá con Active
                Service-->>Controller: Lỗi: Có mã giá con Active
                Controller-->>Admin: "Có mã giá con đang Active"<br/>Danh sách mã giá con
            else Không có mã giá con Active
                Service->>DB: 6. Cập nhật status = Inactive
                DB-->>Service: Thành công
                
                Service->>DB: 7. Ghi audit log (DEACTIVATE)
                DB-->>Service: OK
                
                Service-->>Controller: Vô hiệu hóa thành công
                Controller-->>Admin: ✅ "Vô hiệu hóa mã giá thành công"
            end
        end
    end
```

**Giải thích Sequence Diagram:**

**Kiểm tra tồn tại (Bước 1-3):**
- Admin yêu cầu vô hiệu hóa với priceCodeId
- Hệ thống query mã giá từ database
- Nếu không tồn tại → Trả về lỗi

**Kiểm tra trạng thái (Bước 4):**
- Kiểm tra mã giá đang Active
- Nếu đã Inactive → Trả về lỗi

**Kiểm tra mã giá con (Bước 5):**
- Query các mã giá kế thừa từ mã giá này
- Kiểm tra có mã giá con nào đang Active không
- Nếu có → Trả về lỗi với danh sách mã giá con

**Vô hiệu hóa (Bước 6-7):**
- Cập nhật status = Inactive
- Ghi audit log
- Trả về thành công

---

### 4. Class Diagram

```mermaid
classDiagram
    class PriceCodeController {
        +deactivatePriceCode() Kết quả
    }
    
    class PriceCodeService {
        +deactivatePriceCode() Kết quả
        +checkCanDeactivate() Boolean
        +validateNoActiveChildren() Boolean
        +getActiveChildren() Danh sách
    }
    
    class PriceCode {
        +id: ID
        +categoryId: ID nhóm hàng
        +parentPriceCodeId: ID mã giá gốc
        +status: Trạng thái
        +deactivatedAt: Thời gian vô hiệu hóa
        +deactivatedBy: Người vô hiệu hóa
        +isActive() Boolean
        +isInactive() Boolean
        +deactivate() Vô hiệu hóa
        +hasActiveChildren() Boolean
    }
    
    class AuditLog {
        +id: ID
        +entityType: Loại thực thể
        +entityId: ID thực thể
        +action: Hành động
        +oldValue: Giá trị cũ
        +newValue: Giá trị mới
        +changedBy: Người thay đổi
        +changedAt: Thời gian thay đổi
    }
    
    class PriceCodeRepository {
        +findById() Mã giá
        +updateStatus() Cập nhật trạng thái
        +findActiveChildrenByParentId() Danh sách con Active
        +saveAuditLog() Lưu audit log
    }
    
    PriceCodeController --> PriceCodeService : sử dụng
    PriceCodeService --> PriceCodeRepository : sử dụng
    PriceCodeRepository --> PriceCode : quản lý
    PriceCodeRepository --> AuditLog : ghi log
    PriceCode "1" --> "0..*" AuditLog : có lịch sử
    PriceCode "0..*" --> "0..1" PriceCode : kế thừa từ
    PriceCode "1" --> "0..*" PriceCode : có mã giá con
    
    note for PriceCode "Kiểm tra trước khi Deactive:\n- Đang Active\n- Không có mã con Active"
    note for AuditLog "Ghi action = DEACTIVATE"
```