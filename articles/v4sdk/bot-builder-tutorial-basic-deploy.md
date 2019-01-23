---
title: 建立和部署基本 Bot 的教學課程 | Microsoft Docs
description: 了解如何建立基本 Bot，然後將其部署至 Azure。
keywords: echo bot, 部署, azure, 教學課程
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dbde6eba946e27aaa6b883f1e9205adc63cb22f8
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360942"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>教學課程：建立及部署基本 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本教學課程會引導您使用 Bot Framework SDK 建立基本 Bot，然後將其部署至 Azure。 如果您已經建立基本 Bot 並且在本機執行，請直接跳到[部署您的 Bot](#deploy-your-bot) 一節。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立基本 Echo Bot
> * 在本機執行並與其互動
> * 發佈您的 Bot

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>部署您的 Bot

現在，將 Bot 部署至 Azure。

### <a name="prerequisites"></a>必要條件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>登入 Azure CLI 並設定訂用帳戶

您已在本機建立及測試 Bot，而現在想要將其部署至 Azure。

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>建立 Web 應用程式 Bot

如果您還沒有可供發佈 Bot 的資源群組，請建立一個：

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

在繼續之前，請根據您用來登入 Azure 的電子郵件帳戶類型，閱讀您適用的指示。

#### <a name="msa-email-account"></a>MSA 電子郵件帳戶

如果您使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子郵件帳戶，則必須在應用程式註冊入口網站上建立應用程式識別碼和應用程式密碼，以搭配 `az bot create` 命令使用。

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>公司或學校帳戶

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>從 Azure 下載 Bot

接下來，下載您剛建立的 Bot。 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>將下載的 .bot 檔案解密並使用於專案中

.bot 檔案中的敏感性資訊已加密，讓我們將其解密以便使用。 

首先，瀏覽至已下載的 Bot 目錄。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>在本機測試 Bot

此時，您 Bot 的運作方式應該與使用舊 `.bot` 檔案時相同。 確保使用新 `.bot` 檔案能如預期般運作。

您現在應該在模擬器中看見「生產」端點。 如果未看見，您可能仍然使用舊的 `.bot` 檔案。

### <a name="publish-your-bot-to-azure"></a>將 Bot 發佈至 Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

現在，您應該能夠在網路聊天中測試您的 Bot。

## <a name="additional-resources"></a>其他資源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [將服務新增至您的 Bot](bot-builder-tutorial-add-qna.md)

