# Use Case UC-5: Activate Bảng giá

---

| **Use Case ID** | **UC-5** |
|-----------------|----------|
| **Use Case Name** | Activate Bảng giá |
| **Description** | Use Case "Activate Bảng giá" cho phép Admin kích hoạt bảng giá từ trạng thái Draft sang Active để áp dụng trong hệ thống. Hệ thống tự động chuyển bảng giá Active cũ (cùng loại) sang Inactive và snapshot toàn bộ giá tại thời điểm activate. |
| **Actor(s)** | Admin |
| **Priority** | Must Have |
| **Trigger** | Admin yêu cầu activate một Bảng giá đang Draft |

---

## Input

| Tên trường | Loại | Bắt buộc | Mô tả | Ràng buộc |
|------------|------|----------|-------|-----------|
| `priceListId` | Số | Có | ID bảng giá cần activate | Bảng giá phải tồn tại và đang Draft |
| `confirmMissingPrices` | Boolean | Không | Xác nhận activate dù thiếu giá | Bắt buộc = true nếu có giá thiếu |

---

## Output

### Trường hợp thành công:

| Tên trường | Loại | Mô tả |
|------------|------|-------|
| `id` | Số | ID bảng giá đã activate |
| `priceListCode` | Văn bản | Mã bảng giá |
| `priceListName` | Văn bản | Tên bảng giá |
| `status` | Văn bản | Trạng thái mới = "Active" |
| `activatedAt` | Ngày giờ | Thời gian activate |
| `activatedBy` | Văn bản | Người activate |
| `snapshotCount` | Số | Số mã giá đã snapshot |
| `previousActivePriceList` | Thông tin | Bảng giá Active cũ (đã chuyển Inactive) - nếu có |
| `message` | Văn bản | "Activate bảng giá thành công" |

### Trường hợp lỗi:

| Mã lỗi | Thông báo | Mô tả |
|--------|-----------|-------|
| `PRICE_LIST_NOT_FOUND` | "Bảng giá không tồn tại" | Không tìm thấy bảng giá |
| `INVALID_STATUS` | "Chỉ có thể activate bảng giá Draft" | Bảng giá không phải Draft |
| `MISSING_PRICES` | "Bảng giá còn thiếu giá" | Có mã giá chưa nhập đủ giá |
| `MISSING_PRICES_NOT_CONFIRMED` | "Vui lòng xác nhận activate với giá thiếu" | Thiếu giá nhưng chưa confirm |
| `ALL_PRICES_NULL` | "Không thể activate. Tất cả mã giá đều chưa có giá" | Không có mã giá nào có giá |

---

## Pre-Condition(s)

- Bảng giá đã tồn tại trong hệ thống
- Bảng giá đang có trạng thái Draft
- Admin đã đăng nhập và có quyền activate bảng giá
- (Optional) Bảng giá đã nhập đủ giá cho tất cả mã giá

---

## Post-Condition(s)

- Bảng giá chuyển sang trạng thái Active
- Hệ thống snapshot toàn bộ giá tại thời điểm activate
- Bảng giá Active cũ (cùng loại) tự động chuyển sang Inactive
- Bảng giá Active mới được áp dụng trong hệ thống
- Hệ thống ghi nhận thông tin người activate và thời gian activate
- Audit log ghi nhận hành động ACTIVATE và AUTO_DEACTIVATE

---

## Basic Flow

1. Admin yêu cầu activate một bảng giá đang Draft
2. Hệ thống kiểm tra tính hợp lệ:
   - Bảng giá tồn tại
   - Bảng giá đang Draft
3. Hệ thống kiểm tra giá:
   - Đếm số mã giá đã nhập đủ giá (mua VÀ bán)
   - Đếm số mã giá còn thiếu giá
4. Nếu có mã giá thiếu giá:
   - Hệ thống hiển thị danh sách mã giá thiếu giá chi tiết
   - Yêu cầu Admin xác nhận: "Activate dù thiếu giá?"
   - Nếu Admin không xác nhận → Use case kết thúc
