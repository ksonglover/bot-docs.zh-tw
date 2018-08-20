---
title: 將 Bot 移轉至 Azure | Microsoft Docs
description: 了解如何將您的 Bot 從舊版 Bot Framework 入口網站移轉至 Azure 入口網站中的 Bot Service。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 6/22/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e8c927ce49a2d8aecf0113a09cbedcf1135a8380
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299655"
---
# <a name="migrate-your-bot-to-azure"></a>將 Bot 移轉至 Azure

所有在 [Bot Framework 入口網站](http://dev.botframework.com)中建立的 **Azure Bot Service (預覽版)** Bot，都必須移轉至 Azure 中的新 Bot Service。 此服務已在 2017 年 12 月正式推出 (GA)。 

請注意，僅連線至下列通道的註冊 Bot *不需要*移轉：**Teams**、**Skype** 或 **Cortana**。 例如，連線至 **Facebook** 和 **Skype** 的註冊 Bot *需要*移轉，但連線至 **Skype** 和 **Cortana** 註冊 Bot 則*不需要*移轉。

> [!IMPORTANT]
> 移轉使用 Node.js 建立的 Functions Bot 之前，您必須使用 **Azure Functions Pack** 將 **node_modules** 模組封裝在一起。 這麼做可改善移轉期間的效能和 Functions Bot 在移轉之後的執行效能。 若要封裝您的模組，請參閱[使用 Funcpack 封裝 Functions Bot](#package-a-functions-bot-with-funcpack)。

若要移轉您的 Bot，請執行下列作業：

1. 登入 [Bot Framework 入口網站](http://dev.botframework.com)，然後按一下 [我的 Bot]。
2. 為您要移轉的 Bot 按一下 [移轉] 按鈕。
3. 接受**條款**，然後按一下 [移轉] 開始進行移轉程序，或按一下 [取消] 以取消此動作。

> [!IMPORTANT]
> 移轉工作正在進行時，請勿離開頁面或重新整理頁面。 這麼做會導致移轉工作過早停止，且您將必須重新執行此動作。 為了確保移轉順利完成，請等候確認訊息出現。

移轉程序順利完成後，[移轉狀態] 會指出您的 Bot 已移轉，且 [回復移轉] 按鈕在移轉日期後的一週內都可供使用，以備在問題發生時回復。

按一下已移轉的 Bot 名稱，將會在 [Azure 入口網站](http://portal.azure.com)中開啟該 Bot。

## <a name="package-a-functions-bot-with-funcpack"></a>使用 Funcpack 封裝 Functions Bot

使用 Node.js 建立的 Functions Bot 在移轉之前，必須先使用 [Funcpack](https://github.com/Azure/azure-functions-pack) 進行封裝。 若要以 Funcpack 封裝您的專案，請執行下列步驟：

1.  將您的程式碼[下載](bot-service-build-download-source-code.md#download-bot-source-code)到本機 (如果尚未下載)。
2.  將 **packages.json** 中的 npm 套件更新為最新版本，然後執行 `npm install`。
3.  開啟 **messages/index.js**，並將 `module.exports = { default: connector.listen() }` 變更為 `module.exports = connector.listen();`
4.  透過 npm 安裝 Funcpack：`npm install -g azure-functions-pack`
5.  若要封裝 **node_modules** 目錄，請執行下列命令：`funcpack pack ./`
6.  使用 Bot Framework 模擬器執行 Functions Bot，在本機測試您的 Bot。 請在[這裡](https://github.com/Azure/azure-functions-pack#how-to-run)深入了解如何執行 *Funcpack* Bot。 
7.  將您的程式碼上傳回 Azure。 請確定 `.funcpack` 目錄已上傳。 您不需要上傳 **node_modules** 目錄。
8. 測試您遠端的 Bot，以確定它會如預期回應。
9. 使用上述步驟[移轉您的 Bot](#migrate-your-bot-to-azure)。

## <a name="migration-under-the-hood"></a>實際移轉狀況

您可以根據所要移轉的 Bot 類型，利用下列清單深入了解實際的運作情況。

* **Web 應用程式 Bot** 或 **Functions Bot**：針對這些類型的 Bot，系統會將舊有 Bot 中原始程式碼專案複製到新的 Bot。 其他資源 (例如 Bot 的儲存體、Application Insights、LUIS 等) 則會保持原狀。 在這類情況下，新 Bot 中現有資源會具有識別碼/金鑰/密碼的複本。 
* **Bot 通道註冊**：針對此類型的 Bot，移轉程序只會建立新的 **Bot 通道註冊**，並從舊的 Bot 將端點複製過去。 
* 無論您要移轉哪些類型的 Bot，移轉程序都不會以任何形式修改現有 Bot 的狀態。 這可讓您在必要時安全地進行回復。
* 如果您設定了[持續部署](bot-service-build-continuous-deployment.md)，您將必須重新設定，原始檔控制才會轉而連線至新的 Bot。

## <a name="understanding-azure-resources-after-migration"></a>了解移轉之後的 Azure 資源
移轉完成後，您的**資源群組**將會包含 Bot 運作所需的多項 Azure 資源。 資源的類型和數目取決於您所移轉的 Bot 類型。 若要深入了解，請參閱以下各節。

### <a name="registration-bot"></a>註冊 Bot

這是最簡單的類型。 Azure 中的資源群組只會包含一項屬於「Bot 通道註冊」的資源。 此資源相當於先前位於 Bot Framework 開發人員入口網站中的 Bot 記錄。

![Azure 中的 Bot 通道註冊 Bot 清單](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Web 應用程式 Bot
「Web 應用程式 Bot」移轉將會佈建「Web 應用程式 Bot」類型的 Bot Service 資源，和新的 App Service Web 應用程式 (在下列螢幕擷取畫面中以綠色顯示)。 先前的 Azure Bot Service (預覽版) Bot 會保留於原處，且可以刪除 (在下列螢幕擷取畫面中以紅色顯示)。

![Azure 中的 Web 應用程式 Bot 清單](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Functions Bot
Functions Bot 移轉將會佈建 "Functions Bot" 類型的 Bot Service 資源，和新的 App Service Functions 應用程式 (在下列螢幕擷取畫面中以綠色顯示)。 先前的 Azure Bot Service (預覽版) Bot 會保留於原處，且可以刪除 (在下列螢幕擷取畫面中以紅色顯示)。

![Azure 中的 Functions Bot 清單](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>回復移轉

如果 Bot 在移轉期間或移轉之後發生錯誤，您可以**回復移轉**。 若要回復移轉，請執行下列作業：

1. 登入 [Bot Framework 入口網站](http://dev.botframework.com)，然後按一下 [我的 Bot]。
2. 為您要回復的 Bot 按一下 [回復移轉] 按鈕。 此時會出現提示。
3. 按一下 [是，我要回復] 以繼續作業，或按 [取消] 以取消回復動作。

> [!NOTE]
> 回復功能可在移轉後的一週內使用，但只有在移轉後的 Bot 有問題時，才應該使用。

## <a name="migration-troubleshootingknown-issues"></a>移轉疑難排解/已知問題
我的 node.js/Functions Bot 已成功移轉，但無法回應：

* **檢查端點**
  * 移至 Bot 資源中的 [設定] 刀鋒視窗，並確認 Bot 端點具有 "code" 查詢字串參數及其值。 如果沒有，您必須將其加入。
* **在 Kudu 中檢查備份的祕密資料夾**
  * 在某些罕見的情況下，某些備份祕密檔案可能會產生衝突。 請移至 Kudu 中的 **home\data\Functions\secrets** 資料夾，並刪除您在該處找到的任何 **host.snapshot** (或 **host.backup**) 檔案。 其中應該只有一個 **host.json** 和一個 **messages.json**。 最後，請重新啟動 App Service，然後重新嘗試與 Bot 聊天。

若有任何其他問題，請將 CRI 提交至 Azure 支援，或在 [GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues) 中提出問題。


## <a name="next-steps"></a>後續步驟

現在，您的 Bot 已移轉，接下來請了解如何從 Azure 入口網站管理您的 Bot。

> [!div class="nextstepaction"]
> [管理 Bot](bot-service-manage-overview.md)
