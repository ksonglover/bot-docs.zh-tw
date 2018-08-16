---
title: 將 Bot 連線至 GroupMe | Microsoft Docs
description: 深入了解如何設定 Bot 與 GroupMe 的連線。
keywords: Bot 頻道, GroupMe, 建立 GroupMe, 認證
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2cc3081f75c755d14f32f02fe9d0d3a3c48bcf14
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299395"
---
# <a name="connect-a-bot-to-groupme"></a>將 Bot 連線至 GroupMe

您可以設定您的 Bot 使用 GroupMe 群組傳訊應用程式與其他人通訊。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>註冊 GroupMe 帳戶

如果您沒有 GroupMe 帳戶，請註冊[免費新帳戶](https://web.groupme.com/signup)。

## <a name="create-a-groupme-application"></a>建立 GroupMe 應用程式

為您的 Bot [建立 GroupMe 應用程式](https://dev.groupme.com/applications/new)。

使用這個回呼 URL：`https://groupme.botframework.com/Home/Login`

![建立應用程式](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>收集認證

1. 在 [重新導向 URL] 欄位中，複製 **client_id=** 之後的值。
2. 複製 [存取權杖] 值。

![複製用戶端識別碼和存取權杖](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>提交認證

1. 在 dev.botframework.com，將您剛剛複製的 **client_id** 值貼到 [用戶端識別碼] 欄位。
2. 將 [存取權杖] 值貼到 [存取權杖] 欄位。
2. 按一下 [檔案] 。

![輸入認證](~/media/channels/GM-StepClientIDToken.png)
