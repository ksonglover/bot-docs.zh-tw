---
title: 測試 Cortana 技能 |Microsoft Docs
description: 了解如何藉由叫用 Cortana 技能測試 Cortana bot。
keywords: Bot Builder SDK，註冊您的機器人，cortana
author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/18
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 15711999271d1bb8e93c1ad72eb0bc4b6acb484a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999965"
---
# <a name="test-a-cortana-skill"></a>測試 Cortana 技能

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
如果您已經建置使用 Bot Builder SDK Cortana 技能，您可以藉由從 Cortana 叫用來進行測試。 下列指示會引導您完成試用 Cortana 技能所需的步驟。

## <a name="register-your-bot"></a>註冊您的機器人
如果您在 Azure 中使用 Bot Service [建立您的 bot](~/bot-service-quickstart.md)，則您的 bot 已註冊，而您可以略過此步驟。

如果您已在其他地方部署您的機器人，或如果您想要在本機測試您的機器人，您必須[註冊](bot-service-quickstart-registration.md)您的 bot 將它連接到 Cortana。 在註冊過程中，您必須提供您機器人的**傳訊端點**。 如果您選擇要在本機上測試您的機器人，您需要執行如[ngrok](http://ngrok.com)的通道軟體，以取得您本機機器人的端點。

## <a name="get-messaging-endpoint-using-ngrok"></a>使用 ngrok 取得訊息端點

如果您在本機執行 bot，可以透過測試執行通道軟體，例如[ngrok](https://ngrok.com) 來取得端點。 若要使用 ngrok 從主控台視窗型別取得端點： 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

這會設定並顯示 ngrok 轉送連結，將要求轉送到您於連接埠 3978 上執行的機器人。 轉送連結的網址應該看起來像這樣： `https://0d6c4024.ngrok.io`。  附加 `/api/messages` 至連結，以形成傳訊端點網址，格式如下： `https://0d6c4024.ngrok.io/api/messages`。 

在 bot 的[設定](~/bot-service-manage-settings.md)刀鋒視窗**設定**區段中，輸入此端點網址。

## <a name="enable-speech-recognition-priming"></a>啟用語音辨識預備
如果您的 Bot 使用 Language Understanding (LUIS) 應用程式，請確定您已為 LUIS 應用程式識別碼與您已註冊的 bot 服務建立關聯。 這可協助您的 Bot 識別 LUIS 模型中定義的口語發音。 請參閱[語音預備](~/bot-service-manage-speech-priming.md)以取得詳細資訊。

## <a name="add-the-cortana-channel"></a>新增 Cortana 通道
若要新增 Cortana 做為通道，請依照[將 Bot 連線至 Cortana ](bot-service-channel-connect-cortana.md)中所列的步驟操作。

## <a name="test-using-web-chat-control"></a>使用網路聊天控制項進行測試

若要使用 Bot 服務中的整合式網路聊天控制項測試 Bot，請按一下**以網路聊天測試**並輸入訊息，即可驗證您的 Bo 是否正常運作。

## <a name="test-using-emulator"></a>使用模擬器進行測試

若要使用[模擬器](~/bot-service-debug-emulator.md)測試 Bot，請執行下列動作：

1. 執行 Bot。
2. 開啟模擬器，並填入所需的資訊。 若要尋找 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。 
3. 按一下**連線**即可將模擬器連線至 Bot。
4. 輸入訊息，以驗證您的 Bot 是否正常運作。

## <a name="test-using-cortana"></a>使用 Cortana 進行測試
您可以對 Cortana 說出引動片語，以引動您的 Cortana 技能。 
1. 開啟 Cortana。
2. 在 Cortana 中開啟「筆記本」，然後按一下**關於我**即可查看您要為 Cortana  使用的帳戶。 請確定您是透過註冊 Bot 所使用的 Microsoft 帳戶登入。 
   ![登入 Cortana 的筆記本](~/media/cortana/cortana-notebook.png)
2. 在 Cortana 應用程式或  Windows 的「詢問我任何事 」 搜尋方塊中按一下麥克風按鈕，然後說出您的 Bot 的[引動片語][InvocationNameGuidelines]。 引動片語包含*引動名稱*，可以唯一識別要引動的技能。 比方說，如果技能的引動名稱為「北風相片」 ，適當的引動片語可以包含 「 詢問北風相片...」 或「 告訴北風相片...」。

   您可以在為 Cortana 設定時指定 Bot 的*引動名稱*。
   ![在設定 Cortana 通道時輸入引動名稱](~/media/cortana/cortana-invocation-name-callout.png)

3. 如果 Cortana 辨識出您的引動片語，您的 Bot 就會在 Cortana 的畫布中啟動。 

## <a name="troubleshoot"></a>疑難排解

如果您的 Cortana 技能無法啟動，請檢查下列各項：
* 確定您是透過在 Bot Framework 入口網站中註冊 Bot 所使用的 Microsoft 帳戶登入。
* 按一下**在網路聊天中測試**來開啟**聊天**視窗並輸入訊息，即可檢查 Bot 是否正常運作。
* 請檢查您的引動名稱是否符合[指導方針][InvocationNameGuidelines]。 如果您的引動名稱長度超過三個單字、難以發音或聽起來像其他字，Cortana 可能就難以辨識。
* 如果您的技能使用 LUIS 模型，請務必[啟用語音辨識預備](~/bot-service-manage-speech-priming.md)。

請參閱[啟用 Cortana 技能的偵錯][ Cortana-TestBestPractice]以取得其他疑難排解提示以及如何啟用 Cortana 儀表板中技能偵錯的相關資訊。 


## <a name="next-steps"></a>後續步驟

測試 Cortana 技能，並確認其按照您想要的方式運作後，您可以對一群 beta 版測試人員部署，或公開發佈。 如需詳細資訊，請參閱 [發佈 Cortana 技能][Cortana-Publish]。

## <a name="additional-resources"></a>其他資源
* [Cortana 技能套件][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill
