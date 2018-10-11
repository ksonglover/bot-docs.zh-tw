---
title: 使用 Bot Framework 模擬器對 Bot 進行測試和偵錯 | Microsoft Docs
description: 了解如何使用 Bot Framework 模擬器桌面應用程式對 Bot 進行檢查、測試和偵錯。
keywords: 文字記錄, msbot 工具, 語言服務, 語音辨識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/20/2018
ms.openlocfilehash: 9fb42795d789f89d0bdc74d348d50dbc0ccaa7cb
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389521"
---
# <a name="debug-with-the-emulator"></a>使用模擬器進行偵錯

Bot Framework 模擬器是一項桌面應用程式，可讓 Bot 開發人員在本機或從遠端對其 Bot 進行測試和偵錯。 使用此模擬器時，您能與您的 Bot 聊天，並檢查您的 Bot 所傳送及接收的訊息。 模擬器會顯示在 Web 聊天 UI 中出現的相同訊息，並在您與 Bot 交換訊息時記錄 JSON 要求和回應。 將您的 Bot 部署至雲端之前，請先在本機使用模擬器加以測試。 即使您尚未使用 Azure Bot Service [建好](./bot-service-quickstart.md) Bot 或將其設定為在任何通道上執行，您仍可使用模擬器來測試 Bot。

## <a name="prerequisites"></a>必要條件
- 安裝[模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases)
- 安裝 [ngrok][ngrokDownload]通道軟體

## <a name="connect-to-a-bot-running-on-localhost"></a>連線至執行於本機主機的 Bot

若要連線至執行於本機主機的 Bot，請按一下 [開啟 Bot]，並選取 .bot 檔案。 

![模擬器 UI](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用偵測器檢視詳細訊息活動

傳送訊息給您的 Bot，而 Bot 應該會有所回應。 您可以按一下對話視窗內的訊息泡泡，並使用視窗右側的**偵測器**功能檢查原始 JSON 活動。 訊息泡泡經選取後會變成黃色，且活動 JSON 物件將會顯示在聊天視窗左側。 JSON 資訊包含主要中繼資料，包括 channelID、活動類型、交談識別碼、文字訊息，端點 URL 等。您可以檢查使用者所傳送的活動，以及 Bot 所回應的活動。 

![模擬器訊息活動](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>儲存及載入與 Bot 交談的文字記錄

模擬器中的活動可以儲存為文字記錄。 從開啟的即時聊天視窗中，選取 [另存文字記錄]，以儲存文字記錄檔案。 您可以隨時使用 [重新開始] 按鈕來清除對話，並重新連線至 Bot。  

![模擬器儲存文字記錄](media/emulator-v4/emulator-live-chat.png)

若要載入文字記錄，只需選取 [檔案] > [開啟文字記錄檔案]，再選取文字記錄即可。 新的 [文字記錄] 視窗隨即開啟，並將訊息活動轉譯為輸出視窗。 

![模擬器載入文字記錄](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>新增服務 

您可以直接從模擬器輕鬆地將 LUIS 應用程式、QnA 知識庫或分派模型新增到 .bot 檔案。 在 .bot 檔案載入後，請選取模擬器視窗最左側的 [服務] 按鈕。 您會在 [服務] 功能表下方看到用來新增 LUIS、QnA Maker 和分派的選項。 

若要新增服務應用程式，只需按一下 [+] 按鈕，然後選取您想要新增的服務。 系統會提示您登入 Azure 入口網站，以將服務新增至 Bot 檔案，並將服務連線到您的 Bot 應用程式。 

![LUIS 連線](media/emulator-v4/emulator-connect-luis-btn.png)

在任一服務連線時，您都可以返回即時聊天視窗，並確認您的服務已連線並運作中。 

![QnA 已連線](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>檢查服務

透過新的 v4 模擬器，您也可以檢查 LUIS 和 QnA 的 JSON 回應。 使用已連接語言服務的 Bot 時，您可以在右下方選取 [記錄] 視窗中的 [追蹤]。 這項新工具也提供直接從模擬器更新語言服務的功能。 

![LUIS 偵測器](media/emulator-v4/emulator-luis-inspector.png)

在已連接 LUIS 服務的情況下，您會發現追蹤連結指定了 **Luis 追蹤**。 加以選取後，您會看到 LUIS 服務的原始回應，其中包含意圖、實體及其指定的分數。 您也可以選擇重新指派使用者語句的意圖。 

![QnA 偵測器](media/emulator-v4/emulator-qna-inspector.png)

在已連接 QnA 服務的情況下，記錄將會顯示 **QnA 追蹤**，若加以選取，您將可預覽與該活動相關聯的問答組和信賴分數。 在此處，您可以新增答案的替代問題用語。

## <a name="configure-ngrok"></a>設定 ngrok

如果您使用 Windows，並且在防火牆或其他網路界限以外執行 Bot Framework 模擬器，而想要連線至遠端裝載的 Bot，您必須安裝並設定 **ngrok** 通道軟體。 Bot Framework 模擬器可與 ngrok 通道軟體 (由 [inconshreveable][inconshreveable] 所開發) 緊密整合，並且可在需要時自動加以啟動。

開啟 [模擬器設定] 對話方塊、輸入 ngrok 的路徑、選取是否略過 ngrok 而使用本機位址，然後按一下 [儲存]。

![ngrok 路徑](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>其他資源

Bot Framework 模擬器是開放原始碼。 您可以[參與][EmulatorGithubContribute]開發，並[提交 Bug 和建議][EmulatorGithubBugs]。



[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
