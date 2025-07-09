---

fhir_reference: https://build.fhir.org/endpoint.html

---

在 FHIR 中，`Endpoint` 資源用於描述可連接的技術終端點的詳細資訊，該終端點可用於傳遞或檢索各種醫療資訊。這類似於描述一種服務的「位置」或「入口」，並提供足夠的技術細節以確保可以安全地建立連接。

Endpoint 資源適用於多種情境：

- **服務目錄**：指定醫療機構、科別或設備的通訊點
- **排程系統**：查詢空閒時段和預約的端點
- **訊息傳遞**：指定傳送臨床訊息的目的地
- **資料交換**：確定資料應該傳送到哪裡（如轉診資訊）
- **影像系統**：DICOM 影像查詢或存儲位置

## 核心欄位

```json
{
  "resourceType": "Endpoint",
  "id": "example-dicom",
  "meta": {
    "profile": [
      "http://example.org/fhir/StructureDefinition/tw-dicom-endpoint"
    ]
  },
  "identifier": [
    {
      "system": "http://hospital.example.org/endpoints",
      "value": "DICOM-SERVER-001"
    }
  ],
  "status": "active",
  "connectionType": {
    "coding": [
      {
        "system": "http://terminology.hl7.org/CodeSystem/endpoint-connection-type",
        "code": "dicom-wado-rs",
        "display": "DICOM WADO-RS"
      }
    ]
  },
  "name": "醫院 DICOM 伺服器",
  "description": "醫院 PACS 系統 DICOM 影像查詢端點",
  "environmentType": [
    {
      "coding": [
        {
          "system": "http://hl7.org/fhir/endpoint-environment",
          "code": "prod",
          "display": "Production"
        }
      ]
    }
  ],
  "managingOrganization": {
    "reference": "Organization/hospital-A",
    "display": "範例醫院 A"
  },
  "contact": [
    {
      "system": "email",
      "value": "pacs-admin@example.org",
      "use": "work"
    }
  ],
  "period": {
    "start": "2025-01-01T00:00:00+08:00"
  },
  "payload": [
    {
      "type": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/endpoint-payload-type",
              "code": "any",
              "display": "Any content"
            }
          ],
          "text": "DICOM 影像"
        }
      ],
      "mimeType": ["application/dicom"]
    }
  ],
  "address": "https://pacs.hospital.example.org/wado-rs",
  "header": ["Authorization: Bearer {token}"]
}
```

## 重要欄位說明

| 欄位 | 說明 | 必須性 |
|------|------|-------|
| `status` | 端點狀態（active、suspended、off、entered-in-error） | 必須 |
| `connectionType` | 連接類型，如 hl7-fhir-rest、dicom-wado-rs、ihe-xds 等 | 必須 |
| `name` | 端點名稱，方便人類閱讀 | 選填 |
| `description` | 端點詳細描述 | 選填 |
| `environmentType` | 環境類型，如 prod（正式）、test（測試） | 選填 |
| `address` | 端點的 URL | 必須 |
| `payload` | 支援的資料格式（mime-type）與類型 | 選填 |
| `header` | 連接時需要的 HTTP 標頭 | 選填 |

## 常見用途

1. **DICOM 影像存取**：放射科影像儲存、查詢和檢索端點
2. **健檢系統整合**：健檢系統與第三方系統間的整合端點
3. **HIE 整合**：健康資訊交換系統的存取點
4. **遠距醫療**：視訊會診或遠程監控的技術端點
5. **API 管理**：機構對外開放的 FHIR API 端點描述

## 與其他資源關聯

```
Organization ──┐
               │
Location ──────┼───> Endpoint <──── HealthcareService
               │
Practitioner ──┘
```

Endpoint 資源可以連結到多種其他資源，指明它們的技術通訊方式。例如，一個 [[HealthcareService]] 可以有多個 Endpoint，分別用於不同的通訊目的（預約、轉診、結果查詢等）。
