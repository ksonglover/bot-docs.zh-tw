---
title: 使用 Azure CLI 部署 Bot | Microsoft Docs
description: 將您的 Bot 部署至 Azure 雲端。
keywords: 部署 bot, azure 部署, 發佈 bot, az 部署 bot, visual studio 部署 bot, msbot 發佈, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360738"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何使用 `az` 和 `msbot` 將 C# 和 JavaScript Bot 部署至 Azure。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# Bot

您已在本機建立及測試 Bot，而現在想要將其部署至 Azure。 這些步驟假設您已建立必要的 Azure 資源。

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

.bot 檔案中的敏感性資訊已加密。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="update-the-bot-file"></a>更新 .bot 檔案

如果 Bot 使用 LUIS、QnA Maker 或分派服務，您必須將這些參考新增至 .bot 檔案。 否則，您可以略過此步驟。

1. 使用新的 .bot 檔案，在 BotFramework Emulator 中開啟 Bot。 Bot 不需要在本機執行。
1. 在 [BOT 總管] 面板中，展開 [服務] 區段。
1. 若要新增 LUIS 應用程式的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增 Language Understanding (LUIS)]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的 LUIS 應用程式清單。 選取您的 Bot 適用的項目。
1. 若要新增 QnA Maker 知識庫的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增 QnA Maker]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的知識庫清單。 選取您的 Bot 適用的項目。
1. 若要新增分派模型的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增分派]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的分派模型清單。 選取您的 Bot 適用的項目。

### <a name="test-your-bot-locally"></a>在本機測試 Bot

此時，您 Bot 的運作方式應該與使用舊 .bot 檔案時相同。 確保使用新 .bot 檔案能如預期般運作。

### <a name="publish-your-bot-to-azure"></a>將 Bot 發佈至 Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>其他資源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
