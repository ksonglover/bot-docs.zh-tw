---
title: 建立對話設計工具 Bot | Microsoft Docs
description: 了解如何使用對話設計工具來建立新的 Bot。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 1b33d08e56bf8ae473b28cf1cf3a26d8138ce861
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299018"
---
# <a name="create-a-new-conversation-designer-bot"></a>建立新的對話設計工具 Bot
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

本教學課程會逐步引導您建立新的對話設計工具 Bot。 

## <a name="prerequisites"></a>必要條件

- 對話設計工具需要 Azure 訂用帳戶。 您可以從<a href="https://azure.microsoft.com/en-us/" target="_blank">這裡</a>開始使用
- 如果您還沒有這麼做，請確認在建立帳戶之後至少登入 [LUIS 入口網站](https://luis.ai)一次。
- 熟悉 JavaScript 程式設計。 自訂指令碼函式以 JavaScript 撰寫。
- Microsoft Edge 或 Google Chrome

## <a name="create-a-conversation-designer-bot"></a>建立對話設計工具 Bot

若要建立對話設計工具 Bot，請遵循下列步驟：
1. 移至 https://dev.botframework.com/ 並登入。 使用您提供給 Microsoft Corporation 的電子郵件地址，來參與這個私人預覽版本。
2. 在右上方導覽面板中，按一下 [建立 Bot]。 
3. 按一下 [建立] 以「使用對話設計工具建立 Bot」。
4. 從多個[範例 Bot](conversation-designer-sample-bots.md) 中選取一個來開始。 按 [下一步] 。 如果您不確定要使用哪一個 **範例 Bot**，則只需選擇您認為最接近您要建置之 Bot 的那一個即可。 您稍後可以切換至不同的**範例 Bot**。
5. 完成所有欄位，然後按一下 [建立 Bot]；完成 Bot 佈建大約需要 2 分鐘。 

## <a name="bot-provisioning"></a>Bot 佈建

當您建立對話設計工具 Bot 時，會自動佈建下列 Azure 功能： 

1. 具有您所指定之 Bot 名稱的 Azure 資源群組
2. Azure App Service
3. Azure App Service 方案 
4. Azure 儲存體帳戶
5. Application Insights 
6. 適用於 [LUIS.ai](https://luis.ai) \(英文\) 的認知服務訂用帳戶。 LUIS 應用程式是利用 **Bot 處理** (加上隨機產生的字串) 作為應用程式名稱來建立的。
7. Microsoft 帳戶單一應用程式。 [深入了解](https://apps.dev.microsoft.com/#/appList)

## <a name="welcome-message"></a>歡迎使用訊息

一旦佈建 Bot 之後，對話設計工具將會開啟 Bot 的 [建置] 頁面。 歡迎使用訊息隨即出現，並提供可協助您開始使用的資訊。 探索這些選項或關閉訊息，並開始使用您的 Bot。 您可以在左上方導覽面板中按一下省略符號 (**...**)，然後選擇 [歡迎使用] 選項，藉以返回歡迎使用訊息。

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [儲存 Bot](conversation-designer-save-bot.md)

## <a name="additional-resources"></a>其他資源
* 了解[工作](conversation-designer-tasks.md)
* 深入了解[適用於 Node 的 Bot Builder SDK](../nodejs/index.md) 
