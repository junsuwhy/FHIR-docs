---
fhir_reference: https://build.fhir.org/appointment.html
---


在 FHIR 中，`Appointment` 用來描述**「已提出或已確認的預約行為」**，可涵蓋門診看診、健檢檢查、手術房排程等情境。  
對健檢機構而言，`Appointment` 通常結合 **病人、檢查服務、設備/地點** 與 **具體時段 (Slot)** 共同運作：

- **核心欄位**  
  - `status`：proposed／pending／booked／arrived／fulfilled／cancelled／noshow／entered-in-error／checked-in／waitlist（追蹤全流程）  
  - `start` / `end`：預約起訖時間（ISO-8601）  
  - `slot[]`：指向一或多個 `Slot`，確保該時段已被鎖定  
  - `participant[]`：列出所有關係人／資源（病人、醫師、設備、地點…），並標示各自 `status`  
  - `serviceType` / `serviceCategory`：可用來表示健檢套餐、檢查目的  
  - `appointmentType`：預約類型/樣式，健檢場景建議使用 `CHECKUP`（例行檢查）
  - `reason`：就醫/檢查原因，可包含診斷碼或檢查原因碼
  - `basedOn`：若由公司、保險或轉診單代為排檢，可連到 `ServiceRequest`  
  - `recurrenceTemplate`：重複約診模板，定義如何自動生成一系列約診（健檢方案應該用不到，未實做）
  - `virtualService`：視訊、電話等遠距服務詳情（COVID-19 後新增重要功能）
  - `account`：關聯至帳務資訊（健檢費用處理）
  
### 重複約診模式

> **備註**：考量到健檢業務特性，本專案初期將**不實作** `recurrenceTemplate` 相關的重複約診功能。**
  - **模板式重複（Template-based）**：使用 `recurrenceTemplate` 設定重複規則（每週、每月等）
  - **序列式串接（Series）**：通過 `previousAppointment` 建立約診鏈，適合不規則追蹤
  - **取代關係（Replacement）**：使用 `replaces` 指向被取代的約診。這在改期時特別重要，它能建立新舊約診之間的明確關聯，方便追蹤預約的變更歷史。

- **預約生命週期管理**
  - `created`：預約建立時間（首次提出）
  - `cancellationDate`：取消日期時間
  - `cancellationReason`：取消原因（CodeableConcept）
  - `class`：預約類別（門診、住院、急診、虛擬等）
  - #priority ：優先級別（0-9，0最高優先（錯誤），實做需用 CodeableConcept 寫法，值請參考 [[ActPriority]]）
  - `minutesDuration`：預計持續時間（分鐘）

- **參與者管理（Participant）**
  - `type`：參與者類型（CodeableConcept）- 主診醫師、技師、設備、翻譯等
  - `period`：參與時間範圍（可能只參與部分時段）
  - `required`：是否必要（required/optional/information-only）
  - `status`：參與狀態（needs-action/accepted/declined/tentative）

- **患者指示與備註**
  - `patientInstruction`：給患者的指示（如空腹8小時、攜帶文件等）
  - `note`：內部註記（Annotation[]）
  - `comment`：簡短備註

> **取消 / 改期**：  
> - 取消：把 `Appointment.status` 改 `cancelled`，設定 `cancellationDate` 和 `cancellationReason`，並把相關 slot 的 `status` 改回 `free`  
> - 改期：建立新 `Appointment` 並使用 `replaces` 指向原約診，原約診狀態改為 `entered-in-error`
> - 重複約診取消：可個別取消特定次數（設定 `occurrenceChanged`），或取消整個系列

```json
{
  "resourceType": "Appointment",
  "meta": {
    "profile": [
      "http://example.org/fhir/StructureDefinition/tw-health-check-appointment"
    ]
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/appointment-id",
      "value": "HC-20250610-0001"
    }
  ],
  "id": "hc-20250610-0001",
  "status": "booked",
  "class": {
    "coding": [
      {
        "system": "http://terminology.hl7.org.tw/CodeSystem/v3-ActCode",
        "code": "AMB",
        "display": "門診"
      }
    ]
  },
  "serviceCategory": [
    {
      "coding": [
        {
          "system": "http://example.org/service-category",
          "code": "health-check",
          "display": "健康檢查"
        }
      ]
    }
  ],
  "serviceType": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-type",
          "code": "114",
          "display": "Health assessment"
        }
      ],
      "text": "健檢套餐 A"
    }
  ],
  "reason": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "185349003",
          "display": "Health check"
        }
      ],
      "text": "年度健康檢查"
    }
  ],
  "appointmentType": {
    "coding": [
      {
        "system": "http://terminology.hl7.org/ValueSet/v2-0276",
        "code": "CHECKUP",
        "display": "A routine check-up, such as an annual physical"
      }
    ]
  },
  "priority": {
    "coding": [
      {
        "system": "http://terminology.hl7.org/CodeSystem/v3-ActPriority",
        "code": "ROU",
        "display": "Routine"
      }
    ]
  },
  "description": "年度健檢：血液、X 光、超音波",
  "start": "2025-06-10T08:30:00+08:00",
  "end":   "2025-06-10T10:30:00+08:00",
  "minutesDuration": 120,
  "slot": [
    { "reference": "Slot/xray-001-20250610-0830" },
    { "reference": "Slot/ultrasound-001-20250610-0900" }
  ],
  "created": "2025-05-20T10:15:00+08:00",
  "participant": [
    {
      "actor": { "reference": "Patient/p-0001", "display": "張大同" },
      "status": "accepted",
      "required": "required",
      "type": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/encounter-participant-type",
              "code": "SBJ",
              "display": "subject"
            }
          ]
        }
      ]
    },
    {
      "actor": { "reference": "PractitionerRole/tech-001", "display": "放射技師 A" },
      "status": "accepted",
      "required": "required",
      "type": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/encounter-participant-type",
              "code": "ATND",
              "display": "attender"
            }
          ]
        }
      ],
      "period": {
        "start": "2025-06-10T08:30:00+08:00",
        "end": "2025-06-10T09:00:00+08:00"
      }
    },
    {
      "actor": { "reference": "Device/xray-001", "display": "X 光機 XRAY-001" },
      "status": "accepted",
      "required": "required"
    },
    {
      "actor": { "reference": "Location/xray-room-1", "display": "X 光室 1" },
      "status": "accepted",
      "required": "required"
    },
    {
      "actor": { "reference": "HealthcareService/health-check-A", "display": "健檢服務 A" },
      "status": "accepted",
      "required": "information-only"
    }
  ],
  "basedOn": [
    {
      "reference": "ServiceRequest/corp-check-2025-0001",
      "display": "ABC 公司年度員工健檢申請"
    }
  ],
  "patientInstruction": [
    {
      "coding": [
        {
          "system": "http://example.org/patient-instructions",
          "code": "fasting-8h",
          "display": "空腹8小時"
        }
      ],
      "text": "請於檢查前8小時開始禁食，可飲用少量白開水"
    }
  ],
  "comment": "公司統一結帳；請病人提前 15 分鐘報到。",
  "note": [
    {
      "authorReference": {
        "reference": "Practitioner/admin-001"
      },
      "time": "2025-05-20T10:20:00+08:00",
      "text": "已電話確認病人了解空腹要求"
    }
  ],
  "account": [
    {
      "reference": "Account/corp-abc-2025",
      "display": "ABC 公司健檢帳戶"
    }
  ]
}
```

