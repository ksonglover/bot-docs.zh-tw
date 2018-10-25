---
title: 將 Skype 即時媒體 Bot 部署至 Azure | Microsoft Docs
description: 了解如何使用 Visual Studio 的內建的發佈功能將 Skype 即時音訊視訊 Bot 部署至 Azure 中。
author: MalarGit
ms.author: malarch
manager: ssulzer
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 22cce8ad5bef3c1c6f08a8efc28118e0209dd3af
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999435"
---
# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>將即時媒體 Bot 從 Visual Studio 部署至 Azure

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

即時媒體 Bot 可裝載於「IaaS」Azure 虛擬機器或「傳統」的 Azure 雲端服務中。 本文說明如何使用 Visual Studio 內建的發佈功能，部署裝載於 Azure 雲端服務背景工作角色的 Bot。

## <a name="prerequisites"></a>必要條件

您必須擁有 Microsoft Azure 訂用帳戶，才能將 Bot 部署至 Azure。 若您還沒有訂用帳戶，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。 此外，您必須擁有 Visual Studio 才能執行本文所述的程序。 如果您還沒有 Visual Studio，可以免費下載<a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a>。

### <a name="certificate-from-a-valid-certificate-authority"></a>來自有效憑證授權單位的憑證
您必須使用受信任的憑證授權單位 (CA) 的有效憑證設定 Bot。 主體名稱 (SN) 或憑證最後一個主體別名 (SAN) 項目即是雲端服務的名稱。 目前不支援萬用字元憑證。 如果您使用 CNAME 來指向雲端服務，CNAME 應為憑證的 SN 或最後一個 SAN 項目。

## <a name="configure-application-settings"></a>設定應用程式設定
若要讓 Bot 在雲端中正常運作，您必須確定其應用程式設定正確無誤。 更具體來說，您必須設定背景工作角色 app.config 檔案中的下列關鍵索引碼：
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> 若要尋找您 Bot 的**應用程式識別碼**和**應用程式密碼**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

## <a name="create-worker-role-in-the-azure-portal"></a>在 Azure 入口網站中建立背景工作角色
### <a name="step-1-create-cloud-serviceclassic"></a>步驟 1：建立雲端服務 (傳統)
登入 <a href="https://portal.azure.com">Azure 入口網站</a>。 按一下畫面左側的 **+**，然後選擇 [雲端服務 (傳統)]。 提供表單的必要資訊，然後按一下 [建立]。

![建立雲端服務](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> 在註冊 Bot 時應在 URL 提供 Bot 的 DNS 名稱。

### <a name="step-2-upload-the-certificate-for-the-bot"></a>步驟 2：上傳 Bot 憑證
建立 Bot 後，請上傳 Bot 的憑證。

![Upload certificate](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>使用背景工作角色的詳細資訊修改服務設定
Bot 的完全合格網域名稱 (FQDN) 不適用於 Azure RoleEnvironment API。 因此，必須與 FQDN 一起提供 Bot。 也必須了解 HTTPS 的憑證。 您可以在背景工作角色的服務設定 (.cscfg) 檔案中設定這些項目。

> [!TIP]
> 如果您要從 BotBuilder-RealTimeMediaCalling Git 存放庫部署範例，
> - 請將 $DnsName$ 替換為在服務設定中使用的雲端服務名稱或 CNAME。
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - 將 $CertThumbprint$ 替換為從設定上傳至下列行的 Bot 的憑證指紋。
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>從 Visual Studio 發佈 Bot
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>步驟 1：在 Visual Studio 中啟動 Microsoft Azure 發佈精靈

在 Visual Studio 中，開啟您的專案。 在 [方案總管] 中，以滑鼠右鍵按一下雲端服務專案，然後選取 [發佈]。 如此即可啟動 Microsoft Azure 發佈精靈。 使用您的認證來登入適當的訂用帳戶。

![以滑鼠右鍵按一下專案，然後選擇 [發佈] 即可啟動 Microsoft Azure 發佈精靈](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>步驟 2：發佈 Bot

按 [下一步] 。 這會開啟 [設定] 索引標籤。指定雲端服務、環境、組建組態和部署 Bot 的服務組態。

![按一下 [下一步] 並前往「設定」索引標籤](../media/real-time-media-bot-publish-settings.png)

您可以自行選擇**進階設定**並指定部署記錄檔 (用於偵錯問題) 的儲存體帳戶。

![按一下「進階設定」索引標籤](../media/real-time-media-bot-publish-advanced-settings.png)

驗證**摘要**索引標籤中的設定，然後按一下 [發佈] 以將 Bot 部署至 Microsoft Azure。
