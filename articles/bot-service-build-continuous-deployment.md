---
redirect_url: /bot-framework/bot-builder-deploy-az-cli
ms.openlocfilehash: 8149471553658df6778e2983bae114e80c846c9b
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971421"
---
<a name="--"></a><!--
---
標題：設定 Bot Service 的持續部署 | Microsoft Docs 描述：了解如何設定從 Bot Service 原始檔控制的持續部署。 keywords: 持續部署, 發佈, 部署, azure 入口網站 author: ivorb ms.author: v-ivorb manager: kamrani ms.topic: article ms.service: bot-service ms.date:12/06/2018
---

# <a name="set-up-continuous-deployment"></a>設定連續部署
如果您將程式碼簽入 **GitHub** 或 **Azure DevOps (前身為 Visual Studio Team Services)**，請使用持續部署來自動將來源存放庫中的程式碼變更部署至 Azure。 在本主題中，我們將討論如何設定 **GitHub** 和 **Azure DevOps** 的持續部署。

> [!NOTE]
> 本文中涵蓋的案例假設您已將 Bot 部署至 Azure，而您現在想要啟用該 Bot 的持續部署。 此外，知道在設定持續部署之後，Azure 入口網站中的線上程式碼編輯器會變成唯讀狀態。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 進行持續部署

若要設定使用您要部署至 Azure 的來源程式碼所在的 GitHub 存放庫進行連續部署，請執行下列作業：

1. 在 [Azure 入口網站](https://portal.azure.com)中，請移至 Bot 的 [所有 App Service 設定] 刀鋒視窗，然後按一下 [部署選項 (傳統)]。 

1. 按一下 [選擇來源]，然後選取 [GitHub]。

   ![選擇 GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. 依序按一下 [授權] 和 [授權] 按鈕，然後遵循提示來提供 Azure 授權，以存取您的 GitHub 帳戶。

1. 按一下 [選擇專案]，然後選取專案。

1. 按一下 [選擇分支]，然後選取分支。

1. 按一下 [確定] 完成安裝程序。

使用 GitHub 進行持續部署的設定現已完成。 每當您認可來源程式碼存放庫時，變更就會自動部署至 Azure Bot 服務。

## <a name="continuous-deployment-using-azure-devops"></a>使用 Azure DevOps 進行持續部署

1. 在 [Azure 入口網站](https://portal.azure.com)中，請移至 Bot 的 [所有 App Service 設定] 刀鋒視窗，然後按一下 [部署選項 (傳統)]。 
2. 按一下 [選擇來源]，然後選取 [Visual Studio Team Services]。 請記住，Visual Studio Team Services 現在是 Azure DevOps Services。

   ![選擇 Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. 按一下 [選擇帳戶]，然後選取帳戶。

> [!NOTE]
> 如果未看到您的帳戶列出，您必須[將帳戶連結至 Azure 訂用帳戶](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav)。 請注意，只支援 VSTS Git 專案。

4. 按一下 [選擇專案]，然後選取專案。
5. 按一下 [選擇分支]，然後選取分支。
6. 按一下 [確定] 完成安裝程序。

   ![Visual Studio 設定](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

使用 Azure DevOps 進行持續部署的設定現已完成。 每當您認可時，變更便會自動部署至 Azure。

## <a name="disable-continuous-deployment"></a>停用連續部署

設定 Bot 以進行持續部署時，您可能不會使用線上程式碼編輯器來變更 Bot。 如果您想要使用線上程式碼編輯器，可以暫時停用持續部署。

若要停用持續部署，請執行下列作業：
1. 在 [Azure 入口網站](https://portal.azure.com)中，請移至 Bot 的 [所有 App Service 設定] 刀鋒視窗，然後按一下 [部署選項 (傳統)]。 
2. 按一下 [中斷連線]，停用持續部署。 若要重新啟用持續部署，請重複上述適當章節的步驟。

## <a name="additional-information"></a>其他資訊
- Visual Studio Team Services 現在是 [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)


-->
