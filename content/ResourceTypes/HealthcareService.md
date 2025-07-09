---

fhir_reference: https://build.fhir.org/healthcareservice.html

---

在 FHIR 中，`HealthcareService` 用來描述**機構對外提供的具體「服務內容」**──  
例如健檢中心提供的「標準健檢套餐 A」或「高階心臟檢查」，又或醫院的「骨科門診」、「職業醫學評估」等。  
在預約流程裡，它常被放進 `Appointment.serviceType`，或直接由前端列為「可選方案」供民眾挑選。

## **關鍵屬性**：

- `appointmentRequired`：指示該服務是否**需要預約**。
- `referralRequired`：指示是否需要轉診才能使用此服務。
- `location`：指向提供該服務的**地點 (Location)** 資源。
- `providedBy`：指向提供該服務的**機構 (Organization)** 資源。
- `category` 和 `type`：分別表示服務的廣泛分類和具體類型 (使用 `CodeableConcept`)。
- `availability`：新的 `Availability` 型別，用於宣告**服務可用的時段**，包含 `availableTime` 和 `notAvailableTime` 等。
- `contact`：使用 `ExtendedContactDetail` 型別，提供更豐富的聯絡資訊，如用途、姓名、電話、地址等。
- `endpoint`：可以引用一個或多個 [[Endpoint]] 資源，描述提供該醫療服務的技術通訊點。


在 FHIR v6（CI-Build）中，`HealthcareService` 有以下重要更新：

1. **`availability`（`Availability` 型別）** 來宣告可服務時段，`availability.availableTime` 內再細列 `daysOfWeek / availableStartTime / availableEndTime` 等欄位，取代 R4 的 `availableTime`／`notAvailable` 直屬欄位。

2. **`contact`（`ExtendedContactDetail` 型別）** 取代 R4 的 `telecom`，提供更豐富的聯絡資訊結構，包含：
   - `purpose`：聯絡用途（如預約、查詢等）
   - `name`：聯絡人姓名
   - `telecom`：聯絡方式（電話、電子郵件等）
   - `address`：聯絡地址
   - `organization`：負責此聯絡方式的組織
   - `period`：此聯絡資訊的有效期間

3. **`meta`** 提供資源的元資料，包含：
   - `versionId`：版本識別碼
   - `lastUpdated`：最後更新時間
   - `profile`：符合的 Profile URL
   - `tag`：資源標籤（如服務分類）

4. **`photo`（`Attachment` 型別）** 用於識別服務的圖片，包含：
   - `contentType`：圖片的 MIME 類型
   - `url`：圖片的 URL
   - `title`：圖片標題
   - `creation`：圖片建立日期  

健檢中心常見的 `HealthcareService` 範例：

- **健檢套餐 A**：基本血液＋胸部 X 光  
- **健檢套餐 B**：套餐 A＋腹部超音波  
- **健檢套餐 C**：套餐 B＋腦部 MRI  
- **企業員工職業健檢**  
- **自費心臟斷層掃描（CT）**

> 一個機構 (`Organization`) 可同時提供多個 `HealthcareService`；  
> 若服務必須在特定地點或設備完成，可透過 `Eligibility`、`Location`、`characteristic` 等欄位標示限制。

下例示範「健檢套餐 A」的 **完整 R6 結構**。

```json
{
  "resourceType": "HealthcareService",
  "id": "health-check-A",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2025-05-25T10:00:00+08:00",
    "profile": [
      "http://example.org/fhir/StructureDefinition/tw-health-check-service"
    ],
    "tag": [
      {
        "system": "http://example.org/fhir/CodeSystem/service-classification",
        "code": "premium",
        "display": "高階健檢"
      }
    ]
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/health-service-id",
      "value": "HC-SVC-A"
    }
  ],
  "active": true,
  "providedBy": {
    "reference": "Organization/health-check-center",
    "display": "台灣健康檢查中心"
  },
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "32",
          "display": "Preventive Care"
        }
      ],
      "text": "預防保健"
    }
  ],
  "type": [
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
  "name": "健檢套餐 A",
  "comment": "包含血液檢查、胸部 X 光、身體組成分析，需空腹 8 小時。",
  "extraDetails": "預估檢查時間 2 小時，報到後由專人帶位。",
  "photo": {
    "contentType": "image/jpeg",
    "url": "https://example.hospital.org/images/health-check-a.jpg",
    "title": "健檢套餐 A 宣傳圖片",
    "creation": "2025-01-15"
  },
  "contact": [
    {
      "purpose": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/contact-purpose",
            "code": "APPT",
            "display": "Appointment"
          }
        ],
        "text": "預約專線"
      },
      "name": [
        {
          "text": "健檢預約中心"
        }
      ],
      "telecom": [
        {
          "system": "phone",
          "value": "02-1234-5678",
          "use": "work"
        },
        {
          "system": "email",
          "value": "appointment@health-check.tw",
          "use": "work"
        }
      ],
      "address": {
        "line": ["臺北市中正區健康路 123 號"],
        "city": "臺北市",
        "district": "中正區",
        "postalCode": "100"
      },
      "period": {
        "start": "2025-01-01",
        "end": "2025-12-31"
      }
    }
  ],
  "appointmentRequired": true,
  "availability": [
    {
      "period": {
        "start": "2025-01-01",
        "end": "2025-12-31"
      },
      "availableTime": [
        {
          "daysOfWeek": ["mon", "tue", "wed", "thu", "fri"],
          "availableStartTime": "08:00:00",
          "availableEndTime": "17:00:00"
        }
      ],
      "notAvailableTime": [
        {
          "description": "國定假日休診"
        }
      ]
    }
  ],
  "eligibility": [
    {
      "code": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/eligibility",
            "code": "self-pay",
            "display": "Self pay only"
          }
        ],
        "text": "自費專案"
      },
      "comment": "員工健檢請改用企業方案 HC-SVC-CORP"
    }
  ],
  "program": [
    {
      "text": "年度健康檢查優惠專案"
    }
  ],
  "location": [
    {
      "reference": "Location/health-check-zone",
      "display": "健檢區 A"
    }
  ]
}
```

```
Organization ─▶ HealthcareService ─▶ Schedule ─▶ Slot ─▶ Appointment
                            ▲                         ▲         ▲
                         Location                  Device     Patient
```

| 應用情境          | 解法                                                           |
| ------------- | ------------------------------------------------------------ |
| 分類健檢套餐（A/B/C） | 各建一筆 `HealthcareService`，並在 `Appointment.serviceType` 指向對應套餐 |
| 限定地點／設備       | 在 `HealthcareService.location`（或 `characteristic`）列出可用檢查區、機器 |
| 企業員工專案        | 另建專屬 `HealthcareService`，並在 `eligibility` 標示「企業合約」           |
| 線上預約強制選時段     | 設 `appointmentRequired = true`，前端必須鎖定 `Slot` 後才能送單           |
| 宣傳優惠          | 用 `program[]`、`extraDetails` 填寫活動名稱與說明，前端可直接顯示               |
| 多種聯絡方式        | 使用 `contact[]` 陣列，每個項目可設定不同用途（預約、查詢、緊急）                      |
| 服務識別圖片        | 透過 `photo` 欄位上傳服務圖片，支援 URL 或 base64 內嵌資料                     |

| FHIR 資源             | 角色 / 功能                 |
| ------------------- | ----------------------- |
| `HealthcareService` | 對外提供的健檢 / 門診服務內容        |
| `Organization`      | 提供服務的醫院 / 健檢中心          |
| [[Schedule]]        | 該服務的班表（與地點 / 設備 / 人員綁定） |
| [[Slot]]            | 可被預約的具體時段               |
| [[Appointment]]     | 病人或公司真正下單的預約            |
| `Location`          | 實際檢查區、診間                |
| [[Device]]          | 檢查儀器（CT、X 光…）           |
| `Patient`           | 接受服務的個案                 |

> **實作小提醒**
>
> * 若服務價目、檢查項目清單很長，可把明細放進 `PlanDefinition` 或附件 (Attachment) 並在 `HealthcareService` 透過 `[[Endpoint]]` 或 extension 連結。
> * 線上掛號流程：先列出 `HealthcareService` → 使用者選套餐 → 系統搜尋符合套餐之 `[[Slot]]` → 產生 `[[Appointment]]`。
> * 若不同地點提供相同套餐，可共用一筆 `HealthcareService` 並在 `location[]` 列多個點，或各自建檔視維護需求決定。
> * `availability.period` 可分季或全年，排程器再依此產生對應 [[Schedule]]/[[Slot]]。
> * 假日、機器歲修可用 `notAvailableTime` 設定文字或期間。
> * 多地點共用套餐：可共用一筆 HealthcareService，在 location[] 列出所有檢查區；或各地獨立建檔以便分開報表。
> * R5/R6 版本中，`contact` 使用 ExtendedContactDetail 型別，提供更豐富的聯絡資訊結構。
> * `meta` 欄位應包含完整的版本控制資訊，包括 versionId、lastUpdated 等。
> * `photo` 欄位使用 Attachment 型別，需要至少包含 contentType 和 url 或 data。

[FHIR文件參考](https://build.fhir.org/healthcareservice.html)