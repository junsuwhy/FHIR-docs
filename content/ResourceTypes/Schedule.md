---

fhir_reference: https://build.fhir.org/schedule.html

---

# Schedule 資源

在 FHIR 中，`Schedule` 是用來表示某個「資源（人或物）」在什麼時段內**有開放提供服務**的容器。它定義了時間窗口（period），在這個窗口內會有具體的時段（Slot）可供預約。

## 主要概念

`Schedule` 是「可預約時間段的總體範圍」，不細分到每個具體時段。系統會基於 Schedule 衍生多個 `Slot` 來對應具體可預約格。

這個「資源」（actor）可以是：
- 醫師（`Practitioner`）
- 醫療角色（`PractitionerRole`）  
- 檢查儀器（[[Device]]）
- 特定地點（`Location`）
- 醫療服務（[[HealthcareService]]）
- 照護團隊（`CareTeam`）
- 病患（`Patient`）- 用於病患可提供服務的情況
- 相關人員（`RelatedPerson`）

## 重要欄位說明

### serviceType (R5/R6 更新)
在 FHIR R5/R6 中，`serviceType` 改為 `CodeableReference` 型別，可以：
- 使用 CodeableConcept 指定服務類型代碼
- 直接參考 [[HealthcareService]] 資源，藉 HealthcareService 的 #schedule 欄位

### actor (必填)
一個或多個提供服務的資源參考。當單一資源可在不同地點提供不同服務時，應考慮將 actor 設為組合（如同時包含 Practitioner 和 Location）。

### planningHorizon
定義此 Schedule 涵蓋的時間範圍。系統應在此範圍內建立相應的 [[Slot]]。

## **關鍵屬性**

- `actor`：**必需**欄位，指向一個或多個提供服務的資源，這些資源可以是 `Patient`、`Practitioner` (醫師)、`Device`、`HealthcareService` 或 `Location`。這表示一個 `Schedule` 可以是為多個相關聯的資源（例如，醫師和診間）定義的。
- `planningHorizon`：一個 `Period` 資料型別，定義排班表涵蓋的時間段。建議設定為 1-3 個月，系統應自動延展。
- `active`：指示排班表是否在使用中。維修或停機時可設為 `false`。
- `serviceCategory`、`serviceType`、`specialty`：與 `HealthcareService` 類似，描述排班表提供的服務類別、類型和專業。


## 完整範例：CT 掃描儀排程

```json
{
  "id": "e7b8c9a2-4f3d-4e1a-9b7a-2f8e6d3c5a1b",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2025-05-26T10:00:00+08:00",
    "profile": [
      "http://example.org/fhir/StructureDefinition/tw-ct-schedule"
    ]
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/schedule-id",
      "value": "ct-A-day-shift-2025-Q3"
    }
  ],
  "active": true,
  "serviceCategory": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "30",
          "display": "Diagnostic procedures"
        }
      ],
      "text": "診斷程序"
    }
  ],
  "serviceType": [
    {
      "concept": {
        "coding": [
          {
            "system": "http://snomed.info/sct",
            "code": "363680008",
            "display": "Radiographic imaging procedure"
          }
        ],
        "text": "CT 掃描"
      },
      "reference": {
        "reference": "HealthcareService/ct-scanning-service",
        "display": "CT 掃描服務"
      }
    }
  ],
  "specialty": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "394914008",
          "display": "Radiology - speciality"
        }
      ],
      "text": "放射科"
    }
  ],
  "name": "CT-A 日班排程",
  "actor": [
    {
      "reference": "Device/ct-001",
      "display": "CT 掃描儀 A"
    },
    {
      "reference": "Location/radiology-room-3",
      "display": "放射科第三檢查室"
    }
  ],
  "planningHorizon": {
    "start": "2025-06-01T00:00:00+08:00",
    "end": "2025-08-31T23:59:59+08:00"
  },
  "comment": "CT 機台 A 每週一至週五 08:00–17:00 開放使用。系統每日自動產生 30 分鐘時段的 Slot。國定假日暫停服務。"
}
```

