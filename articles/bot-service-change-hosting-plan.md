---
title: 將 C# Bot 從使用情況方案遷移至 App Service 方案 | Microsoft Docs
description: 將 Bot Service C# Bot 從使用情況主控方案遷移至 App Service 主控方案。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: aafbfb2a38e2d5370cb2db5721dd7bc130497d74
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999215"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>變更您的 Bot Service 主控方案

本主題說明您可以如何將具有使用情況方案的 C# 指令碼 Bot，遷移至具有 App Service 方案的 C# Bot。 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>App Service 方案 Bot 的優點

App Service 方案上的 Bot 會以 Azure Web 應用程式的形式執行。 Web 應用程式 Bot 可以執行使用情況方案 Bot 無法執行的作業：

- Web 應用程式 Bot 可以新增自訂路由定義。
- Web 應用程式 Bot 可以啟用 Websocket 伺服器。 
- 利用使用情況主控方案的 Bot，具有與 Azure Functions 上所執行所有程式碼相同的限制。 如需詳細資訊，請參閱 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 使用情況和 App Service 方案</a>。

## <a name="download-your-existing-bot-source"></a>下載您現有的 Bot 原始碼

請遵循下列步驟來下載您現有 Bot 的原始程式碼：

1. 在您的 Azure Bot 內，按一下 [設定] 索引標籤，然後展開 [持續部署] 區段。  
2. 請按一下藍色按鈕以下載 zip 檔案，其中包含您 Bot 的原始程式碼。  
    ![下載 Bot zip 檔案](~/media/continuous-deployment-consumption-download.png)
3. 將已下載的 zip 檔案內容，解壓縮到本機資料夾。 


## <a name="create-a-bot-template"></a>建立 Bot 範本

Bot Service 對於使用情況方案和 App Service 方案 Bot 具有相同範本。 若要從使用情況方案 Bot 遷移，請根據相同的範本，在 Bot Service 中建立新的 App Service 方案 Bot。 相同範本兩個主控類型之間的基礎程式碼可能不相同，但是新 Web 應用程式的結構和組態功能，與您現有 Bot 所使用的類似。

## <a name="download-the-new-bot-source"></a>下載新的 Bot 原始碼

請遵循下列步驟來下載新 Bot 的原始程式碼：

1. 在您的 Azure Bot 內，按一下 [建置] 索引標籤，尋找 [下載原始程式碼] 區段，然後按一下 [下載 zip 檔案]。 
2. 將已下載的 zip 檔案內容，解壓縮到本機資料夾。

## <a name="add-source-files-to-new-solution"></a>將來源檔案新增至新的解決方案

某些 .csx 檔案可能會經過編譯，並且在新的解決方案中以 .cs 檔案的形式執行。 除了 `run.csx` 以外，請在您的解決方案中為每個 .csx 檔案建立 .cs 檔案。 您會手動遷移 `run.csx` 邏輯。 在您的 .cs 檔案中，可能需要新增類別宣告以及選擇性新增命名空間宣告。

## <a name="migrate-runcsx-logic-into-your-project"></a>將 run.csx 邏輯遷移至您的專案

C# 指令碼專案具有 `Run` 方法，該方法會處理不同的 `ActivityTypes` 值。 將您的活動處理邏輯匯入到 `MessageController.cs` 中的 `MessageController.Post` 方法。

## <a name="remove-compiler-keywords"></a>移除編譯器關鍵字

C# 指令檔可以使用 `#r` 關鍵字來包含參考模組。 請移除這幾行，同時也新增這些項目作為您 Visual Studio 專案的參考。 並移除 `#load` 關鍵字，這些關鍵字會在檔案編譯中插入其他原始程式碼檔案。 取而代之，將所有 `.csx` 檔案新增至您的專案，作為 `.cs` 原始程式碼。

## <a name="add-references-from-projectjson"></a>從 project.json 中新增參考

如果您的使用情況方案 Bot 會在其 `project.json` 檔案中新增 NuGet 參考，請以滑鼠右鍵按一下 [方案總管] 窗格中的專案，然後按一下 [新增參考]，將這些參考新增至新的 Visual Studio 解決方案。

### <a name="add-references-that-were-implicit"></a>新增隱含的參考

使用情況方案上的 Bot Service Bot，會以隱含方式在所有 .csx 來源檔案中包含這些參考。 遷移到 .cs 來源檔案的來源，可能會需要為這些類別新增明確參考：

- NuGet 套件 `Microsoft.Azure.WebJobs` 中的 `TraceWriter`，會提供 `Microsoft.Azure.WebJobs.Host` 命名空間類型。 
- NuGet 套件 `Microsoft.Azure.WebJobs.Extensions` 中的計時器觸發程序
- `Newtonsoft.Json`、`Microsoft.ServiceBus` 和其他自動參考的組件
- `System.Threading.Tasks` 和其他自動匯入的命名空間

如需其他指引，請參閱<a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>將 .NET 類別庫發佈為函式應用程式</a>中的「轉換為類別檔案」。

## <a name="debug-your-new-bot"></a>針對新的 Bot 進行偵錯

App Service 方案上的 Bot 比起使用情況方案上的 Bot，更容易在本機進行偵錯。 您可以使用[模擬器](bot-service-debug-emulator.md)，在本機針對遷移的程式碼進行偵錯。

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>從 Visual Studio 發佈，或設定持續部署

最後，藉由匯入原始程式碼的 `.PublishSettings` 檔案並按一下 [發佈]，或者[設定持續部署](bot-service-debug-bot.md)，將遷移的原始程式碼發佈到 Bot Service。
