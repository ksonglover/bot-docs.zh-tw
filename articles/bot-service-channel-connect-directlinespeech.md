---
title: 將 Bot 連線至 Direct Line Speech
titleSuffix: Bot Service
description: 將現有 Bot Framework Bot 連線到 Direct Line Speech 通道，以進行高可靠性、低延遲語音輸入、語音輸出互動的概觀和所需步驟。
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: travisw
ms.openlocfilehash: 55bb6b63f35b2cb064229ed0a827af422ca83882
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592218"
---
# <a name="connect-a-bot-to-direct-line-speech"></a>將 Bot 連線至 Direct Line Speech

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

您可以設定 Bot，允許用戶端應用程式透過 Direct Line Speech 通道與其通訊。

建置好您的 Bot 後，透過 Direct Line Speech 上線，將可使用 [Speech SDK](https://aka.ms/speech/sdk) 與用戶端應用程式進行低延遲、高可靠性連線。 這些連線最適合用於語音輸入、語音輸出的交談式體驗。 如需有關 Direct Line Speech 以及如何建置用戶端應用程式的詳細資訊，請瀏覽[自訂語音優先虛擬助理](https://aka.ms/bots/speech/va)頁面。 

## <a name="add-the-direct-line-speech-channel"></a>新增 Direct Line Speech 通道

1. 若要新增 Direct Line Speech 通道，請先在 [Azure 入口網站](https://portal.azure.com)開啟 Bot，然後從您的資源中選取 **Bot 通道註冊**資源。 按一下 [設定] 刀鋒視窗中的 [通道]  。

    ![醒目提示位置以便選取所要連線的通道](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "選取通道")

1. 在通道選取頁面中，尋找並按一下 `Direct Line Speech` 以選擇通道。

    ![選取 Direct Line Speech 通道](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "與 Direct Line Speech 連線")

1. Direct Line Speech 通道需要認知服務資源。 您可以使用現有的資源，或依照[指示](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)建立新的認知服務資源。 

    ![選取 Direct Line Speech 通道](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-cognitivesericesaccount-selection.png "選取認知服務資源")

1. 在您檢閱使用規定後，按一下 `Save` 確認您的通道選取項目。

    ![儲存 Direct Line Speech 通道的啟用](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "儲存通道設定")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>啟用 Bot Framework 通訊協定串流擴充功能

在 Direct Line Speech 通道連線到 Bot 的情況下，您現在需要啟用 Bot Framework 通訊協定串流擴充功能支援，以獲得最佳、低延遲的互動。

1. 如果您還沒這麼做，請在 [Azure 入口網站](https://portal.azure.com)中開啟您 Bot 的刀鋒視窗。 

1. 按一下 [Bot 管理]  類別之下的 [設定]  (在 [通道]  的正下方)。 按一下 [啟用串流端點]  核取方塊。

    ![啟用串流通訊協定](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "啟用串流延伸模組支援")

1. 按一下頁面頂端的 [儲存]  。

1. 在相同刀鋒視窗的 [App Service 設定]  類別之下，按一下 [設定]  。

    ![瀏覽至 App Service 設定](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "設定 App Service")

1. 按一下 `General settings` ，然後選取可啟用 `Web socket` 支援的選項。

    ![啟用 App Service 的 WebSocket](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "啟用 Websocket")

1. 按一下設定頁面頂端的 `Save`。

1. Bot Framework 通訊協定串流擴充功能現已針對您的 Bot 啟用。 您現在已準備好更新 Bot 程式碼並[整合串流擴充功能支援](https://aka.ms/botframework/addstreamingprotocolsupport)至現有的 Bot 專案。

## <a name="adding-protocol-support-to-your-bot"></a>將通訊協定支援新增至您的 Bot

連線 Direct Line Speech 通道並啟用 Bot Framework 通訊協定串流擴充功能支援後，只要將程式碼新增至您的 Bot 來支援最佳化的通訊。 遵循[將串流擴充功能支援新增至您的 Bot](https://aka.ms/botframework/addstreamingprotocolsupport) 上的指示，以確保與 Direct Line Speech 完全相容。