5. Hệ thống kiểm tra bảng giá Active cũ:
   - Tìm bảng giá Active cùng loại (priceListType)
   - Nếu tìm thấy → Chuẩn bị chuyển sang Inactive
6. Hệ thống thực hiện activate:
   - **Snapshot** toàn bộ giá của tất cả mã giá (lưu giá tại thời điểm này)
   - Chuyển status từ Draft → Active
   - Ghi nhận thời gian activate (activatedAt)
   - Ghi nhận người activate (activatedBy)
7. Nếu có bảng giá Active cũ:
   - Hệ thống tự động chuyển bảng cũ từ Active → Inactive
   - Ghi nhận thời gian deactivate
   - Ghi audit log: AUTO_DEACTIVATE
8. Hệ thống ghi audit log:
   - Bảng giá mới: Action = ACTIVATE
   - Bảng giá cũ: Action = AUTO_DEACTIVATE (nếu có)
9. Hệ thống trả về kết quả thành công với thông tin:
   - Bảng giá mới đã Active
   - Số mã giá đã snapshot
   - Thông tin bảng giá cũ (nếu có)

Use case kết thúc.

---

## Alternative Flow

### 4a. Có mã giá thiếu giá, Admin xác nhận tiếp tục

4a. Hệ thống phát hiện có mã giá thiếu giá (mua hoặc bán = NULL)

4a1. Hệ thống hiển thị cảnh báo:
```
⚠️ CẢNH BÁO: Bảng giá còn thiếu giá

Tổng số mã giá: 7
Đã nhập đủ giá: 4 (57%)
Còn thiếu giá: 3 (43%)

Chi tiết mã giá thiếu:
  1. NHANVRTL - Thiếu giá bán
  2. MVRTL - Thiếu cả giá mua và bán
  3. BANVI - Thiếu giá mua

Bạn có chắc chắn muốn Activate bảng giá này?
```

4a2. Admin xác nhận: "Tiếp tục Activate"

4a3. Use case quay lại bước 5 (tiếp tục activate)

---

## Exception Flow

### 2a. Bảng giá không tồn tại

2a. Hệ thống không tìm thấy bảng giá với ID được cung cấp

2a1. Hệ thống trả về lỗi: "Bảng giá không tồn tại hoặc đã bị xóa."

2a2. Use case kết thúc

### 2b. Bảng giá không phải Draft

2b. Hệ thống phát hiện bảng giá đang ở trạng thái Active hoặc Inactive

2b1. Hệ thống trả về lỗi: "Chỉ có thể activate bảng giá Draft. Bảng giá này đang ở trạng thái [Status]."

2b2. Use case kết thúc

### 4b. Tất cả mã giá đều chưa có giá

4b. Hệ thống phát hiện **tất cả** mã giá đều chưa nhập giá (NULL/NULL)

4b1. Hệ thống trả về lỗi: "Không thể activate. Tất cả mã giá đều chưa có giá. Vui lòng nhập giá trước khi activate."

4b2. Use case kết thúc

### 4c. Admin không xác nhận activate với giá thiếu

4c. Admin từ chối activate khi có giá thiếu

4c1. Hệ thống hủy thao tác

4c2. Admin quay lại UC02 để nhập giá

4c3. Use case kết thúc

---

## Business Rules

### BR-UC05-001: Chỉ Admin được activate

- Chỉ Admin mới có quyền activate bảng giá
- Nhân viên không có quyền này
- Lý do: Activate bảng giá ảnh hưởng toàn hệ thống (giá mua/bán)

### BR-UC05-002: Chỉ activate bảng giá Draft

- Chỉ có thể activate bảng giá đang ở trạng thái **Draft**
- Bảng giá **Active**: Đã được activate, không cần activate lại
- Bảng giá **Inactive**: Không thể activate lại (one-way transition)
- Mục đích: Đảm bảo luồng trạng thái Draft → Active → Inactive

**Ví dụ:**
```
Bảng giá Draft:
  ✅ Có thể Activate

Bảng giá Active:
  ❌ Không thể Activate (đã Active rồi)

Bảng giá Inactive:
  ❌ Không thể Activate lại
  → Phải tạo bảng giá mới hoặc Sao chép (UC06)
```

### BR-UC05-003: Kiểm tra giá đầy đủ

Hệ thống kiểm tra giá trước khi activate:

**Cấp độ 1: Bắt buộc**
- Ít nhất **một mã giá** phải có giá (không phải tất cả NULL)
- Nếu tất cả mã giá đều NULL → Từ chối activate

**Cấp độ 2: Cảnh báo (tùy cấu hình)**
- Nếu có mã giá thiếu giá (mua hoặc bán = NULL)
- Hệ thống hiển thị cảnh báo chi tiết
- Yêu cầu Admin xác nhận
- Nếu không xác nhận → Hủy activate

**Ví dụ:**
```
Bảng giá có 7 mã giá:
  - 0/7 có giá → ❌ Từ chối (tất cả NULL)
  - 1/7 có giá → ⚠️ Cảnh báo (yêu cầu xác nhận)
  - 7/7 có giá → ✅ Cho phép (không cảnh báo)
```

### BR-UC05-004: Auto-transition bảng giá cũ

Khi activate bảng giá mới:
- Hệ thống **tự động** tìm bảng giá Active cũ **cùng loại** (priceListType)
- **Tự động** chuyển bảng cũ từ Active → Inactive
- Ghi audit log: AUTO_DEACTIVATE
- Đảm bảo: **Chỉ 1 bảng Active** cho mỗi loại tại một thời điểm

**Ví dụ:**
```
Trước activate:
  - "Bảng giá vàng - 03/03/2026" (GOLD): Active
  - "Bảng giá vàng - 04/03/2026" (GOLD): Draft

Admin activate "Bảng giá vàng - 04/03/2026":
→ Hệ thống tự động:
  1. "Bảng giá vàng - 04/03/2026": Draft → Active ✅
  2. "Bảng giá vàng - 03/03/2026": Active → Inactive ⚡ (auto)

Sau activate:
  - "Bảng giá vàng - 03/03/2026" (GOLD): Inactive
  - "Bảng giá vàng - 04/03/2026" (GOLD): Active
  
→ Chỉ 1 bảng GOLD Active tại một thời điểm
```

### BR-UC05-005: Snapshot mechanism

Tại thời điểm activate, hệ thống **snapshot** toàn bộ giá:
- Lưu giá của **tất cả mã giá** tại thời điểm activate
- Giá được "đóng băng" (frozen), không thay đổi trong tương lai
- Dù hệ số mã giá thay đổi sau này → Bảng giá Active **không** bị ảnh hưởng

**Ví dụ:**
```
04/03/2026 15:46 - Activate bảng giá:
  NHANVRTL: 85M / 87M
  QTVRTL (Hệ số 1): 85M / 87M (tính từ NHANVRTL)
  MNVT9999 (Hệ số 0.994): 84.49M / 86.48M

→ Snapshot: Lưu giá 85M, 87M, 84.49M, 86.48M

05/03/2026 - Admin cập nhật hệ số QTVRTL từ 1 → 1.05:
→ Bảng giá 04/03 vẫn giữ nguyên:
  QTVRTL: 85M / 87M (không đổi, đã snapshot)

→ Bảng giá mới tạo sau 05/03:
  QTVRTL: 85M × 1.05 = 89.25M (hệ số mới)
```

### BR-UC05-006: Một loại - Một Active

