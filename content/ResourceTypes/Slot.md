---
fhir_reference: https://build.fhir.org/slot.html
---

# FHIR 資源：Slot (時段)

本文件基於 [FHIR R5/R6 Slot 資源](https://build.fhir.org/slot.html) 規範，提供在 Hygieia 系統中的應用說明。

`Slot` 代表一個可供預約的具體時間段。它是 `Schedule` (排程) 的細分，定義了某個資源（如醫師、設備、診間）在特定時間點的可預約性。

## 1. 核心概念

- **源自排程 (Schedule)**：每個 `Slot` **必須**透過 `schedule` 欄位關聯到一個 `Schedule` 資源。此關聯確立了該時段是為哪個服務提供者 (`Schedule.actor`) 所設。
- **狀態 (Status)**：`status` 欄位是**必需**的，用以描述時段的當前狀態（例如：`free` 可用、`busy` 忙碌）。
- **預約流程**：當一個 `Appointment` (預約) 建立時，會引用一個或多個 `Slot`，並將其狀態更新為 `busy`。
- **時間範圍**：`start` 和 `end` 是**必需**欄位，定義了時段的精確起訖時間。
- **服務定義**：`Slot` 可以透過 `serviceType`, `serviceCategory`, `specialty` 等欄位，詳細說明此時段提供的服務類型，這些資訊可以繼承自 `Schedule` 或在此處覆寫。

---

## 2. 欄位定義 (Field Definitions)

### **關鍵屬性**

- `schedule`：**必需**欄位，**必須**參考一個 `Schedule` 資源，以指明其所屬的排班表。
- `status`：**必需**欄位，表示該時段的**可預約狀態**。狀態值包括：
	- `free` (空閒可預約)。
	- `busy` (已被預約)。
	- `busy-unavailable` (不可用，例如設備維護、醫師請假)。
	- `busy-tentative` (已被暫定預約，但尚未最終確認)。
	- `entered-in-error` (被錯誤地輸入)。
- `start` 和 `end`：**必需**欄位，定義時段的開始和結束時間 (`instant` 型別)。
- `overbooked`：布林值，可以標示該時段是否已**超額預約**，例如團體治療課程。

下表列出 `Slot` 資源的核心欄位。

| 欄位名稱                | FHIR 路徑                | 型別                                       | 基數     | 說明                                                                                                                   |
| ------------------- | ---------------------- | ---------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------- |
| **#schedule**       | `Slot.schedule`        | `Reference(Schedule)`                    | `1..1` | **(必填)** 關聯的排程資源。                                                                                                    |
| **status**          | `Slot.status`          | `code`                                   | `1..1` | **(必填)** 時段狀態。枚舉值：`free`, `busy`, `busy-unavailable`, `busy-tentative`, `entered-in-error`。                          |
| **start**           | `Slot.start`           | `instant`                                | `1..1` | **(必填)** 時段開始時間 (含時區)。                                                                                               |
| **end**             | `Slot.end`             | `instant`                                | `1..1` | **(必填)** 時段結束時間 (含時區)。                                                                                               |
| identifier          | `Slot.identifier`      | `Identifier[]`                           | `0..*` | 業務標識符，用於跨系統識別。                                                                                                       |
| serviceCategory     | `Slot.serviceCategory` | `CodeableConcept[]`                      | `0..*` | 服務的廣泛分類，例如「諮詢」或「診斷」。                                                                                                 |
| **serviceType**     | `Slot.serviceType`     | `CodeableReference(HealthcareService)[]` | `0..*` | 提供的具體服務類型。可為代碼 (`CodeableConcept`) 或直接引用 `HealthcareService` 資源。此欄位在 R5 中從 `CodeableConcept` 改為 `CodeableReference`。 |
| specialty           | `Slot.specialty`       | `CodeableConcept[]`                      | `0..*` | 服務涉及的專科，例如「心臟科」。                                                                                                     |
| **appointmentType** | `Slot.appointmentType` | `CodeableConcept[]`                      | `0..*` | 此時段接受的預約類型，例如「初診」或「複診」。                                                                                              |
| overbooked          | `Slot.overbooked`      | `boolean`                                | `0..1` | 標示時段是否已超額預約。                                                                                                         |
| comment             | `Slot.comment`         | `string`                                 | `0..1` | 關於此時段的額外備註。                                                                                                          |

---

## 3. 應用情境與實作考量

| 應用情境 | 建議作法 |
|---|---|
| 批次產生時段 | 根據 `Schedule` 的規則，使用排程服務定期（如每日）產生未來 N 天的 `Slot`。 |
| 團體服務/課程 | 使用單一 `Slot` 並配合擴充欄位（如 `capacity`）或業務邏輯來管理預約數量。 |
| 緊急加號 | 建立一個新的 `Slot` 或將現有 `Slot` 的 `overbooked` 設為 `true`，並在 `comment` 中說明。 |
| 設備維護/請假 | 將受影響時段的 `status` 更新為 `busy-unavailable`。 |
| 候補名單 | 當 `Slot` 狀態從 `busy` 變回 `free` 時，可觸發通知給候補名單中的使用者。 |

---

## 4. 範例

### 範例 1：CT 檢查時段

此範例展示一個標準的放射科檢查時段。

```json
{
  "resourceType": "Slot",
  "id": "slot-ct-20250715-0900",
  "status": "free",
  "schedule": {
    "reference": "Schedule/sched-ct-room1-morning"
  },
  "start": "2025-07-15T09:00:00+08:00",
  "end": "2025-07-15T09:30:00+08:00",
  "serviceType": [
    {
      "concept": {
        "coding": [
          {
            "system": "http://snomed.info/sct",
            "code": "77477000",
            "display": "Computerized axial tomography"
          }
        ]
      },
      "reference": {
        "reference": "HealthcareService/hs-ct-scanning"
      }
    }
  ],
  "specialty": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "394914008",
          "display": "Radiology"
        }
      ]
    }
  ],
  "appointmentType": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/v2-0276",
          "code": "ROUTINE"
        }
      ]
    }
  ],
  "comment": "需空腹4小時。請提前15分鐘報到。"
}
```

### 範例 2：團體諮商時段

此範例展示一個可供多人預約的團體課程時段。在我們的實作中，會使用擴充欄位 `capacity` 和 `booked_count` 來管理名額。

```json
{
  "resourceType": "Slot",
  "id": "slot-group-therapy-20250720-1400",
  "status": "free",
  "schedule": {
    "reference": "Schedule/sched-group-therapy-room"
  },
  "start": "2025-07-20T14:00:00+08:00",
  "end": "2025-07-20T15:30:00+08:00",
  "serviceCategory": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "5",
          "display": "Counseling"
        }
      ]
    }
  ],
  "appointmentType": [
    {
      "coding": [
        {
          "system": "http://example.org/appointment-types",
          "code": "GROUP"
        }
      ]
    }
  ],
  "comment": "焦慮管理團體課程。最多可容納5位參與者。"
}
```

---

## 5. 與其他資源的關係

```
[Schedule] ─────┐
     │           │ generates
     │           │
     ▼           ▼
[HealthcareService]  ┌─────────┐   ┌─────────────┐
[Practitioner]     │  Slot   │ ◀─┤ Appointment │
[Device]           └─────────┘   └─────────────┘
[Location]             ▲                 ▲
     │                 │                 │
     └─ (actor) ───────┘                 │ books
                                         │
                                     [Patient]
```
- [[Schedule]] 定義了資源的可用時間，並從中產生多個 `Slot`。
- `Slot` 繼承 [[Schedule]] 的 `actor` (服務提供者) 和服務類型資訊。
- [[Appointment]] 預約一個或多個 `Slot`，並將 `Patient` (病患) 設為參與者。

[FHIR文件參考](https://build.fhir.org/slot.html)