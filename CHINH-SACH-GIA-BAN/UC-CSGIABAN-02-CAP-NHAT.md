# Use Case UC-CSGIABAN-02: Cập Nhật Chính Sách Giá Bán

---

| **Use Case ID** | **UC-CSGIABAN-02** |
|-----------------|---------------------|
| **Use Case Name** | Cập Nhật Chính Sách Giá Bán |
| **Description** | Use Case "Cập Nhật Chính Sách Giá Bán" cho phép Admin chỉnh sửa thông tin chính sách giá đã tồn tại. Việc cập nhật chính sách đã có hiệu lực sẽ được ghi log đầy đủ và yêu cầu xác nhận từ Admin do có thể ảnh hưởng đến giá bán hiện tại. |
| **Actor(s)** | Admin |
| **Priority** | Must Have |
| **Trigger** | Admin chọn chức năng "Cập nhật chính sách giá" từ danh sách hoặc chi tiết chính sách |

---

## Input

| Tên trường | Loại | Bắt buộc | Mô tả | Ràng buộc |
|------------|------|----------|-------|-----------|
| `policyId` | Số | Có | ID chính sách cần cập nhật | Phải tồn tại trong hệ thống |
| `effectiveDate` | Ngày giờ | Có | Ngày có hiệu lực mới | Không được nhỏ hơn ngày hiện tại (trừ trường hợp backdate với quyền đặc biệt) |
| `businessLineId` | Số | Có | Mảng kinh doanh | Phải là mảng kinh doanh hợp lệ |
| `scopeType` | Enum | Có | Loại phạm vi áp dụng | `ALL_SYSTEM` hoặc `SPECIFIC_STORE` hoặc `SPECIFIC_REGION` |
| `scopeId` | Số | Có (nếu scope cụ thể) | ID cửa hàng/khu vực | Bắt buộc khi scopeType không phải ALL_SYSTEM |
| `productLineId` | Số | Có | Dòng sản phẩm | Phải thuộc mảng kinh doanh đã chọn |
| `priceCodeId` | Số | Có | QTTG bán (Price Code) | Phải là mã giá Active, phù hợp với dòng sản phẩm |
| `description` | Văn bản | Không | Ghi chú/Mô tả | Max 500 ký tự |
| `updateReason` | Văn bản | Có (nếu đã có hiệu lực) | Lý do cập nhật | Bắt buộc khi cập nhật chính sách đã Active, max 200 ký tự |

**Quy tắc đầu vào:**
- Chính sách phải tồn tại và ở trạng thái Active
- Khi cập nhật chính sách đã có hiệu lực (effectiveDate <= ngày hiện tại), phải cung cấp lý do cập nhật
- Nếu thay đổi dòng sản phẩm hoặc phạm vi, phải kiểm tra xung đột tương tự như tạo mới
- Không cho phép cập nhật chính sách đã bị ngưng hiệu lực (Inactive)

---

## Output

### Trường hợp thành công:

| Tên trường | Loại | Mô tả |
|------------|------|-------|
| `id` | Số | ID chính sách đã cập nhật |
| `policyCode` | Văn bản | Mã quy tắc (không thay đổi) |
| `effectiveDate` | Ngày giờ | Ngày có hiệu lực mới |
| `businessLine` | Thông tin | Thông tin mảng kinh doanh (id, name) |
| `scopeType` | Văn bản | Loại phạm vi áp dụng |
| `scopeName` | Văn bản | Tên phạm vi (Toàn hệ thống / Tên cửa hàng / Tên khu vực) |
| `productLine` | Thông tin | Thông tin dòng sản phẩm (id, code, name) |
| `priceCode` | Thông tin | Thông tin QTTG bán (id, code, name) |
| `pricingFormula` | Văn bản | Công thức tính giá (hiển thị từ Price Code) |
| `status` | Văn bản | Trạng thái (Active) |
| `description` | Văn bản | Ghi chú/Mô tả |
| `updatedAt` | Ngày giờ | Thời gian cập nhật |
| `updatedBy` | Văn bản | Người cập nhật |
| `updateReason` | Văn bản | Lý do cập nhật (nếu có) |

### Trường hợp lỗi:

| Mã lỗi | Thông báo | Mô tả |
|--------|-----------|-------|
| `POLICY_NOT_FOUND` | "Chính sách không tồn tại" | Không tìm thấy chính sách với ID đã cho |
| `POLICY_INACTIVE` | "Không thể cập nhật chính sách đã ngưng hiệu lực" | Chính sách có status = Inactive |
| `INVALID_EFFECTIVE_DATE` | "Ngày có hiệu lực không được nhỏ hơn ngày hiện tại" | Ngày hiệu lực trong quá khứ (không có quyền backdate) |
| `PRODUCT_LINE_MISMATCH` | "Dòng sản phẩm không thuộc mảng kinh doanh đã chọn" | Dòng sản phẩm không khớp với mảng kinh doanh |
| `POLICY_CONFLICT` | "Dòng sản phẩm đã có chính sách giá khác đang áp dụng" | Vi phạm BR-CSGIABAN-03 khi thay đổi dòng sản phẩm/phạm vi |
| `INACTIVE_PRICE_CODE` | "Mã giá đã bị vô hiệu hóa" | Price Code không ở trạng thái Active |
| `UPDATE_REASON_REQUIRED` | "Vui lòng nhập lý do cập nhật" | Thiếu lý do khi cập nhật chính sách đã có hiệu lực |
| `INVALID_SCOPE` | "Cửa hàng/Khu vực không tồn tại" | scopeId không hợp lệ |

---

## Pre-Condition(s)

- Chính sách giá đã tồn tại trong hệ thống
- Chính sách đang ở trạng thái Active
- Mảng kinh doanh và dòng sản phẩm đã được cấu hình
- Mã giá (Price Code) mới (nếu thay đổi) đang ở trạng thái Active
- Admin đã đăng nhập và có quyền cập nhật chính sách giá

---

## Post-Condition(s)

- Chính sách giá được cập nhật thành công với thông tin mới
- Hệ thống ghi nhận thông tin người cập nhật và thời gian cập nhật
- Audit log ghi nhận đầy đủ thông tin thay đổi (old values → new values)
- (Nếu thay đổi Price Code) Công thức tính giá mới được snapshot
- (Nếu chính sách đã có hiệu lực) Lý do cập nhật được lưu trữ trong audit log

---

## Basic Flow

1. Admin yêu cầu cập nhật chính sách giá (từ danh sách hoặc chi tiết)
2. Hệ thống kiểm tra chính sách tồn tại và đang Active
3. Hệ thống hiển thị form cập nhật với thông tin hiện tại:
   - Mã quy tắc (chỉ xem, không sửa được)
   - Ngày có hiệu lực
   - Mảng kinh doanh
   - Phạm vi áp dụng
   - Dòng sản phẩm (lọc theo mảng kinh doanh)
   - QTTG bán - Price Code
   - Công thức tính giá (hiển thị từ Price Code)
   - Ghi chú/Mô tả
4. Admin chỉnh sửa các thông tin cần thiết
5. Hệ thống load và hiển thị công thức tính giá tương ứng (nếu thay đổi Price Code)
6. Hệ thống kiểm tra xem chính sách đã có hiệu lực chưa (effectiveDate <= ngày hiện tại)
7. Nếu chính sách đã có hiệu lực:
   - Hệ thống yêu cầu Admin nhập lý do cập nhật (bắt buộc)
   - Hiển thị cảnh báo về tác động của việc thay đổi
8. Admin xác nhận cập nhật và cung cấp lý do (nếu cần)
9. Hệ thống kiểm tra tính hợp lệ của dữ liệu:
   - Ngày có hiệu lực >= ngày hiện tại (hoặc có quyền backdate)
   - Dòng sản phẩm thuộc mảng kinh doanh đã chọn
   - Price Code đang ở trạng thái Active
   - (Nếu thay đổi dòng sản phẩm/phạm vi) Không có chính sách Active khác xung đột
10. Hệ thống lưu lại giá trị cũ cho audit log
11. Hệ thống cập nhật chính sách với thông tin mới
12. (Nếu thay đổi Price Code) Hệ thống snapshot công thức tính giá mới
13. Hệ thống ghi audit log với đầy đủ thông tin:
    - Các trường đã thay đổi (old value → new value)
    - Người cập nhật và thời gian
    - Lý do cập nhật (nếu có)