- Mỗi loại bảng giá (priceListType) chỉ có **duy nhất 1 bảng Active**
- Không thể có 2 bảng GOLD Active cùng lúc
- Có thể có nhiều bảng Draft cùng loại
- Có thể có nhiều bảng Inactive cùng loại
- Khi activate bảng mới → Bảng Active cũ **tự động** chuyển Inactive

**Ví dụ:**
```
Loại GOLD:
  - Bảng A: Draft ✅
  - Bảng B: Draft ✅
  - Bảng C: Active ✅ (chỉ 1)
  - Bảng D: Inactive ✅
  - Bảng E: Inactive ✅

Activate Bảng A:
→ Bảng A: Active ✅
→ Bảng C: Inactive ⚡ (auto)

Loại SILVER:
  - Bảng F: Active ✅ (SILVER khác GOLD)
→ Không ảnh hưởng
```

### BR-UC05-007: Ghi nhận audit log

Mỗi lần activate, hệ thống ghi nhận đầy đủ:

**Bảng giá mới được activate:**
- Action: ACTIVATE
- Thời gian activate (activatedAt)
- Người activate (activatedBy)
- Trạng thái trước: Draft
- Trạng thái sau: Active
- Snapshot count: Số mã giá đã snapshot

**Bảng giá cũ bị auto-deactivate:**
- Action: AUTO_DEACTIVATE
- Thời gian deactivate
- Triggered by: Activate bảng giá mới
- Trạng thái trước: Active
- Trạng thái sau: Inactive

### BR-UC05-008: Không ảnh hưởng bảng giá khác loại

Việc activate bảng giá **chỉ ảnh hưởng** bảng giá **cùng loại**:
- Activate bảng GOLD → Chỉ ảnh hưởng bảng GOLD Active cũ
- Không ảnh hưởng bảng SILVER, PLATINUM, v.v.
- Mỗi loại độc lập về trạng thái Active

**Ví dụ:**
```
Trước:
  - Bảng GOLD-A: Active
  - Bảng GOLD-B: Draft
  - Bảng SILVER-C: Active
  - Bảng SILVER-D: Draft

Activate GOLD-B:
→ GOLD-A: Inactive ⚡
→ GOLD-B: Active ✅
→ SILVER-C: Active ✅ (không đổi)
→ SILVER-D: Draft (không đổi)
```

---

## Diagrams

### 1. Use Case Diagram - UC05: Activate Bảng giá

