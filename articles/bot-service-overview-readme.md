---
title: Bot 服務的運作方式 | Microsoft Docs
description: 深入了解的 Bot 服務的特性和功能。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bdc86e5e64971e503157fe69a8b962e1d9b88542
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998865"
---
# <a name="how-bot-service-works"></a>Bot 服務 的運作方式

Bot 服務會提供建立 Bot 的核心元件，包括用於開發 Bot 的 Bot Builder SDK 和連線 Bot 與通道的 Bot Framework。 建立支援 .NET 和 Node.js 的 Bot 時，Bot 服務提供五種範本供您選擇。

> [!IMPORTANT]
> 您必須擁有 Microsoft Azure 訂用帳戶，才可以使用 Bot 服務。 若您還沒有訂用帳戶，則可以註冊<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免費帳戶</a>。

## <a name="hosting-plans"></a>主機方案
Bot 服務會為您的 Bot 提供兩項主控方案。 您可以選擇適合您需求的主控方案。

### <a name="app-service-plan"></a>App Service 方案

使用 App Service 方案的 Bot 是標準的 Azure Web 應用程式，您可以將它設定來配置可預測成本及規模的預先定義容量。 透過使用此主控方案的 Bot，您可以：

* 使用進階的瀏覽器內程式碼編輯器，在線上編輯 Bot 原始程式碼。
* 使用 Visual Studio 來對您的 C# Bot 進行下載、偵錯及重新發佈。
* 輕鬆設定適用於 Visual Studio Online 和 Github 的持續部署。
* 使用針對 Bot 建立器 SDK 準備的範例程式碼。

### <a name="consumption-plan"></a>取用方案
使用取用方案的 Bot 是在 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> 上執行的無伺服器 Bot，並使用按執行數付費的 Azure Functions 定價。 使用此主控方案的 Bot 可調整來處理大量的流量尖峰。 您可以使用基本的瀏覽器內程式碼編輯器，在線上編輯 Bot 原始程式碼。 如需取用方案 Bot 的執行階段環境詳細資訊，請參閱 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 取用和 App Service 方案</a>。

## <a name="templates"></a>範本

Bot 服務能讓您快速且輕鬆地使用五種範本之一，以 C# 或 Node.js 建立 Bot。

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

[深入了解](bot-service-concept-templates.md)不同的範本以及它們如何協助您建置 Bot。

## <a name="develop-and-deploy"></a>開發與部署

根據預設，Bot 服務可讓您使用線上程式碼編輯器，直接在瀏覽器中開發您的 Bot，不需要任何工具鏈。 

您可以使用 Bot Builder SDK 和 IDE 在本機開發及偵錯您的 Bot，例如 Visual Studio 2017。 您可以使用 Visual Studio 2017 或 Azure CLI 將您的 Bot 直接發佈到 Azure。 您也可以使用您選擇的 VSTS 或 GitHub 等原始檔控制系統[設定持續部署](bot-service-continuous-deployment.md)。 設定持續部署後，您就可以在本機電腦的 IDE 中開發及偵錯，而您向原始檔控制認可的任何程式碼變更都會自動部署至 Azure。  

> [!TIP]
> 啟用持續部署之後，請務必只透過連續部署修改您的程式碼，不要透過其他機制，以免發生衝突。

## <a name="manage-your-bot"></a>管理您的 Bot 

在使用 Bot 服務建立 Bot 的過程中，您要指定您的 Bot 名稱、設定其主控方案、選擇定價層，以及設定一些其他設定。 在建立您的 Bot 之後，您可以變更其設定、設定它在一或多個通道上執行，以及使用網路聊天測試它。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Bot 服務建立 Bot](bot-service-quickstart.md)