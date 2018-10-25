---
title: 從原始檔控制或 Visual Studio 發佈 Bot Service | Microsoft Docs
description: 了解如何從 Visual Studio 一次性地發佈 Bot Service Bot，或從原始檔控制持續發佈。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 38b26ed5a50409de64518562faabf532f45c857e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999145"
---
# <a name="publish-a-bot-to-bot-service"></a>將 Bot 發佈至 Bot Service

在您更新 C# Bot 原始程式碼之後，您可以使用 Visual Studio 將其發佈至正在 Bot Service 中執行的 Bot。 您也可以在每次將原始程式檔簽入原始檔控制服務時，自動發佈在任何整合式開發環境 (IDE) 中撰寫的 C# 或 Node.js Bot 原始程式碼。


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>使用線上程式碼編輯器在 App Service 方案上發佈 Bot

如果您未設定連續部署，您可以在線上程式碼編輯器中修改原始程式檔。 若要部署已修改的原始檔，請遵循下列步驟。

4. 按一下 [開啟主控台] 圖示。  
    ![主控台圖示](~/media/azure-bot-service-console-icon.png)
2. 在 [主控台] 視窗中輸入 **build.cmd**，然後按 Enter 鍵。


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>使用 Visual Studio 在 App Service 方案上發佈 C# Bot 

若要設定從 Visual Studio 使用 `.PublishSettings` 檔案進行發佈的作業，請執行下列步驟：

1. 在 Azure 入口網站中按一下您的 Bot Service，再按一下 [組建] 索引標籤，然後按一下 [下載 zip 檔案]。
3. 將已下載 zip 的檔案內容解壓縮到本機資料夾。
4. 在 [總管] 中找出 Bot 的 Visual Studio 解決方案 (.sln) 檔案，並按兩下。
4. 在 Visual Studio 中按一下 [檢視]，然後按一下 [方案總管]。
5. 在 [方案總管] 窗格中，以滑鼠右鍵按一下您的專案，然後按一下 [發佈...][發佈] 視窗隨即開啟。 
6. 在 [發佈] 視窗中按一下 [建立新的設定檔]，再按一下 [匯入設定檔]，然後按一下 [確定]。
7. 瀏覽至您的專案資料夾，再瀏覽至 **PostDeployScripts** 資料夾，選取以 `.PublishSettings` 結尾的檔案，然後按一下 [開啟]。

您現在已針對這個專案設定發佈。 若要將本機原始程式碼發佈至 Bot Service，請依序按一下您的專案、[發佈...] 和 [發佈] 按鈕。 

## <a name="set-up-continuous-deployment"></a>設定連續部署

根據預設，Bot Service 可讓您使用 Azure 編輯器直接在瀏覽器中開發 Bot，而不需要本機編輯器或原始檔控制。 不過，Azure 編輯器無法讓您在自己的應用程式中管理檔案 (例如，新增檔案、重新命名檔案或刪除檔案)。 如果您想要能夠在自己的應用程式中管理檔案，您可以設定連續部署，並使用整合式開發環境 (IDE) 和您選擇的原始檔控制系統 (例如 Visual Studio Team、GitHub、Bitbucket)。 設定連續部署後，您向原始檔控制認可的任何程式碼變更都會自動部署至 Azure。 在設定連續部署之後，您可以[在本機對 Bot 進行偵錯](bot-service-debug-bot.md)。

> [!NOTE]
> 如果您為 Bot 啟用連續部署，則必須將程式碼變更簽入原始檔控制服務中。 如果您想要再次使用 Azure 編輯器來編輯程式碼，您必須先[停用連續部署](#disable-continuous-deployment)。

您可以完成下列步驟，為 Bot 應用程式啟用連續部署。

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>在 App Service 方案上為 Bot 設定連續部署

本節說明如何為使用具有 App Service 主控方案的 Bot Service，所建立的 Bot 啟用連續部署。

1. 在 Azure 入口網站中找出您的 Azure Bot，按一下 [組建] 索引標籤，然後找出 [從原始檔控制進行連續部署] 區段。
2. 針對 Visual Studio Online 或 GitHub，請提供在這些網站上簽發給您的存取權杖。 您的原始檔會從 Azure 提取到原始檔存放庫中。
3. 針對其他原始檔控制系統，請選取 [其他] 並依照顯示的步驟操作。 
3. 按一下 [啟用] 。  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>建立空的存放庫並下載 Bot 原始程式碼

如果您想要使用 Visual Studio Online 或 Github *以外的*原始檔控制服務，請遵循下列步驟。 Visual Studio Online 和 Github 會從 Azure 提取 Bot 的原始程式碼，因此這兩項服務的使用者可略過這些步驟。

3. 針對 App Service 方案上的 Bot，請在 Azure 上找出您的 Bot 頁面、按一下 [建置] 索引標籤，找出 [下載原始程式碼] 區段，然後按一下 [下載 zip 檔案]。
1. 在 Azure 支援的其中一個原始檔控制系統中，建立空的存放庫。

    ![原始檔控制系統](~/media/continuous-integration-sourcecontrolsystem.png)

3. 將已下載的 zip 檔案內容解壓縮到您預計要同步部署來源的本機資料夾。
4. 按一下 [設定]，並依照顯示的步驟操作。 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>在取用方案上為 Bot 設定連續部署 

選擇 Bot 的部署來源，然後連線至您的存放庫。 

1. 在 Azure 入口網站中，在您的 Azure Bot 內按一下 [設定] 索引標籤，然後按一下 [設定] 以展開 [連續部署] 區段。  
2. 依照步驟操作，然後按一下核取方塊，以確認您已準備就緒。 
3. 按一下 [設定]，選取您先前用來建立空存放庫的原始檔控制系統所對應的部署來源，然後完成步驟加以連線。   


## <a name="disable-continuous-deployment"></a>停用連續部署 

當您停用連續部署時，原始檔控制服務會繼續運作，但您簽入的變更將不會自動發佈至 Azure。 若要停用連續部署，請執行下列步驟：

1. 如果您的 Bot 具有 App Service 主控方案，請在 Azure 入口網站中找出您的 Azure Bot，按一下 [組建] 索引標籤，然後找出 [從原始檔控制進行連續部署] 區段，*或...* 
2. 如果您的 Bot 具有取用方案，請按一下 [設定] 索引標籤、展開 [連續部署] 區段，然後按一下 [設定]。
3. 在 [部署] 窗格中，選取啟用連續部署的原始檔控制服務，然後按一下 [中斷連線]。  


## <a name="additional-resources"></a>其他資源

若要了解在設定連續部署後如何在本機對 Bot 進行偵錯，請參閱[對 Bot Service Bot 進行偵錯](bot-service-debug-bot.md)。

本文著重於 Bot Service 的特定連續部署功能。 如需連續部署如何運用於 Azure App Service 的相關資訊，請參閱<a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">持續部署至 Azure App Service</a>。