```mermaid
graph LR
    Actor["👤 Admin"]
    
    UC05["UC05: Activate Bảng giá<br/>(Draft → Active + Auto Inactive cũ)"]
    
    Actor -->|Thực hiện| UC05
    
    style UC05 fill:#bbdefb,stroke:#1976d2,stroke-width:2px
    style Actor fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

### 2. Activity Diagram - Luồng Activate Bảng giá

```mermaid
flowchart TD
    Start([Bắt đầu]) --> Input[Admin yêu cầu activate<br/>priceListId]
    
    Input --> CheckExists{Bảng giá<br/>tồn tại?}
    
    CheckExists -->|Không| Error1[Lỗi: Không tìm thấy]
    Error1 --> End1([Kết thúc])
    
    CheckExists -->|Có| CheckStatus{Status<br/>= Draft?}
    
    CheckStatus -->|Không| Error2[Lỗi: Chỉ activate Draft]
    Error2 --> End1
    
    CheckStatus -->|Có| CountPrices[Đếm số mã giá:<br/>- Đã nhập đủ giá<br/>- Còn thiếu giá]
    
    CountPrices --> CheckAllNull{Tất cả<br/>đều NULL?}
    
    CheckAllNull -->|Có| Error3[Lỗi: Không có giá nào]
    Error3 --> End1
    
    CheckAllNull -->|Không| CheckMissing{Có mã giá<br/>thiếu giá?}
    
    CheckMissing -->|Có| ShowWarning[Hiển thị cảnh báo:<br/>Danh sách mã giá thiếu giá]
    ShowWarning --> Confirm{Admin<br/>xác nhận?}
    
    Confirm -->|Không| Cancel[Hủy activate]
    Cancel --> End1
    
    Confirm -->|Có| FindOldActive
    CheckMissing -->|Không| FindOldActive[Tìm bảng giá Active cũ<br/>cùng loại]
    
    FindOldActive --> HasOld{Có bảng<br/>Active cũ?}
    
    HasOld -->|Có| PrepareDeactivate[Chuẩn bị chuyển<br/>bảng cũ → Inactive]
    HasOld -->|Không| Snapshot
    
    PrepareDeactivate --> Snapshot[Snapshot toàn bộ giá<br/>của tất cả mã giá]
    
    Snapshot --> ActivateNew[Cập nhật bảng mới:<br/>- Status = Active<br/>- activatedAt<br/>- activatedBy]
    
    ActivateNew --> CheckOld2{Có bảng<br/>cũ?}
    
    CheckOld2 -->|Có| DeactivateOld[Tự động chuyển bảng cũ:<br/>- Status = Inactive<br/>- deactivatedAt]
    CheckOld2 -->|Không| LogNew
    
    DeactivateOld --> LogBoth[Ghi audit log:<br/>- ACTIVATE: bảng mới<br/>- AUTO_DEACTIVATE: bảng cũ]
    LogBoth --> Success
    
    LogNew[Ghi audit log:<br/>- ACTIVATE: bảng mới] --> Success[Trả về kết quả thành công:<br/>- Bảng mới Active<br/>- Snapshot count<br/>- Bảng cũ Inactive]
    
    Success --> End2([Kết thúc])
    
    style Start fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style End1 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style End2 fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style Success fill:#a5d6a7,stroke:#388e3c,stroke-width:2px
    style Error1 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error2 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Error3 fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style ShowWarning fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Snapshot fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style DeactivateOld fill:#ffccbc,stroke:#e64a19,stroke-width:2px
```

### 3. Sequence Diagram - Activate Bảng giá với Auto-transition

```mermaid
sequenceDiagram
    actor Admin as 👤 Admin
    participant Controller as Controller
    participant Service as PriceListService
    participant SnapshotService as SnapshotService
    participant AuditService as AuditService
    participant DB as Database
    
    Admin->>Controller: 1. Yêu cầu activate (priceListId)
    Controller->>Service: 2. Activate price list
    
    Service->>DB: 3. Query bảng giá by ID
    DB-->>Service: Thông tin bảng giá
    
    alt Không tìm thấy
        Service-->>Controller: Lỗi: Không tồn tại
        Controller-->>Admin: "Bảng giá không tồn tại"
    else Tìm thấy
        Service->>Service: 4. Kiểm tra status = Draft
        
        alt Status không phải Draft
            Service-->>Controller: Lỗi: Không phải Draft
            Controller-->>Admin: "Chỉ có thể activate bảng giá Draft"
        else Status = Draft
            Service->>DB: 5. Đếm số mã giá có giá / thiếu giá
            DB-->>Service: Statistics
            
            alt Tất cả đều NULL
                Service-->>Controller: Lỗi: Không có giá
                Controller-->>Admin: "Tất cả mã giá đều chưa có giá"
            else Có ít nhất 1 giá
                
                alt Có mã giá thiếu giá
                    Service-->>Controller: Danh sách thiếu giá
                    Controller-->>Admin: ⚠️ "Có 3 mã giá thiếu. Xác nhận?"
                    Admin->>Controller: Xác nhận tiếp tục
                    Controller->>Service: Confirmed = true
                end
                
                Service->>DB: 6. Tìm bảng Active cũ cùng loại
                DB-->>Service: Old active price list (nếu có)
                
                Service->>SnapshotService: 7. Snapshot toàn bộ giá
                SnapshotService->>DB: Lưu snapshot cho từng entry
                DB-->>SnapshotService: Snapshot saved
                SnapshotService-->>Service: Snapshot count
                
                Service->>DB: 8. Update bảng mới: Draft → Active
                DB-->>Service: Updated
                
                alt Có bảng Active cũ
                    Service->>DB: 9. Update bảng cũ: Active → Inactive
                    DB-->>Service: Updated
                    
                    Service->>AuditService: 10. Log AUTO_DEACTIVATE
                    AuditService->>DB: Save audit log
                end
                
                Service->>AuditService: 11. Log ACTIVATE
                AuditService->>DB: Save audit log
                
                Service-->>Controller: 12. Kết quả thành công
                Controller-->>Admin: 13. ✅ "Activate thành công"
                Note over Admin: - Bảng mới: Active<br/>- Snapshot: 7 mã giá<br/>- Bảng cũ: Inactive
            end
        end
    end
