---
title: 匯入和匯出對話設計工具 Bot | Microsoft Docs
description: 了解如何匯入和匯出對話設計工具 Bot。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: ed055cb0f75d148e6a0dd3248366851901d9a675
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300359"
---
# <a name="export-and-import-a-conversation-designer-bot"></a>匯入和匯出對話設計工具 Bot

對話設計工具中的 Bot 可以 .zip 檔案形式匯出到您的電腦。 這可讓您備份 Bot。 您稍後可以使用任何一個匯出的 .zip 檔案，將 Bot 還原至先前狀態。 

## <a name="export-a-conversation-designer-bot"></a>匯出對話設計工具 Bot

匯出可讓您將對話設計工具 Bot 的目前狀態儲存到本機電腦。 

若要匯出對話設計工具 Bot，請執行下列作業：
1. 在導覽面板的右上角，按一下省略符號 (**...**)。
2. 按一下 [匯出]。 伺服器會蒐集所需的資訊，並傳回可供您儲存 .zip 檔案的選項。
3. 將匯出的 .zip 檔案儲存到您的本機電腦。

您的 Bot 現在會以 .zip 檔案形式儲存在您的電腦上。 您稍後可以將該檔案[匯入](#import-a-conversation-designer-bot)回到 Bot，讓 Bot 還原到此狀態。

## <a name="import-a-conversation-designer-bot"></a>匯入對話設計工具 Bot

匯入作業可讓您將對話設計工具 Bot 還原至先前的狀態。 匯入作業將會以所匯入的 Bot 取代目前的 Bot。 如果您不想喪失您目前在目前 Bot 中擁有的內容，請務必匯入作業之前，先[匯出](#export-a-conversation-designer-bot)目前的 Bot。

若要匯入對話設計工具 Bot，請執行下列作業：
1. 在導覽面板的右上角，按一下省略符號 (**...**)。
2. 按一下 [匯入] 。 
3. 如果您想要將您的工作儲存在目前的 Bot 中，請選擇 [備份並匯入] 選項。 此選項會將目前的 Bot 儲存至本機電腦，然後要求您提供要匯入的 .zip 檔案的位置。 否則，請選擇 [匯入但不要備份]。

您的 Bot 現已匯入。

> [!NOTE]
> 您可以只匯入由對話設計工具所匯出的 Bot。

## <a name="import-a-luis-app-into-a-conversation-designer-bot"></a>將 LUIS 應用程式匯入對話設計工具 Bot 中

如果您有 LUIS 應用程式，而且想要將它使用於對話設計工具 Bot，您可以將 LUIS 應用程式匯入到對話設計工具 Bot 中。 就概念而言，此程序需要您匯出 LUIS 應用程式和對話設計工具 Bot，然後交換 Bot 的 **luis.model** 檔案與 **luis.json** 檔案內容。 然後，將您的變更匯入回到對話設計工具 Bot。 基本上，以 LUIS 應用程式的 LUIS 意圖取代 Bot 中的 LUIS 意圖。 因此，建議您先執行此匯入作業，再開始自訂 Bot 的 LUIS 意圖；否則，此匯入作業將會覆寫您所有的工作。

> [!NOTE]
> 如果您要在與 Bot 相關聯 [LUIS](https://luis.ai) 應用程式 (每個對話設計工具 Bot 都有對應的 LUIS 應用程式) 中進行變更，則不需要執行這些步驟。 反而，只需要進入對話設計工具 Bot，然後按一下 [[儲存](conversation-designer-save-bot.md)]。

若要將 LUIS 應用程式匯入到對話設計工具 Bot，請執行下列作業：

1. 從 [LUIS.ai](https://luis.ai)，啟您的 LUIS 應用程式，然後按一下 [設定]。
2. 選擇您想要使用的應用程式 [版本]，然後按一下 **{ }** 動作圖示。 這個動作會將 **luis.json** 檔案下載到本機電腦。 
3. 從對話設計工具，[建立新的 Bot](conversation-designer-create-bot.md#create-a-conversation-designer-bot) 或開啟現有的 Bot。
4. [匯出](#export-a-conversation-designer-bot) 您的 Bot。 這會將您的 Bot 以 .zip 檔案形式匯出到電腦。
5. 瀏覽至已匯出的 .zip 檔案並將它解壓縮。
6. 在文字編輯器中開啟 **luis.model** 檔案，並以 **luis.json** 檔案的內容取代此檔案的內容。 儲存檔案。
7. 壓縮對話設計工具 Bot 的資料夾。
8. 將 Bot [匯入](#import-a-conversation-designer-bot)回到對話設計工具 Bot。

您的 Bot 現在可以使用您剛匯入的新 LUIS 意圖。

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [工作](conversation-designer-tasks.md)
