---
title: 設定 Bot Service 的持續部署 | Microsoft Docs
description: 了解如何設定從 Bot Service 原始檔控制的持續部署。
keywords: 持續部署, 發佈, 部署, Azure 入口網站
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dc4b524659df3fb0b91b54fca65b4b7dd36378cd
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214273"
---
# <a name="set-up-continuous-deployment"></a>設定連續部署

[!INCLUDE [applies-to](./includes/applies-to.md)]

本文將示範如何設定 Bot 的持續部署。 您可以啟用持續部署，自動將來源存放庫中的程式碼變更部署至 Azure。 在本主題中，我們將討論如何設定 GitHub 的持續部署。 若要了解其他原始檔控制系統的持續部署設定，請參閱此頁面底部的 [其他資源] 區段。

## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](http://portal.azure.com) 。
- 您**必須**先[將 Bot 部署至 Azure](bot-builder-deploy-az-cli.md)，才能啟用持續部署。

## <a name="prepare-your-repository"></a>準備您的存放庫
確定您的存放庫根目錄含有專案中的正確檔案。 這可讓您從 Azure App Service Kudu 組建伺服器取得自動建置。 

|執行階段 | 根目錄檔案 |
|:-------|:---------------------|
| ASP.NET Core | .sln 或 .csproj |
| Node.js | 含啟動指令碼的 server.js、app.js 或 package.json |


## <a name="continuous-deployment-using-github"></a>使用 GitHub 進行持續部署
若要啟用搭配 GitHub 的持續部署，請巡覽至 Azure 入口網站中適用您 Bot 的 **App Service** 頁面。

按一下 [部署中心]   > [GitHub]   > [授權]  。

![持續部署](~/media/azure-bot-build/azure-deployment.png)

在開啟的瀏覽器視窗中，按一下 [授權 AzureAppService]  。 

![Azure Github 權限](~/media/azure-bot-build/azure-deployment-github.png)

在授權 **AzureAppService** 後，返回 Azure 入口網站中的**部署中心**。

1. 按一下 [繼續]  。 

1. 選取 [App Service 建置服務]  。

1. 按一下 [繼續]  。

1. 選取 [組織]  、[存放庫]  和 [分支]  。

1. 按一下 [繼續]  ，然後按一下 [完成]  以完成設定。

此時，搭配 GitHub 的持續部署已設定完成。 每當您認可來源程式碼存放庫時，變更就會自動部署至 Azure Bot 服務。

## <a name="disable-continuous-deployment"></a>停用連續部署

設定 Bot 以進行持續部署時，您可能不會使用線上程式碼編輯器來變更 Bot。 如果您想要使用線上程式碼編輯器，可以暫時停用持續部署。

若要停用持續部署，請執行下列作業：
1. 在 [Azure 入口網站](https://portal.azure.com)中，請移至 Bot 的 [所有 App Service 設定]  刀鋒視窗，然後按一下 [部署中心]  。 
1. 按一下 [中斷連線]  ，停用持續部署。 若要重新啟用持續部署，請重複上述適當章節的步驟。

## <a name="additional-resources"></a>其他資源
- 若要從 BitBucket 與 Azure DevOps Services 啟用連續部署，請參閱[使用 Azure App Service 進行持續部署](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment)。


