---
title: Azure Bot Service 中的使用者驗證 | Microsoft Docs
description: 了解 Azure Bot Service 中的使用者驗證功能。
keywords: azure bot 服務, 驗證, bot framework 權杖服務
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b4b71f7679ad6eea283437acb5a8293855219cf2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299493"
---
# <a name="user-authentication-within-a-conversation"></a>對話中的使用者驗證

若要代表使用者執行某些作業，例如檢查電子郵件、參考行事曆、查看航班狀態或下訂單，Bot 就會需要呼叫外部服務，例如 Microsoft Graph、GitHub 或公司的 REST 服務。
每個外部服務都有保護這些呼叫的方式，而保護這類呼叫的常見方式是使用_使用者權杖_來發出這些要求，使用者權杖是使用者在外部服務上的獨有身分識別 (有時稱為 JWT)。

為了保護以外部服務為目標的呼叫，Bot 必須要求使用者登入，讓其可以取得使用者的權杖來使用該服務。
許多服務支援透過 OAuth 或 OAuth2 通訊協定來擷取權杖。
Azure Bot Service 提供特製化的登入卡片和服務，可搭配 OAuth 通訊協定使用及管理權杖生命週期，而且 Bot 可以使用這些功能來取得使用者權杖。

- _OAuth 連線設定_是 Bot 組態的一部分，註冊在 Azure 的 Azure Bot Service 資源中。

    每個連線設定都包含要使用的外部服務或識別提供者資訊，以及有效的 OAuth 用戶端識別碼及祕密、要啟用的 OAuth 範圍和該外部服務或身分識別所需的任何其他連線中繼資料。

- 在 Bot 的程式碼中，OAuth 連線設定會用來協助登入使用者，並取得使用者權杖。

這些服務包含在登入工作流程中。

![驗證概觀](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>關於 Bot Framework 權杖服務

Bot Framework 權杖服務負責下列事項：

- 促使 OAuth 通訊協定搭配各種不同外部服務使用。
- 安全儲存特定 Bot、通道、對話及使用者的權杖。
- 管理權杖生命週期，包括嘗試重新整理權杖。

例如，可使用 Microsoft Graph API 檢查使用者近期電子郵件的 Bot 會需要 Azure Active Directory 使用者權杖。 在設計階段中，Bot 開發人員會向 Bot Framework 權杖服務註冊 Azure Active Directory 應用程式 (透過 Azure 入口網站)，然後為 Bot 設定 OAuth 連線設定 (名稱為 `GraphConnection`)。 使用者與 Bot 互動時，工作流程為：

1. 使用者要求 Bot：「請查看我的電子郵件」。
1. 包含此訊息的活動會從使用者端傳送到 Bot Framework 通道服務。 通道服務可確保活動內的 `userid` 欄位已設定，並將訊息傳送到 Bot。

    使用者識別碼會專屬於各通道，例如使用者的 Facebook 識別碼或其 SMS 電話號碼。

1. Bot 會收到訊息活動，並判斷其意圖是要檢查使用者的電子郵件。 Bot 會對 Bot Framework 權杖服務發出要求，詢問其是否已有用於 OAuth 連線設定 `GraphConnection` 的 UserId 權杖。
1. 由於這是此使用者第一次與 Bot 互動，Bot Framework 權杖服務還沒有此使用者的權杖，因此會將_NotFound_的結果傳回給 Bot。
1. Bot 會以 `GraphConnection` 連線名稱建立 OAuthCard，並回覆使用者，要求他們使用這張卡片登入。
1. 活動會透過 Bot Framework 通道服務傳遞，以呼叫 Bot Framework 權杖服務為此要求建立有效的 OAuth 登入 URL。 此登入 URL 會新增至 OAuthCard，而該卡片會傳回給使用者。
1. 使用者會看到藉由按一下 OAuthCard 登入按鈕來登入的訊息。
1. 當使用者按一下 [登入] 按鈕時，通道服務便會開啟網頁瀏覽器，以及呼叫外部服務來載入其登入頁面。
1. 使用者會登入此頁面來使用外部服務。 完成後，外部服務會完成 OAuth 通訊協定與 Bot Framework 權杖服務的交換，讓外部服務將使用者權杖傳送給 Bot Framework 權杖服務。 Bot Framework 權杖服務會安全地儲存此權杖，並以此權杖將活動傳送給 Bot。
1. Bot 會收到包含權杖的活動，並且使用該權杖來對 Graph API 進行呼叫。

## <a name="securing-the-sign-in-url"></a>保護登入 URL

當 Bot Framework 促使使用者登入後，如何保護登入 URL 是重要的考量。 當使用者看到登入 URL 時，此 URL 會與特定的對話識別碼和該 Bot 的使用者識別碼相關聯。 此 URL 不得共用，因為這會造成特定 Bot 對話發生登入錯誤。 若要減少與共用登入 URL 相關的安全性攻擊，必須確保按下登入 URL 的機器和人員是_擁有_對話視窗的人員。

某些通道 (例如 Cortana、Teams、Direct Line 和 WebChat) 能夠在不通知使用者的情況下，執行該動作。 例如，WebChat 會使用工作階段 Cookie 確保登入流程會在 WebChat 對話所在的瀏覽器中發生。 不過，針對其他通道，使用者通常會看到 6 位數的_神奇代碼_。 這類似於內建的多重要素驗證，使用者必須完成最後的驗證，也就是輸入 6 位數代碼以證明登入的人員可存取聊天體驗後，Bot Framework 權杖服務才會將權杖釋放給 Bot。

## <a name="azure-activity-directory-application-registration"></a>Azure Activity Directory 應用程式註冊

已註冊為 Azure Bot Service 的每個 Bot 都會使用 Azure Activity Directory (AD) 應用程式識別碼。 請絕對**不要**重複使用此應用程式識別碼和密碼來登入使用者。 Azure Bot 服務的 Azure AD 應用程式識別碼會用來保護 Bot 和 Bot Framework 通道服務之間的服務對服務通訊。 如果您想要將使用者登入 Azure AD，應以適當的權限和範圍建立不同的 Azure AD 應用程式註冊。

## <a name="configure-an-oauth-connection-setting"></a>設定 OAuth 連線設定

如需如何註冊並使用 OAuth 連線設定的詳細資訊，請參閱[將驗證新增至您的 Bot](bot-builder-authentication.md)。