```
Patient ─┐
         ├─▶ Appointment ──▶ Slot ──▶ Schedule
Device ──┤               ▲          ▲
Location ─┘             (多筆)      │
HealthcareService ────────┘        │
Account ─────────────────┘         │
ServiceRequest ──────────┘         │
```

## 健檢場景應用對照表

| 功能需求           | 對應方式                                                   |
| -------------- | ------------------------------------------------------ |
| 預約狀態追蹤         | `Appointment.status` 與 `participant[].status`          |
| 不同健檢套餐 (A/B/C) | `Appointment.serviceType`／`HealthcareService`          |
| 公司批次代為預約       | `basedOn` 連到 `ServiceRequest`；`account` 連到公司帳戶         |
| 檢查設備調度         | 在 `participant` 同時列入 `Device`／`Location`；利用 slot 鎖定時段  |
| 改期或取消          | 使用 `replaces` 建立新預約；原預約 `status` 改為 `entered-in-error` |
| 遠距醫療諮詢         | 使用 `virtualService` 記錄視訊連結、密碼等資訊                       |
| 空腹等檢前準備        | 使用 `patientInstruction` 提供結構化指示                        |
| 重複健檢追蹤         | 使用 `recurrenceTemplate` 或 `previousAppointment` 串接     |

## FHIR 資源關聯

| FHIR 資源                           | 角色 / 功能         |
| --------------------------------- | --------------- |
| `Patient`                         | 被檢查者            |
| `Practitioner / PractitionerRole` | 執行檢查的醫師或技師      |
| [[Device]]                        | 檢查設備 (CT、X 光…)  |
| `Location`                        | 實際檢查房、診間        |
| [[HealthcareService]]             | 健檢服務或套餐分類       |
| [[Schedule]]                      | 班表大範圍           |
| [[Slot]]                          | 可預約時間格          |
| `ServiceRequest`                  | 公司、保險或門診轉介單（選用） |
| `Account`                         | 帳務處理（公司或個人帳戶）   |

## 實作建議

1. **狀態機管理**
   - 建立明確的狀態轉換規則（如 `proposed` → `booked` → `arrived` → `fulfilled`）
   - 使用資料庫觸發器或應用層邏輯確保狀態轉換的一致性

2. **公司／保險批次預約**
   - 先由 HR 送出 `ServiceRequest`，系統批次建立 `Appointment`
   - 使用 `account` 關聯公司帳戶進行統一結帳
   - 可用 Extension 記錄額外的公司資訊（部門、員工編號等）

3. **多檢查項目排程**
   - 後端實作智慧排程演算法，考慮檢查順序（如空腹項目優先）
   - 自動計算各檢查項目間的緩衝時間
   - 使用 `participant[].period` 記錄各資源的參與時段

4. **重複約診管理**
   - 定期健檢追蹤：使用 `recurrenceTemplate` 定義規則
   - 不規則追蹤：使用 `previousAppointment` 建立約診鏈
   - 支援個別修改特定次數的重複約診

5. **效能優化**
   - 使用複合索引優化查詢（如 `status + start`）
   - 定期清理過期的 `proposed` 狀態預約
   - 考慮使用快取儲存熱門時段的可用性資訊

6. **虛擬服務整合**
   - 健檢報告解說可使用 `virtualService` 提供線上諮詢
   - 記錄視訊會議連結、密碼、備用聯絡方式等

> **小提醒**
> - `class` 欄位可區分門診、住院、急診、虛擬等類型
> - 建議使用 `patientInstruction` 提供結構化的檢前準備指示
> - 善用 `note` 記錄內部溝通歷程，提升服務品質

[FHIR文件參考](https://build.fhir.org/appointment.html)