14. Hệ thống trả về kết quả thành công với thông tin đã cập nhật

Use case kết thúc.

---

## Alternative Flow

### A1. Admin hủy bỏ cập nhật

4a. Admin không muốn tiếp tục cập nhật

4a1. Admin chọn "Hủy bỏ"

4a2. Hệ thống hỏi xác nhận: "Các thay đổi chưa được lưu sẽ bị mất. Bạn có chắc chắn muốn hủy?"

4a3. Nếu Admin xác nhận → Quay lại màn hình trước đó, không lưu thay đổi

Use case kết thúc.

### A2. Không có thay đổi nào

4b. Admin không thay đổi bất kỳ thông tin nào và nhấn lưu

4b1. Hệ thống phát hiện không có thay đổi

4b2. Hệ thống thông báo: "Không có thay đổi nào để lưu"

4b3. Use case kết thúc

---

## Exception Flow

**Lưu ý:** Các exception flows được mô tả chi tiết trong **Sequence Diagram** (các nhánh `alt` cho error cases)

### 2a. Chính sách không tồn tại

2a. Hệ thống không tìm thấy chính sách với ID đã cho

2a1. Hệ thống trả về lỗi: "Chính sách không tồn tại hoặc đã bị xóa"

2a2. Use case kết thúc

### 2b. Chính sách đã bị ngưng hiệu lực

2b. Hệ thống phát hiện chính sách có status = Inactive

2b1. Hệ thống trả về lỗi: "Không thể cập nhật chính sách đã ngưng hiệu lực. Chính sách này đã bị vô hiệu hóa vào [Ngày]."

2b2. Use case kết thúc

### 8a. Thiếu lý do cập nhật

8a. Admin không cung cấp lý do cập nhật khi chính sách đã có hiệu lực

8a1. Hệ thống trả về lỗi: "Vui lòng nhập lý do cập nhật. Đây là chính sách đã có hiệu lực và đang ảnh hưởng đến giá bán hiện tại."

8a2. Use case quay lại bước 7

### 9a. Ngày hiệu lực không hợp lệ

9a. Hệ thống phát hiện ngày có hiệu lực < ngày hiện tại và Admin không có quyền backdate

9a1. Hệ thống trả về lỗi: "Ngày có hiệu lực không được nhỏ hơn ngày hiện tại. Ngày hiện tại: [DD/MM/YYYY]"

9a2. Use case quay lại bước 4

### 9b. Dòng sản phẩm không thuộc mảng kinh doanh

9b. Hệ thống phát hiện dòng sản phẩm được chọn không thuộc mảng kinh doanh đã chọn

9b1. Hệ thống trả về lỗi: "Dòng sản phẩm '[Tên dòng SP]' không thuộc mảng kinh doanh '[Tên mảng KD]'. Vui lòng chọn lại."

9b2. Use case quay lại bước 4

### 9c. Xung đột chính sách giá (khi thay đổi dòng sản phẩm/phạm vi)

9c. Hệ thống phát hiện đã có chính sách giá Active khác cho dòng sản phẩm mới tại phạm vi mới

9c1. Hệ thống kiểm tra:
- Cùng dòng sản phẩm (mới)
- Cùng phạm vi (mới, hoặc có chính sách Toàn hệ thống)
- Trạng thái = Active
- ID khác với chính sách đang cập nhật
- EffectiveDate <= Ngày hiệu lực của chính sách đang cập nhật

9c2. Hệ thống trả về lỗi: 
> "Dòng sản phẩm '[Tên dòng SP]' đã được áp dụng chính sách giá '[Mã chính sách khác]' tại [Phạm vi] từ ngày [Ngày hiệu lực].  
> Vui lòng ngưng hiệu lực chính sách đó hoặc chọn dòng sản phẩm/phạm vi khác."

9c3. Use case quay lại bước 4

### 9d. Price Code không Active

9d. Hệ thống phát hiện Price Code được chọn đã bị vô hiệu hóa (status = Inactive)

9d1. Hệ thống trả về lỗi: "Mã giá '[Mã Price Code]' đã bị vô hiệu hóa. Vui lòng chọn mã giá Active khác."

9d2. Use case quay lại bước 4

