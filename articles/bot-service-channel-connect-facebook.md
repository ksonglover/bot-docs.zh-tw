---
title: 將 Bot 連線到 Facebook Messenger | Microsoft Docs
description: 了解如何設定 Bot 與 Facebook Messenger 的連線。
keywords: Facebook Messenger, Bot 通道, Facebook 應用程式, 應用程式識別碼, 應用程式祕密, Facebook Bot, 認證
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/16/2018
ms.openlocfilehash: 60e39bb652ab5b7ffeeb5ba53bdf4c82f936553e
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707864"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>將 Bot 連線到 Facebook Messenger

若要深入了解如何針對 Facebook Messenger 進行開發，請參閱 [Messenger 平台文件](https://developers.facebook.com/docs/messenger-platform)。 您可以檢閱 Facebook 的[啟動前指導方針](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[快速入門](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)和[設定指南](https://developers.facebook.com/docs/messenger-platform/guides/setup)。

若要設定 Bot 使用 Facebook Messenger 進行通訊，請在 Facebook 頁面上啟用 Facebook Messenger，然後將 Bot 連線到應用程式。

> [!NOTE]
> Facebook UI 可能會因為您所使用的版本而略有不同。

## <a name="copy-the-page-id"></a>複製頁面識別碼

已透過 Facebook 頁面存取 Bot。 請[建立新的 Facebook 頁面](https://www.facebook.com/bookmarks/pages)或移至現有的頁面。

* 請開啟 Facebook 頁面的 [關於] 頁面，然後複製並儲存 [頁面識別碼]。

## <a name="create-a-facebook-app"></a>建立 Facebook 應用程式

請在頁面上[建立新的 Facebook 應用程式](https://developers.facebook.com/quickstarts/?platform=web)，然後為該應用程式產生應用程式識別碼和應用程式祕密。

![建立應用程式識別碼](~/media/channels/FB-CreateAppId.png)

* 複製並儲存 [應用程式識別碼]和 [應用程式祕密]。

![儲存應用程式識別碼和祕密](~/media/channels/FB-get-appid.png)

將 [允許 API 存取應用程式設定] 滑桿設為 [是]。

![應用程式設定](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>啟用 Messenger


在新的 Facebook 應用程式中啟用 Facebook Messenger。

* 在應用程式的 [產品設定] 頁面上，按一下 [開始使用]，然後再次按一下 [開始使用]。


![啟用 Messenger](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>產生頁面存取權杖

在 Messenger 區段的 [權杖產生] 面板中，選取目標頁面。 [頁面存取權杖] 隨即產生。

* 複製並儲存 [頁面存取權杖]。

![產生權杖](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>啟用 Webhook

按一下 [設定 Webhook]，將訊息事件從 Facebook Messenger 轉送到 Bot。

![啟用 Webhook](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>提供 Webhook 回呼 URL 並確認權杖

在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，按一下 [通道] 索引標籤，然後按一下 [Facebook Messenger]。

* 從入口網站複製 [回呼 URL] 和 [確認權杖] 值。

![複製值](~/media/channels/fb-callbackVerify.png)

1. 返回 Facebook Messenger 並且貼上 [回呼 URL] 和 [確認權杖] 值。

2. 在 [訂用帳戶欄位] 底下，選取 [message\_deliveries]、[messages]、[messaging\_options] 及 [messaging\_postbacks]。

3. 按一下 [確認並儲存]。

![設定 Webhook](~/media/channels/FB-webhookConfig.png)

4. 將 Webhook 訂閱至 Facebook 頁面。

![訂閱 Webhook](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>提供 Facebook 認證

在 Azure 入口網站中，貼上先前從 Facebook Messenger 複製的 [Facebook 應用程式識別碼]、[Facebook 應用程式祕密]、[頁面識別碼] 及 [頁面存取權杖] 值。 您可以新增額外的頁面識別碼和存取權杖，以便在多個 Facebook 頁面上使用相同的 Bot。

![輸入認證](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>提交以供審查

Facebook 需要它基本應用程式設定頁面上的 [隱私權原則 URL] 和 [服務條款 URL]。 [管理辦法](https://aka.ms/bf-conduct)頁面包含第三方資源連結，可以協助建立隱私權原則。 [使用條款](https://aka.ms/bf-terms)頁面包含範例條款，可以協助建立適當的「服務條款」文件。

Bot 完成之後，Facebook 對於發佈到 Messenger 的應用程式有自己的[審查程序](https://developers.facebook.com/docs/messenger-platform/app-review)。 系統會測試 Bot 以確保它符合 Facebook 的[平台原則](https://developers.facebook.com/docs/messenger-platform/policy-overview)規範。

## <a name="make-the-app-public-and-publish-the-page"></a>將應用程式設為公用並發佈至頁面

> [!NOTE]
> 在應用程式發佈之前，它都處於[開發模式](https://developers.facebook.com/docs/apps/managing-development-cycle)。 外掛程式和 API 功能僅適用於系統管理員、開發人員和測試人員。

審查成功之後，請在 [應用程式審查] 底下的 [應用程式儀表板] 中，將應用程式設為 [公用]。
確保與這個 Bot 相關聯的 Facebook 頁面已發佈。 狀態會顯示在頁面設定中。

> [!NOTE]
> 您也可以使用 Facebook Workplace。 若要啟用，請為您的 Workplace 建立[自訂整合](https://developers.facebook.com/docs/workplace/custom-integrations-new)，並使用其應用程式識別碼、應用程式祕密和存取權杖。 請使用其 [關於] 頁面上整合名稱後面的數字，而不是傳統的 pageID。 Webhook 可透過 Azure 中顯示的認證來連線。