```

**Giải thích Sequence Diagram:**

**Kiểm tra điều kiện (Bước 3-5):**
- Tìm bảng giá theo ID
- Kiểm tra status = Draft
- Đếm số mã giá có giá và thiếu giá
- Nếu tất cả NULL → Từ chối

**Xác nhận thiếu giá (Bước 5 - conditional):**
- Nếu có mã giá thiếu giá
- Hiển thị cảnh báo chi tiết
- Admin xác nhận tiếp tục

**Snapshot và Activate (Bước 6-8):**
- Tìm bảng Active cũ (cùng loại)
- SnapshotService snapshot toàn bộ giá
- Update bảng mới: Draft → Active

**Auto-transition (Bước 9-11):**
- Nếu có bảng cũ: Active → Inactive
- AuditService ghi 2 logs:
  * AUTO_DEACTIVATE (bảng cũ)
  * ACTIVATE (bảng mới)

---

### 4. Class Diagram

```mermaid
classDiagram
    class PriceListController {
        +activatePriceList() Kết quả
    }
    
    class PriceListService {
        +activate() Bảng giá Active
        +validateDraftStatus() Boolean
        +checkMissingPrices() Statistics
        +findActiveByType() Bảng giá cũ
    }
    
    class SnapshotService {
        +snapshotAllPrices() Void
        +freezePriceEntry() Void
        +recordSnapshotTime() Void
    }
    
    class TransitionService {
        +autoDeactivateOld() Void
        +ensureSingleActive() Void
    }
    
    class AuditService {
        +logActivate() Void
        +logAutoDeactivate() Void
    }
    
    class PriceList {
        +id: ID
        +priceListType: Loại
        +status: Trạng thái
        +activatedAt: Thời gian activate
        +activatedBy: Người activate
        +snapshotAt: Thời gian snapshot
    }
    
    class PriceListEntry {
        +id: ID
        +priceCode: Mã giá
        +buyingPrice: Giá mua
        +sellingPrice: Giá bán
        +isSnapshot: Đã snapshot
        +snapshotAt: Thời gian snapshot
    }
    
    class MissingPriceReport {
        +totalCount: Tổng số
        +filledCount: Đã nhập
        +missingCount: Thiếu giá
        +missingEntries: Danh sách thiếu
    }
    
    class ActivateResult {
        +newActivePriceList: Bảng mới
        +oldInactivePriceList: Bảng cũ
        +snapshotCount: Số snapshot
        +message: Thông báo
    }
    
    PriceListController --> PriceListService : sử dụng
    PriceListService --> SnapshotService : snapshot giá
    PriceListService --> TransitionService : auto-deactivate
    PriceListService --> AuditService : ghi log
    PriceListService --> MissingPriceReport : kiểm tra thiếu giá
    PriceListService --> ActivateResult : trả về
    SnapshotService --> PriceListEntry : snapshot từng entry
    TransitionService --> PriceList : chuyển trạng thái
    
    note for SnapshotService "Đóng băng giá<br/>tại thời điểm activate"
    note for TransitionService "Đảm bảo 1 Active<br/>per type"
    note for AuditService "Ghi 2 logs:<br/>ACTIVATE + AUTO_DEACTIVATE"
