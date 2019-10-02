---
title: 建立和部署基本 Bot 的教學課程 | Microsoft Docs
description: 了解如何建立基本 Bot，然後將其部署至 Azure。
keywords: echo bot, 部署, azure, 教學課程
author: Ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c48d71c21d5e482b497cd38095af377ce2d04f6d
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2019
ms.locfileid: "71694541"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>教學課程：建立及部署基本 Bot

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

本教學課程會引導您使用 Bot Framework SDK 建立基本 Bot，然後將其部署至 Azure。 如果您已經建立基本 Bot 並且在本機執行，請直接跳到[部署您的 Bot](#deploy-your-bot) 一節。

在本教學課程中，您會了解如何：

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

### <a name="prerequisites"></a>必要條件
[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

### <a name="prepare-for-deployment"></a>準備部署
[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

#### <a name="1-login-to-azure"></a>1.登入 Azure
[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

#### <a name="2-set-the-subscription"></a>2.設定訂用帳戶
[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

#### <a name="3-create-an-app-registration"></a>3.建立應用程式註冊
[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

#### <a name="4-deploy-via-arm-template"></a>4.透過 ARM 範本進行部署
您可以將 Bot 部署在新的資源群組或現有的資源群組中。 選擇最適合您的選項。 
##### <a name="deploy-via-arm-template-with-new-resource-group"></a>**透過 ARM 範本部署 (使用新資源群組)**
[!INCLUDE [ARM with new resourece group](~/includes/deploy/snippet-ARM-new-resource-group.md)]

##### <a name="deploy-via-arm-template-with-existing-resource-group"></a>**透過 ARM 範本部署 (使用現有的資源群組)**
[!INCLUDE [ARM with existing resourece group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

#### <a name="5-prepare-your-code-for-deployment"></a>5.準備您的程式碼以進行部署
##### <a name="retrieve-or-create-necessary-iiskudu-files"></a>**擷取或建立必要的 IIS/Kudu 檔案**
[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

##### <a name="zip-up-the-code-directory-manually"></a>**手動壓縮程式碼目錄**
[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

### <a name="deploy-code-to-azure"></a>將程式碼部署至 Azure
[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

### <a name="test-in-web-chat"></a>在網路聊天中測試
[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-resources"></a>其他資源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [在 Bot 中使用 QnA Maker 回答問題](bot-builder-tutorial-add-qna.md)