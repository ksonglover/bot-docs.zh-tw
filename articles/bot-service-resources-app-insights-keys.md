---
title: Application Insights 金鑰 | Microsoft Docs
description: 了解如何取得 Application Insights 金鑰以新增遙測到 Bot。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 87cdd27a3a413ae200273090d4a79b5edbc55dc1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996905"
---
# <a name="application-insights-keys"></a>Application Insights 金鑰

Azure **Application Insights** 會在 Microsoft Azure「資源」中顯示您應用程式的相關資料。 若要新增遙測到您的 Bot，您需要 Azure 訂用帳戶與為您的 Bot 建立的 Application Insights 資源。 針對此資源，您可以取得三個金鑰來設定您的 Bot：

1. 檢測金鑰
2. 應用程式識別碼
3. API 金鑰

此主題將說明如何建立這些 Application Insights 金鑰。

> [!NOTE]
> 在建立 Bot 或在註冊程序期間，我們提供選項讓您*開啟*或*關閉* **Application Insights**。 若您已將它*開啟*，則您的 Bot 已經有所需的所有 Application Insights 金鑰。 不過，若您已將它*關閉*，則您可以依照此主題中的指示協助您手動建立這些金鑰。

## <a name="instrumentation-key"></a>檢測金鑰

若要取得檢測金鑰，請執行下列動作：
1. 從 [Azure 入口網站](http://portal.azure.com)的 [監視器] 區段下，建立新 **Application Insights** 資源 (或使用現有資源)。
![Application Insights 清單的入口網站螢幕擷取畫面](~/media/portal-app-insights-add-new.png)

2. 從 Application Insights 資源清單中，按一下您剛建立的 Application Insight 資源。

3. 按一下 [概觀] 。

4. 展開 [基本] 區塊並尋找 [檢測金鑰]。 
![概觀的入口網站螢幕擷取畫面](~/media/portal-app-insights-instrumentation-key-dropdown.png)
![檢測金鑰的入口網站螢幕擷取畫面](~/media/portal-app-insights-instrumentation-key.png)

5. 複製**檢測金鑰**並將它貼到您 Bot 設定的 [Application Insights 檢測金鑰] 欄位。

## <a name="application-id"></a>應用程式識別碼

若要取得應用程式識別碼，請執行下列動作：
1. 從您的 Application Insights 資源，按一下 [API 存取權]。

2. 複製**應用程式識別碼** 並將它貼到您 Bot 設定的 [Application Insights 應用程式識別碼] 欄位。 
![應用程式識別碼的入口網站螢幕擷取畫面](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>API 金鑰

若要取得 API 金鑰，請執行下列動作：
1. 從 Application Insights 資源，按一下 [API 存取權]。

2. 按一下 [建立 API 金鑰]。

3. 輸入簡短說明，選取 [讀取遙測]選項，並按一下 [產生金鑰] 按鈕。
![應用程式識別碼與 API 金鑰的入口網站螢幕擷取畫面](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > 複製此 **API 金鑰** 並儲存它，因為我們再也不會向您顯示此金鑰。 若遺失此金鑰，您必須重新建立。

4. 將 API 金鑰複製到您 Bot 設定的 [Application Insights API 金鑰] f欄位。

## <a name="additional-resources"></a>其他資源
如需有關如何將這些欄位連結到您 Bot 設定的詳細資訊，請參閱[啟用分析](~/bot-service-manage-analytics.md#enable-analytics)。