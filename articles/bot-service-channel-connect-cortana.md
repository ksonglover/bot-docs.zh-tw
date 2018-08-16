---
title: 將 Bot 連線至 Cortana | Microsoft Docs
description: 了解如何設定 Bot，以透過 Cortana 介面進行存取。
keywords: cortana, Bot 通道, 設定 cortana
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 6e694ce8b54ebd2405d7496d333c2bb27eb344f1
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299174"
---
# <a name="connect-a-bot-to-cortana"></a>將 Bot 連線至 Cortana

Cortana 是具有語音功能的通道，除了文字對話之外，還可以傳送和接收語音訊息。 如要將 Bot 連線至 Cortana，應將 Bot 設計為適用於文字和語音。 Cortana *技能*是可以使用 Cortana 用戶端叫用的 Bot。 發佈 Bot 即可將 Bot 加入可用技能的清單。

如要新增 Cortana 通道，請在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，按一下**通道**刀鋒視窗，然後按一下 **Cortana**。

![新增 Cortana 通道](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>設定 Cortana

將您的 Bot 連線至 Cortana 通道時，Bot 的部分基本資訊會預先填入註冊表格。 請仔細檢閱此項資訊。 此表格包含下列欄位。

| 欄位 | 說明 |
|------|------|
| **技能圖示** | 叫用技能時，圖示就會顯示在 Cortana 畫布上。 圖示也可用於搜尋技能 (例如 Microsoft 網上商店)。 (上限為 32KB，僅限 PNG 檔)。|
| **顯示名稱** | Cortana 技能的名稱會顯示於視覺 UI 的頂端。 (上限為 30 個字元) |
| **引動名稱** | 這是使用者叫用技能時說出的名稱。 名稱不得超過三個字組，且應易於發音。 請參閱[引動名稱指導方針][invocation]，進一步了解如何選擇此名稱。|
| **說明** | Cortana 技能的說明。 此可用於搜尋技能 (例如 Microsoft 網上商店)。 |
| **簡短說明** | 關於您技術功能的簡短說明，用來描述在 Cortana 筆記本中的技能。 |

## <a name="general-bot-information"></a>一般 Bot 資訊

在**透過連線服務區段管理使用者身分識別**下方按下選項即可啟用。 填寫表單。

標有星號 (*) 的欄位為必填項目。 Bot 必須先在 Bot Framework 發佈，才能連線至 Cortana。

![提供一般資訊](~/media/channels/cortana-details.png)

### <a name="sign-in-at-invocation"></a>在引動過程登入

如果您想在使用者叫用您的技能時讓 Cortana 登入，請選取此選項。

### <a name="sign-in-when-required"></a>在必要時登入

如果您使用 Bot Framework 的登入卡來讓使用者登入，請選取此選項。 一般而言，只有在使用者使用需要驗證的功能時，才能使用此選項登入。 當您的技能傳送包含登入卡附件的訊息時，Cortana 會忽略登入卡，並使用連線帳戶設定執行授權流程。

### <a name="connected-service-icon"></a>連線服務圖示

您想要在使用者登入您的技能時顯示的圖示。

### <a name="account-name"></a>帳戶名稱

您想要在使用者登入您的技能時顯示的技能名稱。

### <a name="client-id-for-third-party-services"></a>第三方服務的用戶端識別碼

您的 Bot 應用程式識別碼。 當您註冊 Bot 時，就會收到識別碼。

### <a name="space-separated-list-of-scopes"></a>範圍的空格分隔清單

指定服務所需範圍 (請參閱服務的文件)。

### <a name="authorization-url"></a>授權 URL

設定為 `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`。

### <a name="grant-type"></a>授與類型

選取授權碼，以使用代碼授與流程。 選取要用於隱含流程的隱含。

### <a name="token-url"></a>權杖 URL

如果您要選取授權碼，請設定為`https://login.microsoftonline.com/common/oauth2/v2.0/token`。

### <a name="client-secretpassword-for-third-party-services"></a>第三方服務的用戶端密碼

Bot 的密碼。 當您註冊 Bot 時，就會收到密碼。

### <a name="client-authentication-scheme"></a>用戶端驗證配置

選取 HTTP基本。

### <a name="token-options"></a>權杖選項

設定為 POST。

### <a name="request-user-profile-data-optional"></a>要求使用者設定檔資料 (選用)

Cortana 會提供許多不同類型的使用者設定檔資訊，供您存取並為使用者自訂 Bot。 例如，如果技能有使用者名稱和位置的存取權，則可以為技能自訂回覆，例如「嗨，Kamran，祝你在華盛頓州倍有福有愉快的一天」。

按一下**新增使用者設定檔要求**連結，然後從下拉式清單中選取您想查看的使用者設定檔資訊。 新增用來從 Bot 程式碼存取這項資訊的易記名稱。

### <a name="save-skill"></a>儲存技能

填妥 Cortana 技能的註冊表單後，請按一下 [儲存] 以完成連線。 這麼做會將您重新導向回您 Bot 的通道刀鋒視窗中，Bot 現應已連線至 Cortana。

此時您的 Bot 已經自動部署為您帳戶的 Cortana 技能。

## <a name="next-steps"></a>後續步驟

* [Cortana 技能套件](https://aka.ms/CortanaSkillsDocs)
* [啟用偵錯](bot-service-debug-cortana-skill.md)
* [發佈 Cortana 技能][publish]

[invocation]: https://docs.microsoft.com/en-us/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/en-us/cortana/skills/publish-skill
[connected]: https://aka.ms/CortanaSkillsBotConnectedAccount
[CortanaEntity]: https://aka.ms/lgvcto
