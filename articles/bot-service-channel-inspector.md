---
title: 預覽使用通道偵測器的 Bot 功能 |Microsoft Docs
description: 了解各種 Bot Framework 功能的外觀以及如何使用通道偵測器來處理不同頻道。
keyword: bot, preview features, channel, display, Channel Inspector, Text formatting, markdown, XML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: def3f811704af2e2a22612ba5755eb5dbd2d8044
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299171"
---
# <a name="preview-bot-features-with-the-channel-inspector"></a>預覽使用通道偵測器的 Bot 功能

Bot Framework 可讓您建立各種不同功能的機器人，例如文字、按鈕、影像、展示在轉盤或清單格式的多媒體卡片或更多功能。 不過，每個通道最終控制其傳訊客戶端呈現功能的方式。

即使當多個頻道支援某一功能，每個頻道呈現該功能的方式也可能稍有不同。 在一訊息中包含通道原本不支援的功能之情況下，該通道會嘗試關閉呈現訊息的內容變為文字或靜態影像，這可能會嚴重影響用戶端上的訊息的外觀。 在某些情況下，通道可能完全不支援特定的功能。 舉例來說，GroupMe 用戶端無法顯示正在輸入的指標。

## <a name="the-channel-inspector"></a>通道偵測器

[通道偵測器][inspector]的建立是為了讓您可以預覽各種 Bot Framework 功能的外觀以及如何使用通道偵測器來處理不同頻道。 透過瞭解不同通道如何轉譯其功能，您可以設計機器人以在您溝通用的通道上取得卓越的使用者體驗。 通道偵測器也提供很好的方法讓您瞭解並以視覺化方式探索 Bot Framework 的功能。

> [!NOTE]
> 多媒體卡片是 Bot 資訊交換的開發標準，以確保不同通道上的顯示一致。 請參閱 [.NET][netcard] 或是 [Node.js][nodecard] 文件的多媒體卡片以取得相關詳細資訊。

## <a name="text-formatting"></a>文字格式

文字格式可以在視覺上提升您的文字訊息。 除了**純**文字，您的 Bot 可以傳送使用 **markdown** 或 **xml** 格式的文字訊息給支援的通道。 下列表格列出部分 **markdown** 與 **xml** 中最常使用的文字格式。 每個通道都可能支援比此表所列的文字格式多或少。 您可以確認[通道偵測器][inspector]來查看您想要使用的功能是否在您預計的通道上支援。

### <a name="markdown-text-format"></a>Markdown 文字格式

這些樣式在`textFormat`設定為 **markdown** 時，可能會支援:  

