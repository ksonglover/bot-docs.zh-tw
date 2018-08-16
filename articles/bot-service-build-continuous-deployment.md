---
title: 設定 Bot Service 的持續部署 | Microsoft Docs
description: 了解如何設定從 Bot Service 原始檔控制的持續部署。
keywords: 持續部署, 發佈, 部署, Azure 入口網站
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299199"
---
# <a name="set-up-continuous-deployment"></a>設定連續部署

持續部署可讓您在本機開發 Bot。 如果 Bot 簽入 **GitHub** 或 **Visual Studio Team Services** 等原始檔控制服務，持續部署會相當實用。 只要您將變更簽入回來源存放庫中，您的變更就會自動部署至 Azure。

> [!NOTE]
> 設定持續部署之後，[線上程式碼編輯器](bot-service-build-online-code-editor.md)會變成 *READ ONLY*。

本主題將說明如何針對 **GitHub** 和 **Visual Studio Team Services** 設定持續部署。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 進行持續部署

若要使用 GitHub 設定持續部署，請執行下列作業：

1. [派生](https://help.github.com/articles/fork-a-repo/)包含您要部署至 Azure 之程式碼的 GitHub 存放庫。
2. 從 Azure 入口網站，移至 Bot 的 [建置] 刀鋒視窗，然後按一下 [設定持續部署]。 
3. 按一下 [設定] 。
   
   ![設定持續部署](~/media/azure-bot-build/continuous-deployment-setup.png)

4. 按一下 [選擇來源]，然後選擇 [GitHub]。

   ![選擇 GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. 依序按一下 [授權] 和 [授權] 按鈕，然後遵循提示來提供 Azure 授權，以存取您的 GitHub 帳戶。

使用 GitHub 進行持續部署的設定已完成。 每當您認可時，變更便會自動部署至 Azure。

## <a name="continuous-deployment-using-visual-studio"></a>使用 Visual Studio 進行持續部署

1. 從 Bot 的 [建置] 刀鋒視窗，按一下 [設定持續部署]。 
2. 按一下 [設定] 。
   
   ![設定持續部署](~/media/azure-bot-build/continuous-deployment-setup.png)

3. 按一下 [選擇來源]，然後選擇 [Visual Studio Team Services]。

   ![選擇 Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. 按一下 [選擇您的帳戶]，然後選擇帳戶。

> [!NOTE]
> 如果看不到您的帳戶，請確定它連結至您的 Azure 訂用帳戶。
> 如需詳細資訊，請參閱 [Linking your VSTS account to your Azure subscription](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription) (將 VSTS 帳戶連結至 Azure 訂用帳戶)。

5. 按一下 [選擇專案]，然後選擇專案。

> [!NOTE]
> 只支援 VSTS Git 專案。

6. 按一下 [選擇分支]，然後選擇要分支的分支。
7. 按一下 [確定] 完成安裝程序。

   ![Visual Studio 設定](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

使用 Visual Studio Team Services 進行持續部署的設定已完成。 每當您認可時，變更便會自動部署至 Azure。

## <a name="disable-continuous-deployment"></a>停用連續部署

設定 Bot 以進行持續部署時，您可能不會使用線上程式碼編輯器來變更 Bot。 如果您想要使用線上程式碼編輯器，可以暫時停用持續部署。

若要停用持續部署，請執行下列作業：

1. 從 Bot 的 [建置] 刀鋒視窗，按一下 [設定持續部署]。 
2. 按一下 [中斷連線]，停用持續部署。 若要重新啟用持續部署，請重複上述適當章節的步驟。

## <a name="next-steps"></a>後續步驟
現在，您的 Bot 已設定好持續部署，請使用線上網路聊天來測試程式碼。

> [!div class="nextstepaction"]
> [在網路聊天中測試](bot-service-manage-test-webchat.md)
