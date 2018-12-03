---
title: 企業 Bot 建立新專案 | Microsoft Docs
description: 了解如何根據企業 Bot 範本建立新的 Bot
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bb2f8abccc75fcc1c63589bc41289443cf1fc211
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452000"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>企業 Bot 範本 - 建立新專案

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

企業 Bot 範本結合了我們透過打造對話式體驗所找出的所有最佳做法與支援元件。 此範本適用於下列 Botbuilder SDK 平台：

- .NET
- Node.js (敬請期待)

## <a name="net"></a>.NET

企業 Bot 範本適用於 .NET，以 SDK 的 **V4** 版本為目標。 其也可以 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 套件形式提供。 若要下載，請按下列連結：

- [BotBuilder SDK V4 企業 Bot 範本](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>必要條件

- [Visual Studio 2017 或更新版本](https://www.visualstudio.com/downloads/)
- [Azure 帳戶](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>安裝範本

從已儲存的目錄中，只要開啟 VSIX 套件，企業 Bot 範本就會安裝到 Visual Studio，並在您下次開啟時可供使用。

若要使用範本建立新的 Bot 專案，只要開啟 Visual Studio，然後選取 [檔案] > [新增] > [專案]，並從 Visual C# 中，選取 [Bot Framework] > [企業 Bot 範本] 即可。 這會在本機建立新的 Bot 專案，您可以視需要加以編輯。 

![提供新增專案範本](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>部署您的 Bot

既然您已建立專案，下一個步驟就是建立支援的 Azure 基礎結構並執行設定/部署，讓 Bot 能夠立即可用。 繼續[部署 Bot](bot-builder-enterprise-template-deployment.md)。

> 您必須執行此步驟，否則無法使用 Bot 的相依性 (Azure Bot Service、Application Insights、LUIS 等)。

## <a name="customize-your-bot"></a>自訂您的 Bot

確認成功部署現成的 Bot 之後，您可以針對您的案例和需求自訂 Bot。 繼續[自訂 Bot](bot-builder-enterprise-template-customize.md)。