```

---

## Business Scenario

### Scenario 1: Activate bảng giá đủ giá thành công

```
Bảng giá: "Bảng giá vàng - 04/03/2026" (Draft)
Loại: GOLD
Tổng mã giá: 7
Đã nhập đủ giá: 7 (100%)

Trước activate:
  - "Bảng giá vàng - 03/03/2026" (GOLD): Active
  - "Bảng giá vàng - 04/03/2026" (GOLD): Draft ← Chuẩn bị activate

Admin thực hiện:
1. Chọn "Activate" từ danh sách hoặc chi tiết
2. Xác nhận activate

Hệ thống xử lý:
1. Kiểm tra: Draft ✅, Đủ giá ✅
2. Tìm bảng Active cũ: "Bảng giá vàng - 03/03/2026"
3. Snapshot 7 mã giá của bảng mới:
   - NHANVRTL: 86M / 88M (snapshot)
   - QTVRTL: 86M / 88M (snapshot, auto-calc)
   - MVRTL: 86M / 88M (snapshot, auto-calc)
   - MNVT9999: 85.48M / 87.47M (snapshot, auto-calc)
   - MNVT999: 85.39M / 87.38M (snapshot, auto-calc)
   - BANVI: 86M / 88M (snapshot, auto-calc)
   - VHT99.99: 86M / 88M (snapshot, auto-calc)
4. Activate bảng mới:
   - Status: Draft → Active
   - activatedAt: 04/03/2026 16:00
   - activatedBy: Admin A
5. Auto-deactivate bảng cũ:
   - "Bảng giá vàng - 03/03/2026": Active → Inactive
   - deactivatedAt: 04/03/2026 16:00
   - deactivatedBy: AUTO (triggered by new activation)

Audit Log:
  - 16:00:00 - ACTIVATE - "Bảng giá vàng - 04/03/2026"
  - 16:00:01 - AUTO_DEACTIVATE - "Bảng giá vàng - 03/03/2026"

Kết quả:
✅ Activate thành công
  - Bảng mới: "Bảng giá vàng - 04/03/2026" → Active
  - Snapshot: 7 mã giá
  - Bảng cũ: "Bảng giá vàng - 03/03/2026" → Inactive

Sau activate:
  - Loại GOLD chỉ có 1 bảng Active: "Bảng giá vàng - 04/03/2026"
  - Giá đã được snapshot, không thay đổi dù hệ số mã giá thay đổi sau này
```

### Scenario 2: Activate với giá thiếu (có xác nhận)

```
Bảng giá: "Bảng giá vàng - 05/03/2026" (Draft)
Loại: GOLD
Tổng mã giá: 7
Đã nhập đủ giá: 4 (57%)
Còn thiếu giá: 3 (43%)

Admin thực hiện:
1. Chọn "Activate"

Hệ thống kiểm tra:
→ Phát hiện 3 mã giá thiếu giá

Hiển thị cảnh báo:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ CẢNH BÁO: Bảng giá còn thiếu giá

Tổng số mã giá: 7
Đã nhập đủ giá: 4 (57%)
Còn thiếu giá: 3 (43%)

Chi tiết mã giá thiếu:
  1. NHANVRTL - Thiếu giá bán
     Giá mua: 85M ✅ | Giá bán: NULL ⚠️
  
  2. MVRTL - Thiếu cả 2 giá
     Giá mua: NULL ⚠️ | Giá bán: NULL ⚠️
  
  3. BANVI - Thiếu giá mua
     Giá mua: NULL ⚠️ | Giá bán: 88M ✅

Bạn có chắc chắn muốn Activate bảng giá này?
[Hủy] [Tiếp tục Activate]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Admin chọn: "Tiếp tục Activate"

