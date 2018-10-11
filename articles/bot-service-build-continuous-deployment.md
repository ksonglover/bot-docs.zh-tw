---
title: 設定 Bot Service 的持續部署 | Microsoft Docs
description: 了解如何設定從 Bot Service 原始檔控制的持續部署。
keywords: 持續部署, 發佈, 部署, Azure 入口網站
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
ms.openlocfilehash: 45c89c911106d5b6a1e250f6e6ab3d472c90ab92
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707984"
---
# <a name="set-up-continuous-deployment"></a>設定連續部署
如果您將程式碼簽入 **GitHub** 或 **Azure DevOps (前身為 Visual Studio Team Services)**，請使用持續部署來自動將來源存放庫中的程式碼變更部署至 Azure。 在本主題中，我們將討論如何設定 **GitHub** 和 **Azure DevOps** 的持續部署。

> [!NOTE]
> 設定持續部署之後，Azure 入口網站中的線上程式碼編輯器會變成 READ ONLY。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 進行持續部署

若要使用 GitHub 設定持續部署，請執行下列作業：

1. 使用您要部署至 Azure 的來源程式碼所在的 GitHub 存放庫。 您可以將現有存放庫[分支](https://help.github.com/articles/fork-a-repo/)或建立您自己的存放庫，並將相關的來源程式碼上傳到 GitHub 存放庫。
2. 在 [Azure 入口網站](https://portal.azure.com)中，移至 Bot 的 [建置] 刀鋒視窗，然後按一下 [設定持續部署]。 
3. 按一下 [設定] 。
   
   ![設定持續部署](~/media/azure-bot-build/continuous-deployment-setup.png)

4. 按一下 [選擇來源]，然後選取 [GitHub]。

   ![選擇 GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. 依序按一下 [授權] 和 [授權] 按鈕，然後遵循提示來提供 Azure 授權，以存取您的 GitHub 帳戶。

6. 按一下 [選擇專案]，然後選取專案。

7. 按一下 [選擇分支]，然後選取分支。

8. 按一下 [確定] 完成安裝程序。

使用 GitHub 進行持續部署的設定現已完成。 每當您認可來源程式碼存放庫時，變更就會自動部署至 Azure Bot 服務。

## <a name="continuous-deployment-using-azure-devops"></a>使用 Azure DevOps 進行持續部署

1. 在 [Azure 入口網站](https://portal.azure.com)中，從 Bot 的 [建置] 刀鋒視窗，按一下 [設定持續部署]。 
2. 按一下 [設定] 。
   
   ![設定持續部署](~/media/azure-bot-build/continuous-deployment-setup.png)

3. 按一下 [選擇來源]，然後選取 [Visual Studio Team Services]。 請記住，Visual Studio Team Services 現在是 Azure DevOps Services。

   ![選擇 Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. 按一下 [選擇帳戶]，然後選取帳戶。

> [!NOTE]
> 如果看不到您的帳戶，請確定它連結至您的 Azure 訂用帳戶。 若要將帳戶連結至 Azure 訂用帳戶，請移至 Azure 入口網站，開啟 **Azure DevOps Services 組織 (先前稱為 Team Services)**。 您會看到您在 Azure DevOps 中擁有的組織清單。 對具有所要部署 Bot 來源程式碼的組織按一下，您便會發現 [連接 AAD] 按鈕。 如果所選組織未連結至 Azure 訂用帳戶，此按鈕會有作用。 因此，請按一下此按鈕以設定連線。 您可能需要等候一些時間才會生效。

5. 按一下 [選擇專案]，然後選取專案。

> [!NOTE]
> 只支援 VSTS Git 專案。

6. 按一下 [選擇分支]，然後選取分支。
7. 按一下 [確定] 完成安裝程序。

   ![Visual Studio 設定](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

使用 Azure DevOps 進行持續部署的設定現已完成。 每當您認可時，變更便會自動部署至 Azure。

## <a name="disable-continuous-deployment"></a>停用連續部署

設定 Bot 以進行持續部署時，您可能不會使用線上程式碼編輯器來變更 Bot。 如果您想要使用線上程式碼編輯器，可以暫時停用持續部署。

若要停用持續部署，請執行下列作業：

1. 從 Bot 的 [建置] 刀鋒視窗，按一下 [設定持續部署]。 
2. 按一下 [中斷連線]，停用持續部署。 若要重新啟用持續部署，請重複上述適當章節的步驟。

## <a name="additional-information"></a>其他資訊
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