## 多資源排程範例：CT 掃描儀
```json
{
  "id": "b2c3d4e5-f6a7-8b9c-d0e1-f2g3h4i5j6k7",
  "identifier": [
    {
      "system": "http://example.org/schedule-ids",
      "value": "schedule-ct-001-2024"
    }
  ],
  "active": true,
  "serviceCategory": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "imaging",
          "display": "影像醫學"
        }
      ]
    }
  ],
  "serviceType": [
    {
      "concept": {
        "coding": [
          {
            "system": "http://snomed.info/sct",
            "code": "78783000",
            "display": "Computed tomography (CT) scan"
          }
        ]
      }
    }
  ],
  "specialty": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/specialty",
          "code": "radiology",
          "display": "放射科"
        }
      ]
    }
  ],
  "name": "CT 掃描儀時程 - 2024年下半年",
  "actor": [
    {
      "reference": "Device/ct-scanner-001",
      "display": "Revolution CT - 影像醫學科"
    },
    {
      "reference": "Location/imaging-department",
      "display": "影像醫學科"
    }
  ],
  "planningHorizon": {
    "start": "2024-07-01T08:00:00+08:00",
    "end": "2024-12-31T17:00:00+08:00"
  },
  "comment": "此為影像醫學科CT掃描儀的排程，涵蓋2024年7月至12月。"
}

## 多資源排程範例：醫師門診

```json
{
  "id": "e7b8c9a2-4f3d-4e1a-9b7a-2f8e6d3c5a1b",
  "meta": {
    "versionId": "2",
    "lastUpdated": "2025-05-26T11:00:00+08:00"
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/schedule-id",
      "value": "cardio-clinic-chen-2025-Q3"
    }
  ],
  "active": true,
  "serviceCategory": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "17",
          "display": "General Practice"
        }
      ],
      "text": "門診服務"
    }
  ],
  "serviceType": [
    {
      "reference": {
        "reference": "HealthcareService/cardiology-consultation",
        "display": "心臟科門診"
      }
    }
  ],
  "specialty": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "394579002",
          "display": "Cardiology"
        }
      ],
      "text": "心臟內科"
    }
  ],
  "name": "陳醫師心臟科門診",
  "actor": [
    {
      "reference": "Practitioner/dr-chen",
      "display": "陳大明醫師"
    },
    {
      "reference": "PractitionerRole/dr-chen-cardiologist",
      "display": "陳醫師-心臟科主治醫師"
    },
    {
      "reference": "Location/clinic-room-201",
      "display": "門診二樓 201 診間"
    }
  ],
  "planningHorizon": {
    "start": "2025-06-01",
    "end": "2025-08-31"
  },
  "comment": "每週二、四上午 09:00-12:00，每個病人預設 15 分鐘看診時間"
}
```

## 資源關係圖

```
                    ┌─────────────────┐
                    │ HealthcareService│
                    └────────┬────────┘
                             │ serviceType
                             ▼
┌──────────┐        ┌─────────────┐         ┌──────┐         ┌─────────────┐
│ Device   │──actor─▶│  Schedule   │────────▶│ Slot │────────▶│ Appointment │
└──────────┘        └─────────────┘         └──────┘         └─────────────┘
┌──────────┐              ▲                      ▲                    ▲
│ Location │──actor───────┘                      │                    │
└──────────┘                                     │                    │
┌──────────────┐                                 │                    │
│ Practitioner │─────────────────────────────────┘                    │
└──────────────┘                                                      │
┌──────────┐                                                          │
│ Patient  │──────────────────────────────────────────────────────────┘
└──────────┘
```

## 實作要點

| 元件              | 角色      | 功能說明                                                   |
| --------------- | ------- | ------------------------------------------------------ |
| `Schedule`      | 服務時間容器  | 定義資源的整體可用時間範圍，不含具體時段細節                                 |
| [[Slot]]        | 具體可預約時段 | 基於 Schedule 產生的單一時段（如每 30 分鐘）                          |
| [[Appointment]] | 實際預約    | 病患預約特定 [[Slot]] 的結果                                    |
| #actor          | 提供服務的資源 | 可以是設備、地點、人員等，支援多個 actor 組合                             |
| `serviceType`   | 服務類型    | R5/R6 支援 CodeableReference，可直接參考 [[HealthcareService]] |

## 最佳實踐建議

1. **識別碼管理**
   - 使用 `identifier` 進行跨系統同步，避免只依賴內部 `id`
   - 為每個排程期間（如季度）建立唯一識別碼

2. **狀態管理**
   - 維修或停機時將 `active` 設為 false，保留歷史記錄
   - 使用 `comment` 說明特殊情況（如維修原因）

3. **多資源排程**
   - 當服務需要多個資源時（如醫師＋診間＋設備），在 `actor` 中列出所有必要資源
   - 預約時需檢查所有相關 Schedule 的可用性

4. **時間管理**
   - `planningHorizon` 建議設定為 1-3 個月
   - 系統應自動延展排程，定期產生新的 [[Slot]]
   - 使用 Extension 記錄排程規則（如每週重複模式）

5. **服務分類**
   - 善用 `serviceCategory` 和 `serviceType` 協助前端篩選
   - `name` 應使用使用者友善的顯示名稱

6. **特殊情況處理**
   - 同時段多人預約：建立多個 [[Slot]] 或使用 Extension 擴充容量
   - 臨時調整：更新 Schedule 並重新產生受影響的 [[Slot]]
   - 假日處理：在 [[Slot]] 產生邏輯中排除，或產生後設為 busy

[FHIR文件參考](https://build.fhir.org/schedule.html)