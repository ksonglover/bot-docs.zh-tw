---
title: 下載並重新部署 Bot 服務原始程式碼 | Microsoft Docs
description: 了解如何下載及發佈 Bot 服務。
keywords: download source code, redeploy, deploy, zip file, publish, 下載原始程式碼, 重新部署, 部署, 壓縮文件, 發佈
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: b77e096d28f51f605db9c49d36e796553f9293ef
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756682"
---
# <a name="download-and-redeploy-bot-source-code"></a>下載並重新部署 Bot 服務原始程式碼

Bot 服務允許您下載 Bot 的整個來源專案。 這可讓您使用您選擇的 IDE，在本機上使用 Bot。 完成變更後，您可以將變更發佈回 Azure。 

本主題將說明如何下載您的 Bot 原始程式碼，並將變更發佈回 Azure。 

## <a name="download-bot-source-code"></a>下載 Bot 原始程式碼

若要在本機開發您的 Bot，請執行下列動作：

1. 在 Azure 入口網站中，開啟 Bot 的刀鋒視窗。
2. 在 [BOT 管理] 區段下，按一下 [建置]。
3. 按一下 [下載 zip 檔案]。 

   ![下載原始程式碼](~/media/azure-bot-build/download-zip-file.png)

4. 將 .zip 檔案解壓縮至本機目錄。
5. 瀏覽到解壓縮的資料夾並在您慣用的 IDE 中開啟原始程式檔。
6. 對資源進行變更。 編輯現有的來源檔案，或在專案中加入新的檔案。

當您就緒時，您可以將原始程式檔發佈回 Azure。

## <a name="publish-node-bot-source-code-to-azure"></a>將節點 Bot 原始程式碼發佈至 Azure

若要安裝這些套件，請從命令提示字元瀏覽至您的專案目錄，然後執行下列 NPM 命令。

**注意：** 這些套件只需要加入一次。

```console
npm install --save fs
npm install --save path
npm install --save request
npm install --save zip-folder
```

現在您已準備好將專案發佈至 Microsoft Azure。 若要將專案發佈至 Microsoft Azure 中，請在命令提示字元中執行下列 NPM 命令：

```console
npm run azure-publish
```

> [!NOTE]
> 如果在此 NPM 命令之後遇到錯誤，則您可能需要將 `"scripts": {"azure-publish": "node publish.js"}` 新增到您的 `package.json` 檔案，然後再執行一次。

## <a name="publish-c-bot-source-code-to-azure"></a>將 C# Bot 原始程式碼發佈至 Azure

使用 Visual Studio 將 C# 程式碼發佈至 Azure 是兩個步驟的程序：首先，您必須設定發佈設定。 然後，您就可以發佈您的變更。

若要設定從 Visual Studio 發佈，請執行下列動作：

1. 在 Visual Studio中，按一下 [方案總管]。
2. 以滑鼠右鍵按一下專案名稱，然後按一下 [發佈]。[發佈] 視窗隨即開啟。
3. 依序按一下 [建立新設定檔]、[匯入設定檔]，然後按一下 [確定]。
4. 瀏覽至您的專案資料夾，然後瀏覽至 **PostDeployScripts** 資料夾，並選取以 **.PublishSettings** 結尾的檔案。 按一下 [開啟] 。

您的專案現在已設定為將變更發佈至 Azure。

設定您的專案後，您可以透過執行下列動作將 Bot 原始程式碼發佈回 Azure：

1. 在 Visual Studio中，按一下 [方案總管]。
2. 以滑鼠右鍵按一下專案名稱，然後按一下 [發佈]。
3. 按一下 [發佈] 按鈕，將變更發佈至 Azure。

## <a name="next-steps"></a>後續步驟
現在您已了解如何在本機建置 Bot，您可以為您的 Bot 設定持續部署。

> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
