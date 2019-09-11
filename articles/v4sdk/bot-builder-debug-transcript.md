---
title: 使用文字記錄檔進行 Bot 偵錯 | Microsoft Docs
description: 了解如何使用文字記錄檔，協助進行 Bot 偵錯。
keywords: 偵錯, 常見問題集, 文字記錄檔, 模擬器
author: DanDev33
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 20d3ff54ed7c1ece0678c4b98bea920f8becdaf3
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299373"
---
# <a name="debug-your-bot-using-transcript-files"></a>使用文字記錄檔進行 Bot 偵錯

[!INCLUDE[applies-to](../includes/applies-to.md)]

成功進行 Bot 測試和偵錯的關鍵之一，就是能夠記錄並檢查在執行 Bot 時所發生的一些情況。 本文討論 Bot 文字記錄檔的建立和使用方式，以提供一組詳細的使用者互動和 Bot 回應以供測試和偵錯。

## <a name="the-bot-transcript-file"></a>Bot 文字記錄檔
Bot 文字記錄檔是特製化的 JSON 檔案，可保留使用者與 Bot 之間的互動。 文字記錄檔不僅保留訊息內容，也會保留互動詳細資料，例如使用者識別碼、通道識別碼、通道類型、通道功能、互動時間等。這些資訊接著可用來協助尋找及解決 Bot 測試或偵錯時的問題。 

## <a name="creatingstoring-a-bot-transcript-file"></a>建立/儲存 Bot 文字記錄檔
本文說明如何使用 Microsoft 的 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator)建立 Bot 文字記錄檔。 文字記錄檔也可透過程式設計方式建立，您可以在[這裡](./bot-builder-howto-v4-storage.md#blob-transcript-storage)進一步了解該方法。 在這篇文章中，我們將使用[多回合提示 Bot](https://aka.ms/cs-multi-prompts-sample) 的 Bot Framework 範例程式碼，該程式碼會要求使用者的運輸方式、名稱和年齡，但任何可使用 Microsoft 的 Bot Framework Emulator 存取的程式碼都可用來建立文字記錄檔。

若要開始此程序，請確保您想要測試的 Bot 程式碼是在您的開發環境內執行。 啟動 Bot Framework Emulator，選取 [開啟 Bot]  按鈕，然後輸入瀏覽器中所示的 _localhost: port_ 位址，後面接著 "/api/messages"，如下圖所示。 現在按一下 [連線]  按鈕，以將模擬器連線至 Bot。

![將模擬器連線到程式碼](./media/emulator_open_bot_configuration.png)

將模擬器連線到執行中的程式碼之後，請將模擬的使用者互動傳送給 Bot，以便測試程式碼。 此範例中，我們已傳入使用者的運輸方式、名稱和年齡。 在您輸入所有想要保留的使用者互動之後，請使用 Bot Framework 模擬器來建立及儲存含有此對話的文字記錄檔。 

在 [即時聊天]  索引標籤 (如下所示) 中，選取 [儲存文字記錄]  按鈕。 

![選取儲存文字記錄](./media/emulator_transcript_save.png)

選擇文字記錄檔的位置和名稱，然後選取 [儲存] 按鈕。

![文字記錄另存為 ursula](./media/emulator_transcript_saveas_ursula.png)

您為了使用模擬器測試程式碼而輸入的所有使用者互動和 Bot 回應，現在已儲存到您可稍後重新載入的文字記錄檔中，以協助進行使用者與 Bot 之間的互動偵錯。

## <a name="retrieving-a-bot-transcript-file"></a>擷取 Bot 文字記錄檔
若要使用 Bot Framework Emulator 擷取 Bot 文字記錄檔，請依序選取模擬器左上角的 [檔案]  和 [開啟文字記錄...]  ，如下所示。 接下來，選取您想要擷取的文字記錄檔。 (從模擬器的 [資源]  區段中的 [文字記錄]  清單控制項，也可以存取文字記錄) 

在此範例中，我們會擷取名為 "ursula_user.transcript" 的文字記錄檔。 選取文字記錄檔，就會將整個保留的對話自動載入到標題為「文字記錄」  的新索引標籤。

![擷取已儲存的文字記錄](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>使用文字記錄檔進行偵錯
載入文字記錄檔後，您現在即可對在使用者與 Bot 之間擷取的互動進行偵錯。 若要這樣做，只要按一下模擬器右下方區域中顯示的 LOG  區段中所記錄的任何事件或活動。 在以下所示的範例中，我們選取了使用者傳送 "Hello" 訊息時的第一次反覆運算。 當我們執行此動作時，文字記錄檔中關於此特定互動的所有資訊都會以 JSON 格式顯示在模擬器的 INSPECTOR  視窗中。 由下而上查看其中一些值，我們會看到：
* 反覆運算類型為 message  。
* 訊息的傳送時間。
* 已傳送內含 "Yes" 的純文字。
* 訊息已傳送給 Bot。
* 使用者識別碼和資訊。
* 通道識別碼、功能和資訊。

![使用文字記錄進行偵錯](./media/emulator_transcript_debug.png)

這麼詳細的資訊可讓您追蹤使用者輸入與 Bot 回應之間的逐步互動，而在 Bot 並未以您預期的方式回應或完全不回應使用者的情況下，這就很適合用於偵錯。 擁有這些值以及導致互動失敗的步驟記錄，可讓您逐步執行程式碼、尋找 Bot 未如預期般回應的位置，以及解決這些問題。

使用文字記錄檔搭配 Bot Framework 模擬器，只是可用來協助您進行 Bot 程式碼與使用者互動測試和偵錯的許多工具之一。 若要尋找更多 Bot 測試和偵錯的方法，請參閱下面所列的其他資源。

## <a name="additional-information"></a>其他資訊

如需其他測試和偵錯資訊，請參閱：

* [Bot 測試和偵錯指導方針](./bot-builder-testing-debugging.md)
* [使用 Bot Framework 模擬器進行偵錯](../bot-service-debug-emulator.md)
* [針對一般問題進行疑難排解](../bot-service-troubleshoot-bot-configuration.md)以及該區段中的其他疑難排解文章。
* [Visual Studio 偵錯](https://docs.microsoft.com/visualstudio/debugger/index)
