---
title: 在 VNET 內使用 Direct Line App Service 擴充功能
titleSuffix: Bot Service
description: 在 VNET 內使用 Direct Line App Service 擴充功能
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 7c565d77879641d92a3e331852ff38ea21fdaf9e
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866442"
---
# <a name="use-direct-line-app-service-extension-within-a-vnet"></a>在 VNET 內使用 Direct Line App Service 擴充功能

本文說明如何搭配使用 Direct Line App Service 擴充功能與 Azure 虛擬網路 (VNET)。

## <a name="create-an-app-service-environment-and-other-azure-resources"></a>建立 App Service 環境和其他 Azure 資源

1. Direct Line App Service 擴充功能適用於所有的 **Azure App Service**，包括 **Azure App Service 環境**內裝載的服務。 Azure App Service 環境可提供隔離，且很適合在 VNET 中運作。
    - 您可以在[建立外部 App Service 環境](https://docs.microsoft.com/en-us/azure/app-service/environment/create-external-ase)一文中找到建立外部 App Service 環境的指示。
    - 您可以在[建立及使用內部負載平衡器 App Service 環境](https://docs.microsoft.com/en-us/azure/app-service/environment/create-ilb-ase)一文中找到建立內部 App Service 環境的指示。
1. 建立 App Service 環境之後，您必須在其中新增 App Service 方案，用以部署您的 Bot (並且執行 Direct Line App Service 擴充功能)。 作法：
    - 移至 https://portal.azure.com/ 。
    - 建立新的「App Service 方案」資源。
    - 在 [區域] 下方，選取您的 App Service 環境
    - 完成建立 App Service 方案的作業

## <a name="configure-the-vnet-network-security-groups-nsg"></a>設定 VNET 網路安全性群組 (NSG)

1. Direct Line App Service 擴充功能需要輸出連線，才能發出 HTTP 要求。 這可以在您的 VNET NSG 中設定為與 App Service 環境的子網路相關聯的輸出規則。 所需的規則如下所示：

|來源|任意|
|---|---|
|來源連接埠|*|
|目的地|IP 位址|
|目的地 IP 位址|20.38.80.64、40.82.248.64|
|目的地連接埠範圍|443|
|通訊協定|任意|
|動作|允許|


![Direct Line 擴充功能架構](./media/channels/direct-line-extension-vnet.png)

>[!NOTE]
> 提供的 IP 位址僅供預覽使用。 我們將在下半年轉換為 Azure Bot Service 的服務標籤，屆時將會變更此組態。

### <a name="configure-your-bots-app-service"></a>設定 Bot 的 App Service

針對預覽版本，您必須變更 Direct Line App Service 擴充功能與 Azure 的通訊方式。 為此，您可以使用入口網站或 **檔案，將新的**App Service 應用程式設定`applicationsettings.json`新增至您的應用程式：

- 屬性：DirectLineExtensionABSEndpoint
- 值: https://st-directline.botframework.com/v3/extension

>[!NOTE]
> 只有預覽版的 Direct Line App Service 擴充功能才需要執行此作業。
