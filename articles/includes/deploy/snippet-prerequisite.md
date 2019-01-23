---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360849"
---
- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。
- 安裝最新版的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安裝最新版的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 安裝 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 的最新發行版本。
- 安裝及設定 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- [.bot](~/v4sdk/bot-file-basics.md) 檔案的知識。

使用 msbot 4.3.2 和更新版本，您需要 Azure CLI 2.0.54 版或更新版本。 如果您安裝了 botservice 擴充功能，則使用此命令加以移除。

```cmd
az extension remove --name botservice
```