Hệ thống xử lý:
1. Snapshot tất cả giá (kể cả NULL)
2. Activate bảng mới: Draft → Active
3. Auto-deactivate bảng cũ: Active → Inactive

Kết quả:
✅ Activate thành công (với cảnh báo)
⚠️ Lưu ý: 3 mã giá thiếu giá vẫn ở trạng thái thiếu

→ Bảng giá Active nhưng một số mã giá chưa có đủ giá
→ Có thể cập nhật sau bằng cách tạo bảng giá mới
```

### Scenario 3: Không thể activate (tất cả NULL)

```
Bảng giá: "Bảng giá bạc - 05/03/2026" (Draft)
Loại: SILVER
Tổng mã giá: 5
Đã nhập đủ giá: 0 (0%)

Admin thực hiện:
1. Chọn "Activate"

Hệ thống kiểm tra:
→ Tất cả 5 mã giá đều chưa có giá (NULL/NULL)

Kết quả:
❌ Lỗi: "Không thể activate. Tất cả mã giá đều chưa có giá. 
         Vui lòng nhập giá trước khi activate."

Hướng dẫn:
→ Admin cần:
  1. Quay lại UC02: Cập nhật giá
  2. Nhập giá cho ít nhất 1 mã giá
  3. Sau đó mới có thể Activate
```

### Scenario 4: Admin hủy activate khi thấy cảnh báo thiếu giá

```
Bảng giá: "Bảng giá vàng - 06/03/2026" (Draft)
Thiếu giá: 2/7 mã giá

Admin thực hiện:
1. Chọn "Activate"
2. Hệ thống hiển thị cảnh báo thiếu giá
3. Admin xem lại và quyết định: "Hủy"

Kết quả:
❌ Hủy activate

Admin tiếp tục:
→ Chuyển sang UC02: Cập nhật giá
→ Nhập đủ giá cho 2 mã giá còn thiếu
→ Quay lại Activate (lần này không cảnh báo)
```

### Scenario 5: Activate bảng đầu tiên của loại mới

```
Bảng giá: "Bảng giá bạc - 05/03/2026" (Draft)
Loại: SILVER (lần đầu tạo loại này)
Tổng mã giá: 5
Đã nhập đủ giá: 5 (100%)

Trước activate:
  - Không có bảng SILVER Active nào

Admin thực hiện:
1. Chọn "Activate"
2. Xác nhận

Hệ thống xử lý:
1. Kiểm tra: Draft ✅, Đủ giá ✅
2. Tìm bảng Active cũ loại SILVER: Không tìm thấy
3. Snapshot 5 mã giá
4. Activate bảng mới: Draft → Active
5. Không có bảng cũ để deactivate

Kết quả:
✅ Activate thành công
  - Bảng mới: "Bảng giá bạc - 05/03/2026" → Active
  - Snapshot: 5 mã giá
  - Không có bảng cũ

Sau activate:
  - Loại SILVER có 1 bảng Active (bảng đầu tiên)
```

### Scenario 6: Snapshot ngăn ảnh hưởng thay đổi hệ số

```
04/03/2026 16:00 - Activate "Bảng giá vàng - 04/03"
Snapshot:
  NHANVRTL: 86M / 88M
  QTVRTL (Hệ số 1): 86M / 88M

05/03/2026 10:00 - Admin cập nhật hệ số QTVRTL:
  Hệ số mua: 1 → 1.05
  Hệ số bán: 1 → 1.05

Kiểm tra bảng giá 04/03:
→ QTVRTL vẫn giữ nguyên: 86M / 88M ✅
→ Không bị ảnh hưởng bởi thay đổi hệ số (đã snapshot)

Tạo bảng giá mới 06/03:
→ QTVRTL tính lại: 86M × 1.05 = 90.3M
→ Bảng mới sử dụng hệ số mới

Kết luận:
✅ Snapshot đảm bảo bảng Active không thay đổi
✅ Chỉ bảng mới mới sử dụng hệ số mới
```

---