### 9e. Phạm vi áp dụng không hợp lệ

9e. Hệ thống phát hiện phạm vi áp dụng không hợp lệ:
- Chọn phạm vi cụ thể nhưng không cung cấp scopeId
- scopeId không tồn tại trong hệ thống

9e1. Hệ thống trả về lỗi: 
- Nếu thiếu scopeId: "Vui lòng chọn cửa hàng/khu vực áp dụng"
- Nếu scopeId không tồn tại: "Cửa hàng/Khu vực không tồn tại trong hệ thống"

9e2. Use case quay lại bước 4

---

## Business Rules

### BR-CSGIABAN-01: Thời gian hiệu lực

- Ngày có hiệu lực **không được nhỏ hơn** ngày hiện tại
- **Ngoại lệ**: Admin có quyền đặc biệt có thể backdate (thiết lập ngày trong quá khứ)
- Khi backdate, hệ thống phải hiển thị cảnh báo rõ ràng về tác động

### BR-CSGIABAN-02: Ưu tiên phạm vi

- Khi thay đổi phạm vi áp dụng, phải đảm bảo không vi phạm quy tắc ưu tiên
- Chính sách cụ thể có ưu tiên cao hơn chính sách toàn hệ thống

### BR-CSGIABAN-03: Chính sách duy nhất

- Khi thay đổi dòng sản phẩm hoặc phạm vi, phải kiểm tra xung đột với chính sách khác
- Không được tạo ra tình trạng 2 chính sách Active cùng áp dụng cho 1 dòng sản phẩm tại 1 phạm vi

### BR-CSGIABAN-04: Snapshot Công thức

- Khi thay đổi Price Code, hệ thống phải snapshot lại công thức tính giá mới
- Công thức cũ được lưu trong audit log để tham khảo

### BR-CSGIABAN-06: Audit Log khi cập nhật

**Yêu cầu ghi log đầy đủ:**
- Tất cả các trường đã thay đổi phải được ghi log với format: `[Field]: [Old Value] → [New Value]`
- Thông tin người cập nhật và thời gian cập nhật
- Lý do cập nhật (bắt buộc nếu chính sách đã có hiệu lực)
- IP address và user agent (nếu cập nhật qua web)

**Ví dụ audit log:**
```
Policy ID: PP-2026-001
Updated At: 05/03/2026 14:30:00
Updated By: admin@company.com

Changes:
- effectiveDate: 05/03/2026 → 06/03/2026
- priceCodeId: PC-001 → PC-002
- pricingFormula: "Giá = X × 1.15" → "Giá = X × 1.20"

Update Reason: "Điều chỉnh hệ số theo chính sách mới của công ty"
```

### BR-CSGIABAN-07: Không cập nhật chính sách Inactive

- Chỉ cho phép cập nhật chính sách đang ở trạng thái Active
- Chính sách đã bị ngưng hiệu lực (Inactive) không thể cập nhật
- Mục đích: Đảm bảo tính toàn vẹn của dữ liệu lịch sử

### BR-CSGIABAN-08: Cảnh báo khi cập nhật chính sách đã có hiệu lực

Khi cập nhật chính sách đã có hiệu lực (effectiveDate <= ngày hiện tại):

**Yêu cầu bắt buộc:**
- Admin phải nhập lý do cập nhật (min 10 ký tự, max 200 ký tự)
- Hệ thống phải hiển thị cảnh báo rõ ràng về tác động

**Nội dung cảnh báo:**
> "⚠️ CẢNH BÁO: Chính sách đã có hiệu lực  
> 
> Chính sách này đã có hiệu lực từ [Ngày]. Việc thay đổi sẽ ảnh hưởng đến giá bán hiện tại.
> 
> Các thay đổi bạn đang thực hiện:
> - [Liệt kê các trường đã thay đổi]
> 
> Vui lòng nhập lý do cập nhật và xác nhận với phòng Kế toán trước khi lưu.
> 
> Tất cả thay đổi sẽ được ghi log đầy đủ."

---

## Diagrams

### 1. Use Case Diagram - UC-CSGIABAN-02: Cập Nhật Chính Sách Giá Bán

