---

fhir_reference: https://build.fhir.org/device.html

---

在 FHIR 中，`Device` 是指任何可用來診斷、治療、監測或支援醫療行為的設備、儀器或軟硬體。  
若只想描述「機型」層級，可改用 `DeviceDefinition`；本文件聚焦於「單一實體設備」。

健檢常見的 `Device`：

- X 光機、CT、MRI、超音波儀  
- 血壓計、心電圖儀、骨密儀  
- 自動抽血系統、自動尿液分析機  

## **關鍵屬性**：

- `identifier`：設備的唯一識別碼。
- `status`：設備的狀態 (例如 `active`、`inactive`、`entered-in-error`)。例如，設備維修時可以將 `Device.status` 改為 `inactive`。
- `location`：指向設備所在的**地點 (Location)** 資源。
- `manufacturer`、`modelNumber`、`serialNumber`：製造商、型號和序號.
- `type`：表示設備的種類或類型，使用 `CodeableConcept`。

下方範例示範一台 X 光機（FHIR v6 結構）。

```json
{
  "resourceType": "Device",
  "meta": {
    "profile": [
      "http://example.org/fhir/StructureDefinition/tw-xray-device"
    ]
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/device-id",
      "value": "XRAY-001"
    }
  ],
  "id": "xray-001",
  "status": "active",
  "manufacturer": "GE Healthcare",
  "modelNumber": "Digital 8000",
  "serialNumber": "XRAY-202506-001",
  "name": [
    {
      "value": "X-Ray Digital 8000",
      "type": {
        "coding": [
          {
            "system": "http://hl7.org/fhir/device-nametype",
            "code": "registered-name",
            "display": "Registered device name"
          }
        ]
      }
    }
  ],
  "type": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "70617000",
          "display": "X-ray machine"
        }
      ],
      "text": "X 光機"
    }
  ],
  "location": {
    "reference": "Location/xray-room-1",
    "display": "X 光室 1"
  },
  "safety": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/device-safety",
          "code": "radiation-hazard",
          "display": "Radiation hazard"
        }
      ]
    }
  ],
  "note": [
    {
      "text": "需每半年校正；維修時同步關閉相關 Schedule。"
    }
  ]
}
```

以下提供一個最小內容：

```


```

這樣一個設備即可與排程連結（`Schedule.actor → Device/xray-001`），也能與空間（`Location`）連結，之後預約時會牽涉到 `Slot` 與 `Appointment`。

```
Device ─▶ Schedule ─▶ Slot ─▶ Appointment
          ▲             ▲         ▲
        Location     Device     Patient
```

| 應用情境        | 方法                                                          |
| ----------- | ----------------------------------------------------------- |
| 設備維修時不開放預約  | 將 `Device.status` 改 `inactive`，或停用對應 `Schedule`／停止產生 `Slot` |
| 不同健檢項目需不同設備 | 在預約系統中將健檢項目對應至一或多個 `Device`                                 |
| 同型號多台機器     | 分別建立多個 `Device`（如 `ct-001`、`ct-002`），各自排 `Schedule`         |
| 統計設備使用率     | 透過 `Appointment` 與 `Slot` 與 `Device` 的關聯進行統計                |
| 設備與病患關聯     | 使用 `DeviceAssociation` 記錄設備與病患的關聯（如植入物、穿戴式裝置）           |

| FHIR 資源         | 說明                          |
| --------------- | --------------------------- |
| `Device`        | 儀器設備定義（X 光、CT 等）            |
| [[Schedule]]    | 設備的運作時段（如週一至週五 08:00–17:00） |
| [[Slot]]        | 可預約的時間段（例如每 30 分鐘）          |
| [[Appointment]] | 病患實際預約使用設備的紀錄               |
| `Location`      | 設備所在的實體位置（如檢查室）             |

> **補充注意**
>
> * 若同一時段允許多名病患使用（如教學演示），可在同一時間產生多個 [[Slot]] 或使用擴充欄位紀錄數量。
> * 需追蹤病患植入／外接裝置時，使用 `DeviceAssociation` 資源（Device.patient 和 Device.owner 已被移除）。
> * Device.statusReason 已被移除，狀態原因改用 DeviceAssociation 記錄。
> * Device.url 和 Device.distinctIdentifier 已被移除。
> * Device.version 已被移除，改用 Device.deviceVersion 結構。
> * Device.specialization 已改名為 Device.conformsTo。
> * 新增 Device.availabilityStatus 記錄設備可用性狀態。
> * 新增 Device.biologicalSourceEvent 記錄生物來源事件。
> * 新增 Device.category 用於設備分類。
> * 新增 Device.additive 記錄容器設備的添加物質。

[FHIR文件參考](https://build.fhir.org/device.html)