| Style | Markdown | 範例 |
| ---- | ---- | ---- |
| 粗體 | \*\*text\*\* | **text** |
| 斜體 | \*text\* | *text* |
| 標題 (1-5) | # H1 | # H1 |
| 刪除線 | \~\~text\~\~ | ~~text~~ |
| 水平尺規 | ---- | ---- |
| 未排序清單 | \*文字 |  <ul><li>text</li></ul> |
| 已排序清單 | 1. 文字 | 1. 文字 |
| 預先格式化的文字 | \`text\` | `text`  |
| 區塊引述 | \> 文字 | <blockquote>text</blockquote> |
| 超連結 | \[bing](http://www.bing.com) | [Bing](http://www.bing.com) |
| 影像連結| !\[duck](http://aka.ms/Fo983c) | ![Duck](http://aka.ms/Fo983c) |

> [!NOTE]
> **Markdown** 中的 HTML 標籤在 Microsoft Bot Framework 網路聊天頻道中不支援。 如果您要在 **Markdown** 中，使用 HTML 標籤，您可以在支援它們的 [Direct Line](~/bot-service-channel-connect-directline.md) 通道中轉譯。 或者，您可以藉由設定 `textFormat` 為 **xml** 並將您的 Bot 連接到 Skype 通道來使用下列的 HTML 標籤。

### <a name="xml-text-format"></a>XML 文字格式

這些樣式在`textFormat`設定為 **markdown** 時，可能會支援:

|      Style      |                       XML                       |                   範例                   |
|-----------------|-------------------------------------------------|---------------------------------------------|
|      粗體       |                 \<b\>文字\</b\>                 |            <strong>text</strong>            |
|     斜體      |                 \<i\>文字\</i\>                 |                <em>text</em>                |
|    底線    |                 \<u\>文字\</u\>                 |                 <u>text</u>                 |
|  刪除線  |                 \<s\>文字\</s\>                 |                 <s>text</s>                 |
|    超連結    |   \<a href="http://www.bing.com"\>bing\</a\>    |   <a href="http://www.bing.com">Bing</a>    |
|    段落    |                 \<p\>文字\</p\>                 |                 <p>text</p>                 |
|   分行符號    |                     \<br/\>                     |             第 1 行 <br/>第 2 行              |
| 水平尺規 |                     \<hr/\>                     |                    <hr/>                    |
|  標題 (1-4)   |                \<h1\>文字\</h1\>                |             <h1>標題 1</h1>              |
|      code       |           \<code\>程式碼區塊 \< /code\>           |           <code>code block</code>           |
|      映像      | \<img src="<http://aka.ms/Fo983c>" alt="Duck"\> | <img src="http://aka.ms/Fo983c" alt="Duck"> |

> [!NOTE]
> 只有 Skype 通道支援 `textFormat`**xml**。

## <a name="preview-features-across-various-channels"></a>預覽不同通道上的功能

若要查看通道如何呈現的特定功能，請前往[通道偵測器][inspector]，從**通道**清單上選取通道，並從**功能**清單中選擇功能。 舉例來說，若要查看 Skype 的 Hero 卡片的呈現方式，設定**通道**為 *Skype* 且**功能**為 *HeroCard*。

![通道偵測器會顯示 Skype 通道與 Hero 卡片](~/media/bot-service-channel-inspector.png)

通道偵測器會顯示所選功能在指定的通道上所呈現的預覽。 **備忘稿**區段會傳遞關於訊息限制及/或顯示變更的重要資訊。 舉例來說，某些類型的多媒體卡片只支援一個影像且某些功能可能會在特定的通道上關閉呈現。

> [!NOTE]
> 通道偵測器目前支援這些通道：
> * Cortana
> * 電子郵件
> * Facebook
> * GroupMe
> * Kik
> * Skype
> * 商務用 Skype
> * Slack
> * sms
> * Microsoft Teams
> * Telegram
> * 微信
> * 網路聊天
>
> 在未來可能會加入其他通道。

## <a name="features-that-can-be-previewed"></a>可預覽的功能

目前，通道偵測器可讓您預覽的下列功能。

|功能 | 說明|
| --- | ----|
| 調適型卡片 | 卡片可包含文字、語音、影像、按鈕和輸入欄位的任何組合。 |
| 按鈕| 使用者可以點按的按鈕。 按鈕與其所屬的訊息出現在對話畫布上。 |
| 浮動切換| 精簡、可捲動的水平清單卡片。 針對垂直版面配置，使用清單。|
| ChannelData| 一種傳遞中繼資料以存取卡片，文字和附件之外的特定通道功能的方法。|
| Codesample| 多行、預先格式化的文字區塊用來顯示電腦的程式碼。|
| DirectMessage| 傳送至單一群組對話成員的訊息。
| 文件| 傳送和接收非媒體附件。 |
| Emoji| 顯示支援的 Emoji。
| GroupChat| Bot 會傳送訊息以參與群組交談。 |
| HeroCard| 格式化的附件，通常包含單一的大型影像、點選動作及 (選擇性) 的按鈕，以及描述文字內容。 |
| 映像| 顯示映像附件。 |
| 清單版面配置| 垂直的卡片清單。 針對水平、可捲動的版面配置，使用浮動切換。|
| 位置| 與 Bot 分享使用者的實體位置。 |
| Markdown| 呈現使用 Markdown 格式化的文字。|
| 成員| 分享 Bot 群組聊天期間的對話成員清單。 |
| 回條卡| 向使用者顯示回條。 |
| 簽到卡| 要求使用者輸入驗證憑證。|
| 建議的動作 | 動作以使用者可以點擊輸入的按鈕方式呈現。 |
| 縮圖卡| 格式化的附件，包含單一的小型影像 (縮圖)、點選動作及 (選擇性) 的按鈕，以及描述文字內容。 |
| 輸入| 顯示輸入指標 告知使用者 Bot 仍正常運作，只是部分動作在背景執行是很有幫助的。|
| 影片| 顯示影片附件和播放控制項。|

## <a name="additional-resources"></a>其他資源

* [通道偵測器][inspector]
* 在 [Node.js][nodecard] 和 [.NET][netcard] 中的多媒體卡片
* 在 [Node.js][nodemedia] 和 [.NET][netmedia] 中的媒體附件
* 在 [Node.js][nodebutton] 和 [.NET][netbutton] 中的建議動作

[inspector]: https://docs.botframework.com/en-us/channel-inspector/channels/Skype/

[syntax]: https://daringfireball.net/projects/markdown/syntax

[netcard]: ~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md
[nodecard]: ~/nodejs/bot-builder-nodejs-send-rich-cards.md

[netmedia]: ~/dotnet/bot-builder-dotnet-add-media-attachments.md
[nodemedia]: ~/nodejs/bot-builder-nodejs-send-receive-attachments.md

[netbutton]: ~/dotnet/bot-builder-dotnet-add-suggested-actions.md
[nodebutton]: ~/nodejs/bot-builder-nodejs-send-suggested-actions.md
