---
title: 行為優先級 (ActPriority) 值集
---


欄位為 #priority 

## 簡介
本值集定義了醫療照護行為的優先級別代碼，用於標識醫療行為或服務的緊急程度和優先級別。

## 代碼定義

| 代碼 | 顯示名稱 | 定義 |
|------|---------|------|
| A    | 盡快 (As soon as possible) | 盡快執行，優先級僅次於「緊急」(stat) |
| CR   | 回調結果 (Callback results) | 當結果可用時，執行者應盡快聯繫開立者，即使是初步結果也應回報（在HL7 2.3版本中標記為「C」） |
| EL   | 選擇性 (Elective) | 對病人有益但非生存必需 |
| EM   | 急診 (Emergency) | 需要立即處理的意外情況或緊急狀態 |
| P    | 術前 (Pre-op) | 用於指示在預定手術前需要進行的服務。訂購此類服務時，會檢查執行服務所需的時間，以避免與預定手術時間衝突 |
| PRN  | 需要時 (As needed) | 按需訂單應附帶說明何種情況下需要執行 |
| R    | 常規 (Routine) | 常規服務，在正常工作時間執行 |
| RR   | 緊急報告 (Rush reporting) | 應盡快準備並發送報告 |
| S    | 緊急 (Stat) | 最高優先級（如急診情況） |
| T    | 時間關鍵 (Timing critical) | 盡可能接近要求的時間執行（例如用於抗生素濃度監測） |
| UD   | 依指示使用 (Use as directed) | 按照處方者的指示使用藥物 |
| UR   | 緊急 (Urgent) | 需要立即行動 |
| CS   | 回調排程 (Callback for scheduling) | 用於安排回調以進行排程 |
| CSP  | 回調排程優先 (Callback placer for scheduling) | 開立者回調以進行排程 |
| CSR  | 回調接收排程 (Contact recipient for scheduling) | 聯繫接收者進行排程 |

## 擴展
此值集還包含以下值集中的代碼：
- [ActPriorityCallback](https://terminology.hl7.org/6.5.0/ValueSet-v3-ActPriorityCallback.html)
- [x_EncounterAdmissionUrgency](https://terminology.hl7.org/6.5.0/ValueSet-v3-xEncounterAdmissionUrgency.html)

## 版本資訊
- **版本**：6.5.0
- **發布日期**：2023-11-14
- **版權**：© 2020+ HL7 International - Vocabulary Work Group

## 參考資料
- [HL7 Terminology (THO) v6.5.0](https://terminology.hl7.org/6.5.0/index.html)
- [CodeSystem: v3 ActPriority](https://terminology.hl7.org/6.5.0/CodeSystem-v3-ActPriority.html)