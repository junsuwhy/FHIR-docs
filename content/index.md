---
title: FHIR 資訊地圖
---

## Resource Type 關係說明

在FHIR中，`Appointment`、`Device`、`HealthcareService`、`Schedule`和`Slot`這些資源緊密協作，共同管理醫療服務的預約和排程。它們各自扮演不同的角色：

- **[[HealthcareService]]（醫療服務）**
    - 定義機構對外提供的**具體服務內容**，例如「標準健檢套餐A」、「高階心臟檢查」或「骨科門診」等。
    - 可以指示該服務是否需要預約（`appointmentRequired`）或轉診（`referralRequired`）。
- **[[Device]]（儀器設備）**
    - 代表實際使用的醫療或非醫療「器械」的**個別實例**，例如醫院裡「影像醫學科」中「序號ABC123XYZ」的「Revolution CT」電腦斷層掃描儀。
    - 用於追蹤單一設備的狀態、位置和製造資訊。
- **[[Schedule]]（班表）**
    - 一個容器，用來表示**特定資源（人或物）在某段時間內有開放提供服務的總體範圍**。
    - 它定義了**「規劃時程」（planning horizon）**，即機構目前接受預約的時間區間。
    - `Schedule` 透過 [[Schedule#actor (必填)|actor]] 欄位指向一個或多個提供服務的資源，這些資源可以是 `Practitioner`（醫師）、`HealthcareService`、`Location` 或 `Device` 等。
- **[[Slot]]（時段）**
    - 代表 `Schedule` 中**可被預約的單一、具體時間段**，是最小的時間粒度（例如每15、30或60分鐘一個時段）。
    - 每個 `Slot` 都**必須**參考一個 `Schedule` 資源，以指明其所屬的班表。
    - `Slot` 有一個**狀態（status）**，表示該時段是否 `free`（空閒可預約）、`busy`（已被預約）、`busy-unavailable`（不可用，如設備維護）、`busy-tentative`（暫定預約）或 `entered-in-error`（錯誤輸入）。
- **[[Appointment]]（預約）**
    - 描述**「已提出或已確認的預約行為」**，涉及患者、執行者（如醫師）、設備和地點，並設定了具體的開始/結束時間。
    - 當一個預約被建立時，它會透過 `slot` 欄位**引用一個或多個被佔用的 `Slot` 資源**。

## 運作流程範例：健檢預約

**核心互動模式**：

- **`Schedule` 定義可用時間，並產生 `Slot`**。
- **`Slot` 繼承 `Schedule` 的 `actor` 資訊**，這些 `actor` 可以是 `Device`、`HealthcareService`、`Location`、`Practitioner` 等。
- **`Appointment` 預約一個或多個 `Slot`**。
- **`Patient` 是 `Appointment` 的參與者**。

想像一個病人想要預約健康檢查，這個過程會涉及這些FHIR資源的協同運作：

1. **服務定義**：健檢中心首先會定義其提供的健檢套餐，例如「健檢套餐A」，這會在一個**[[HealthcareService]]**資源中表示。
2. **資源排程**：
    - 為了提供健檢服務，中心需要有可用的**[[Device]]**（例如CT掃描儀）和**`Location`**（例如CT檢查室）。
    - 系統會為這些特定設備和地點設定**[[Schedule]]**，表明它們在哪些大的時間範圍內是開放的，例如「週一至週五上午8點到下午5點」。
    - **[[Schedule]]**的 #actor 欄位會指向這些[[Device]]和`Location`。
3. **生成具體時段**：基於每個[[Schedule]]`，系統會**自動生成多個[[Slot]]`資源**。每個`[[Slot]]代表一個可預約的具體小時間段（例如，每30分鐘一個CT檢查時段）。這些`[[Slot]]的`status`最初通常為`free`。
4. **病人預約**：
    - 病人透過預約系統選擇「健檢套餐A」（即選擇了**[[HealthcareService]]**）。
    - 系統會查詢相關[[Schedule]]`下的`free`狀態的**`[[Slot]]**，並展示給病人可選的時段。
    - 病人選擇一個適合的時段後，系統會創建一個**[[Appointment]]**資源。
    - 這個**[[Appointment]]**會：
        - 引用病患（`Patient`）、所需的醫師（`Practitioner`）、[[Device]]（如CT機）和`Location`（如檢查室）作為`participant`（參與者）。
        - 參考已選定的[[Slot]]資源。
        - 其自身的`status`會從`proposed`（提議）或`pending`（待處理）更新為`booked`（已預約）。
5. **時段狀態更新**：一旦[[Appointment]]`被創建並確認，對應的**`[[Slot]]**資源的`status`會從`free`更新為`busy`，表示該時段已被佔用。

這個流程確保了醫療資源（服務、設備、地點、人員）的可用性能夠被精確定義、管理和預約，實現了醫療系統之間資料的互通性。

**預約狀態變更與管理**：

- **取消 (Cancellation)**：當預約被取消時，`Appointment.status` 會更新為 `cancelled`，同時可以設定 `cancellationReason`。相關的 `Slot` 狀態可以改回 `free` 以便重新預約。
- **改期 (Rescheduling)**：通常會**建立一個新的 `Appointment`** 資源，並使用 `replaces` 欄位指向被改期的原約診。原約診的 `status` 則改為 `entered-in-error`。
- **重複約診**：`Appointment` 支援 `recurrenceTemplate` 或 `previousAppointment` 欄位來追蹤重複的約診系列。