```mermaid
graph LR
    Actor["👤 Admin"]
    
    UC["UC-CSGIABAN-02:<br/>Cập Nhật Chính Sách Giá Bán"]
    
    Include1["«include»<br/>Kiểm tra chính sách<br/>tồn tại và Active"]
    Include2["«include»<br/>Kiểm tra xung đột<br/>(nếu đổi dòng SP/phạm vi)"]
    Include3["«include»<br/>Snapshot công thức mới<br/>(nếu đổi Price Code)"]
    Include4["«include»<br/>Ghi audit log<br/>đầy đủ"]
    
    Extend1["«extend»<br/>Yêu cầu lý do cập nhật<br/>(nếu đã có hiệu lực)"]
    
    Actor -->|Thực hiện| UC
    UC -.->|include| Include1
    UC -.->|include| Include2
    UC -.->|include| Include3
    UC -.->|include| Include4
    UC -.->|extend| Extend1
    
    style UC fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style Actor fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Include1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
    style Include2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
    style Include3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
    style Include4 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px
    style Extend1 fill:#fff3e0,stroke:#e65100,stroke-width:1px
```

### 2. Activity Diagram - Luồng Cập Nhật Chính Sách Giá Bán

```mermaid
flowchart TD
    Start([Bắt đầu]) --> Load[Load thông tin chính sách hiện tại]
    
    Load --> CheckExist{Chính sách<br/>tồn tại?}
    
    CheckExist -->|Không| ErrorNotFound[Lỗi: Không tìm thấy]
    ErrorNotFound --> End([Kết thúc])
    
    CheckExist -->|Có| CheckStatus{Status<br/>= Active?}
    
    CheckStatus -->|Inactive| ErrorInactive[Lỗi: Đã ngưng hiệu lực]
    ErrorInactive --> End
    
    CheckStatus -->|Active| ShowForm[Hiển thị form với dữ liệu hiện tại]
    
    ShowForm --> Edit[Admin chỉnh sửa thông tin]
    
    Edit --> CheckChange{Có thay đổi<br/>nào?}
    
    CheckChange -->|Không| NoChange[Thông báo: Không có thay đổi]
    NoChange --> End
    
    CheckChange -->|Có| CheckEffective{Đã có<br/>hiệu lực?}
    
    CheckEffective -->|Có| RequireReason[Yêu cầu lý do cập nhật<br/>+ Hiển thị cảnh báo]
    
    RequireReason --> HasReason{Có nhập<br/>lý do?}
    
    HasReason -->|Không| ErrorReason[Lỗi: Thiếu lý do]
    ErrorReason --> Edit
    
    HasReason -->|Có| Validate
    CheckEffective -->|Chưa| Validate{Kiểm tra<br/>tính hợp lệ}
    
    Validate -->|Ngày không hợp lệ| Error1[Lỗi: Ngày hiệu lực]
    Validate -->|Dòng SP sai mảng| Error2[Lỗi: Mismatch]
    Validate -->|Xung đột chính sách| Error3[Lỗi: Conflict]
    Validate -->|Price Code Inactive| Error4[Lỗi: PC không Active]
    
    Error1 --> Edit
    Error2 --> Edit
    Error3 --> Edit
    Error4 --> Edit
    
    Validate -->|Hợp lệ| SaveOld[Lưu giá trị cũ<br/>cho audit log]
    
    SaveOld --> Update[Cập nhật chính sách<br/>với thông tin mới]
    
    Update --> SnapshotCheck{Có đổi<br/>Price Code?}
    
    SnapshotCheck -->|Có| Snapshot[Snapshot công thức mới]
    SnapshotCheck -->|Không| AuditLog
    
    Snapshot --> AuditLog[Ghi audit log đầy đủ:<br/>- Old → New values<br/>- Người cập nhật<br/>- Lý do cập nhật]
    
    AuditLog --> Success[✅ Trả về kết quả thành công]
    
    Success --> End
    
    style Start fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style End fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style Success fill:#a5d6a7,stroke:#388e3c,stroke-width:2px
    style ErrorNotFound fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ErrorInactive fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ErrorReason fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error1 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error2 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error3 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error4 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style RequireReason fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style AuditLog fill:#bbdefb,stroke:#1976d2,stroke-width:2px
```

