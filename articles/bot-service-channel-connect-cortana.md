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
ms.openlocfilehash: d8239f4139d27fd12507fed2cae45c67849ffe74
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326356"
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

![預設設定](~/media/channels/cortana-defaultsettings.png)

## <a name="general-bot-information"></a>一般 Bot 資訊

在**透過連線服務區段管理使用者身分識別**下方按下選項即可啟用。 填寫表單。

標有星號 (*) 的欄位為必填項目。 Bot 必須先發佈至 Azure，才能連線至 Cortana。

![管理使用者身分識別，第 1 部分](~/media/channels/cortana-manageidentity-1.png)
![管理使用者身分識別，第 2 部分](~/media/channels/cortana-manageidentity-2.png)

### <a name="when-should-cortana-prompt-for-a-user-to-sign-in"></a>Cortana 何時應該提示使用者登入

如果您希望 Cortana 讓使用者在叫用您的技能時登入，請選取 [在引動過程登入]。

如果您使用 Bot Service 登入卡片讓使用者登入，請選取 [在必要時登入]。 一般而言，只有在使用者使用需要驗證的功能時，才能使用此選項登入。 當您的技能傳送包含登入卡附件的訊息時，Cortana 會忽略登入卡，並使用連線帳戶設定執行授權流程。

### <a name="account-name"></a>帳戶名稱

您想要在使用者登入您的技能時顯示的技能名稱。

### <a name="client-id-for-third-party-services"></a>第三方服務的用戶端識別碼

您的 Bot 應用程式識別碼。 當您註冊 Bot 時，就會收到識別碼。

### <a name="space-separated-list-of-scopes"></a>範圍的空格分隔清單

指定服務所需範圍 (請參閱服務的文件)。

### <a name="authorization-url"></a>授權 URL

設定為 `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`。

### <a name="token-options"></a>權杖選項

選取 [POST] 。

### <a name="grant-type"></a>授與類型

選取 [授權碼] 以使用程式碼授與流程，或選取 [隱含] 以使用隱含流程。

### <a name="token-url"></a>權杖 URL

將 [授權碼] 授與類型設定為 `https://login.microsoftonline.com/common/oauth2/v2.0/token`。

### <a name="client-secretpassword-for-third-party-services"></a>第三方服務的用戶端密碼

Bot 的密碼。 當您註冊 Bot 時，就會收到密碼。

### <a name="client-authentication-scheme"></a>用戶端驗證配置

選取 [HTTP 基本]。

### <a name="internet-access-required-to-authenticate-users"></a>驗證使用者所需的網際網路存取權

讓此屬性保持未核取狀態。

### <a name="request-user-profile-data-optional"></a>要求使用者設定檔資料 (選用)

Cortana 會提供許多不同類型的使用者設定檔資訊，供您存取並為使用者自訂 Bot。 例如，如果技能有使用者名稱和位置的存取權，則可以為技能自訂回覆，例如「嗨，Kamran，祝你在華盛頓州倍有福有愉快的一天」。

按一下 [新增使用者設定檔要求]，然後從下拉式清單中選取您想查看的使用者設定檔資訊。 新增用來從 Bot 程式碼存取這項資訊的易記名稱。

### <a name="deploy-on-cortana"></a>在 Cortana 上部署

填妥 Cortana 技能的註冊表單後，請按一下 [在 Cortana 上部署] 以完成連線。 這麼做會將您重新導向回您 Bot 的通道刀鋒視窗中，Bot 現應已連線至 Cortana。

此時您的 Bot 已經部署為您帳戶的 Cortana 技能。

## <a name="next-steps"></a>後續步驟

* [Cortana 技能套件](https://aka.ms/CortanaSkillsDocs)
* [啟用偵錯](bot-service-debug-cortana-skill.md)
* [發佈 Cortana 技能][publish]

[invocation]: https://docs.microsoft.com/en-us/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/en-us/cortana/skills/publish-skill
[connected]: https://aka.ms/CortanaSkillsBotConnectedAccount
[CortanaEntity]: https://aka.ms/lgvcto