### 3. Sequence Diagram - Cập Nhật Chính Sách Giá Bán

```mermaid
sequenceDiagram
    actor Admin as 👤 Admin
    participant UI as UI Layer
    participant Service as PricingPolicyService
    participant Validator as PolicyValidator
    participant AuditSvc as AuditLogService
    participant DB as Database
    
    Admin->>UI: 1. Yêu cầu cập nhật chính sách
    UI->>Service: 2. getPolicyById(policyId)
    Service->>DB: Query chính sách
    
    alt Chính sách không tồn tại
        DB-->>Service: Not found
        Service-->>UI: Error: POLICY_NOT_FOUND
        UI-->>Admin: "Chính sách không tồn tại"
    else Chính sách Inactive
        DB-->>Service: Policy (status = Inactive)
        Service-->>UI: Error: POLICY_INACTIVE
        UI-->>Admin: "Không thể cập nhật chính sách<br/>đã ngưng hiệu lực"
    else Chính sách Active
        DB-->>Service: Policy data
        Service-->>UI: Policy details
        UI->>Admin: 3. Hiển thị form với dữ liệu hiện tại
        
        Admin->>UI: 4. Chỉnh sửa thông tin
        
        UI->>Service: 5. checkIfEffective(effectiveDate)
        Service-->>UI: isEffective = true/false
        
        alt Đã có hiệu lực
            UI->>Admin: 6. Hiển thị cảnh báo + Yêu cầu lý do
            Admin->>UI: 7. Nhập lý do và xác nhận
            
            alt Không nhập lý do
                UI-->>Admin: ❌ "Vui lòng nhập lý do cập nhật"
            else Có lý do
                UI->>Service: 8. updatePolicy(policyId, newData, reason)
            end
        else Chưa có hiệu lực
            Admin->>UI: 6. Xác nhận cập nhật
            UI->>Service: 7. updatePolicy(policyId, newData)
        end
        
        rect rgb(240, 248, 255)
            Note over Service,DB: Kiểm tra và xác thực
            
            Service->>Validator: 8. validate(newData)
            
            Validator->>DB: Kiểm tra validation rules
            DB-->>Validator: Results
            
            alt Có lỗi validation
                Validator-->>Service: ❌ ValidationError
                Service-->>UI: Error response
                UI-->>Admin: "Thông báo lỗi chi tiết"
            else Hợp lệ
                Validator-->>Service: ✅ Valid
                
                Service->>DB: 9. Load old values
                DB-->>Service: Old policy data
                
                Service->>DB: 10. Update policy
                DB-->>Service: Success
                
                alt Có thay đổi Price Code
                    Service->>Service: 11. Snapshot công thức mới
                end
                
                Service->>AuditSvc: 12. logUpdate(oldData, newData,<br/>updatedBy, reason)
                AuditSvc->>DB: Save audit log
                DB-->>AuditSvc: Success
                AuditSvc-->>Service: Log saved
                
                Service-->>UI: ✅ Success response
                UI-->>Admin: ✅ "Cập nhật chính sách thành công"
            end
        end
    end
```

**Giải thích Sequence Diagram:**

Diagram tập trung vào **business logic** và **luồng xử lý nghiệp vụ**:

**Xử lý nghiệp vụ:**
- Kiểm tra chính sách tồn tại và ở trạng thái Active
- Phân biệt xử lý giữa chính sách đã/chưa có hiệu lực
- Yêu cầu lý do cập nhật đối với chính sách đã có hiệu lực
- Lưu trữ giá trị cũ trước khi cập nhật để ghi audit log

**Nhánh xử lý:**
- **Chính sách không tồn tại/Inactive**: Từ chối cập nhật
- **Chính sách đã có hiệu lực**: Yêu cầu lý do + cảnh báo
- **Validation thất bại**: Trả về lỗi cụ thể
- **Thành công**: Cập nhật, snapshot (nếu cần), ghi audit log

**Xử lý lỗi:**
- Chính sách không tồn tại
- Chính sách đã Inactive
- Thiếu lý do cập nhật (khi đã có hiệu lực)
- Các lỗi validation tương tự UC-CSGIABAN-01

---

### 4. Class Diagram

```mermaid
classDiagram
    class PricingPolicyController {
        +updatePricingPolicy() PricingPolicyResponse
        +getPolicyById() PricingPolicyResponse
    }
    
    class PricingPolicyService {
        +updatePolicy() PricingPolicy
        +getPolicyById() PricingPolicy
        +checkIfEffective() Boolean
        +validateUpdate() ValidationResult
        +checkConflictOnUpdate() Boolean
        +snapshotFormula() String
    }
    
    class PricingPolicy {
        +id: ID
        +policyCode: String
        +effectiveDate: DateTime
        +businessLineId: ID
        +scopeType: ScopeTypeEnum
        +scopeId: ID
        +productLineId: ID
        +priceCodeId: ID
        +pricingFormula: String
        +status: StatusEnum
        +description: String
        +createdAt: DateTime
        +createdBy: String
        +updatedAt: DateTime
        +updatedBy: String
    }
    
    class PolicyValidator {
        +validateUpdate() Boolean
        +checkPolicyExists() Boolean
        +checkPolicyActive() Boolean
        +validateEffectiveDate() Boolean
        +checkConflictOnUpdate() ConflictResult
    }
    
    class AuditLogService {
        +logUpdate() void
        +saveChangeHistory() void
        +compareValues() ChangeSet
    }
    
    class PolicyAuditLog {
        +id: ID
        +policyId: ID
        +action: ActionEnum
        +oldValues: JSON
        +newValues: JSON
        +changedFields: Array
        +updateReason: String
        +updatedBy: String
        +updatedAt: DateTime
        +ipAddress: String
    }
    
    class PricingPolicyRepository {
        +findById() PricingPolicy
        +update() PricingPolicy
        +checkConflict() Boolean
    }
    
    PricingPolicyController --> PricingPolicyService : sử dụng
    PricingPolicyService --> PolicyValidator : xác thực
    PricingPolicyService --> AuditLogService : ghi log
    PricingPolicyService --> PricingPolicyRepository : sử dụng
    PricingPolicyRepository --> PricingPolicy : quản lý
    AuditLogService --> PolicyAuditLog : tạo log
    
    note for PricingPolicy "Thực thể chính lưu trữ<br/>chính sách giá bán.<br/>Có updatedAt và updatedBy"
    note for PolicyAuditLog "Lưu trữ lịch sử thay đổi<br/>với old/new values<br/>và lý do cập nhật"
    note for AuditLogService "Ghi log đầy đủ theo<br/>BR-CSGIABAN-06"
```

---

## Notes

**So sánh với UC-CSGIABAN-01 (Tạo mới):**

| Khía cạnh | UC-01: Tạo mới | UC-02: Cập nhật |
|-----------|----------------|-----------------|
| **Mã quy tắc** | Sinh tự động | Không thay đổi (read-only) |
| **Validation** | Kiểm tra xung đột cơ bản | Kiểm tra xung đột (nếu thay đổi dòng SP/phạm vi) |
| **Lý do thay đổi** | Không yêu cầu | Bắt buộc nếu đã có hiệu lực |
| **Audit log** | Ghi thông tin tạo | Ghi đầy đủ old → new values |
| **Cảnh báo** | Minimal | Nhiều cảnh báo (đặc biệt nếu đã có hiệu lực) |
| **Snapshot công thức** | Luôn luôn | Chỉ khi thay đổi Price Code |

**Quan hệ với các module khác:**
- Module **Quản lý Mã giá (Price Code)**: Cung cấp QTTG bán và công thức tính giá mới
- Module **Audit Log**: Lưu trữ lịch sử thay đổi đầy đủ

**Lưu ý nghiệp vụ quan trọng:**
- Không cho phép cập nhật chính sách Inactive (BR-CSGIABAN-07)
- Yêu cầu lý do cập nhật khi chính sách đã có hiệu lực (BR-CSGIABAN-08)
- Ghi audit log đầy đủ với format old → new values (BR-CSGIABAN-06)
- Hiển thị cảnh báo rõ ràng về tác động khi cập nhật chính sách đang hoạt động

**Tham chiếu:**
- TONG-QUAN.md - Section 5: Business Rules
- UC-CSGIABAN-01-TAO-MOI.md - Use case liên quan
- DEMO.MD - Quy tắc nghiệp vụ chi tiết
