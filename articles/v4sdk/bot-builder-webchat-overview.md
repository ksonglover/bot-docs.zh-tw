---
title: 網路聊天概觀 | Microsoft Docs
description: 了解如何設定 Bot Framework 網路聊天。
keywords: bot framework, 網路聊天, 聊天, 範例, react, 參考
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 1d787d375fcd1ddade544724bb28dea9aaeb1dd6
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671414"
---
# <a name="web-chat-overview"></a>網路聊天概觀

本文包含 [Bot Framework 網路聊天](https://github.com/microsoft/BotFramework-WebChat)元件的詳細資料。 Bot Framework 網路聊天元件是具有高度自訂能力、適用於 Bot Framework V4 SDK 的 Web 型用戶端。 Bot Framework SDK v4 可讓開發人員建立對話模型，並建置精密的聊天機器人應用程式。

如果您想要從網路聊天 v3 遷移至 v4，請往下跳至[移轉區段](#migrating-from-web-chat-v3-to-v4)。

## <a name="how-to-use"></a>使用方式

> [!NOTE]
> 如需舊版網路聊天 (v3)，請瀏覽[網路聊天 v3 分支](https://github.com/Microsoft/BotFramework-WebChat/tree/v3)。

首先，請使用 [Azure Bot Service](https://azure.microsoft.com/services/bot-service/) 建立聊天機器人。
聊天機器人建立好之後，您必須在 Azure 入口網站中[取得聊天機器人的網路聊天祕密](../bot-service-channel-connect-webchat.md#step-1)。 然後使用該祕密[產生權杖](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md)，並將權杖傳遞至網路聊天。

以下會說明如何將網路聊天控制項新增至網站：

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> `userID`、`username`、`locale`、`botAvatarInitials` 和 `userAvatarInitials` 全都是可傳遞至 `renderWebChat` 方法的選擇性參數。 若要深入了解網路聊天屬性，請參閱此 `README` 的[網路聊天 API 參考](#web-chat-api-reference)區段。
> ![網路聊天的螢幕擷取畫面](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>與 JavaScript 整合

網路聊天依設計會使用 JavaScript 或 React 來與您現有的網站整合。 與 JavaScript 整合可讓您獲得適當的樣式和自訂能力。

您可以使用完整的典型網路聊天套件，其內含最常用的功能。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

請參閱[完整網路聊天套件組合](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)的實用範例。

### <a name="integrate-with-react"></a>與 React 整合

若要獲得完整的自訂功能，您可以使用 React 來重新撰寫網路聊天的元件。

若要從 NPM 安裝生產環境組件，請執行 `npm install botframework-webchat`。

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> 您也可以執行 `npm install botframework-webchat@master` 來安裝開發組建，並與網路聊天的 GitHub `master` 分支進行同步處理。

請參閱[透過 React 轉譯的網路聊天](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/)實用範例。

## <a name="customize-web-chat-ui"></a>自訂網路聊天 UI

網路聊天依設計會有自訂功能，而不需要對原始程式碼建立分支。 下表概述以不同方式匯入網路聊天時可進行自訂的種類。 此清單未涵蓋全部種類。

|                               | CDN 套件組合         | React              |
| ----------------------------- | ------------------ | ------------------ |
| 變更色彩                 | :heavy_check_mark: | :heavy_check_mark: |
| 變更大小                  | :heavy_check_mark: | :heavy_check_mark: |
| 更新/取代 CSS 樣式     | :heavy_check_mark: | :heavy_check_mark: |
| 接聽事件              | :heavy_check_mark: | :heavy_check_mark: |
| 與裝載網頁互動 | :heavy_check_mark: | :heavy_check_mark: |
| 自訂轉譯活動      |                    | :heavy_check_mark: |
| 自訂轉譯附件     |                    | :heavy_check_mark: |
| 新增 UI 元件         |                    | :heavy_check_mark: |
| 重新撰寫整個 UI        |                    | :heavy_check_mark: |

請參閱[自訂網路聊天](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md)的詳細資訊，以深入了解自訂功能。

## <a name="migrating-from-web-chat-v3-to-v4"></a>從網路聊天 v3 遷移至 v4

從 v3 遷移至 v4 時，可能的遷移路徑有三個。 首先，請比較您的起始案例：

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>我目前的網站使用取自 Azure Bot Service 的 `<iframe>` 元素來整合網路聊天。 我想要升級至 v4。

從 2019 年 5 月開始，我們便持續將 v4 部署到使用 `<iframe>` 元素來整合網路聊天的網站。 請參閱[內嵌文件](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed)來取得如何使用 `<iframe>` 整合網路聊天的相關資訊。

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>我的網站與網路聊天 v3 整合，並使用網路聊天所提供的自訂選項、完全沒使用自訂功能，或使用極少數網路聊天所沒有提供的自有自訂功能。

請遵循 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 範例的實作，來將網頁從網路聊天 v3 轉換為 v4。

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>我的網站與網路聊天 v3 的分支整合。 我已在網路聊天版本中實作許多自訂功能，所以我擔心 v4 無法與我的需求相容。

本團隊對於網路聊天 v4 最喜歡的其中一件事就是，此版本可以新增自訂功能，**而不需要對網路聊天建立分支**。 雖然這麼做會讓先前已對網路聊天建立分支的 v3 使用者產生額外負荷，但我們會盡力為遇到問題的客戶提供支援。 請使用下列建議：

-  看一下 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 範例的實作。 這是讓網路聊天啟動並執行的最佳起始位置。
-  接下來，請瀏覽[範例清單](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples)來比較您的自訂需求與網路聊天已支援的內容。 這些範例是由使用者經常要求的網路聊天功能所組成。
-  如果範例未提供您所使用的一或多個功能，則請瀏覽[已開啟和已結案的問題](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+)、[範例標籤](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample)與[移轉支援標籤](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22)，來針對您要尋找的功能搜尋其範例要求和/或自訂支援。 在已開啟的問題中新增註解，可協助該小組優先處理需求熱烈的要求，而且我們鼓勵使用者參與我們的社群。
-  如果您在已開啟的要求清單中找不到您的功能，請不用客氣，直接[提出您自己的要求](https://github.com/Microsoft/BotFramework-WebChat/issues/new)。 如同上面的項目，其他在已開啟的問題中新增註解的客戶，也會協助我們將網路聊天使用者最常需要的功能訂出先後次序。
-  最後，如果您需要盡快獲得所需的功能，我們也歡迎您提出網路聊天的[提取要求](https://github.com/Microsoft/BotFramework-WebChat/compare)。 如果您有自行實作該功能的編碼體驗，我們非常歡迎您提供額外支援！ 自行建立功能，表示您可以更快速地將該功能用在網路聊天上，且其他在尋找相同或類似功能的客戶也可利用您的貢獻。
-  請務必看一下此 `README` 的其餘內容，以深入了解 v4。

## <a name="samples-list"></a>範例清單

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;範例&nbsp;名稱&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 說明                                                                                                                                                                                                                         | 連結                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`01.a.getting-started-full-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)                                                                                       | 導入從 CDN 內嵌的網路聊天，並示範簡單但功能完整的網路聊天。 這包括調適型卡片、認知服務及 Markdown-It 相依性。                                                            | [完整的套件組合示範](https://microsoft.github.io/BotFramework-WebChat/01.a.getting-started-full-bundle)                               |
| [`01.b.getting-started-es5-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.b.getting-started-es5-bundle)                                                                                         | 導入功能完整且內嵌 ES5 瀏覽器回溯相容性的網路聊天 (使用網路聊天的 ES5 ponyfill)。                                                                                                                | [ES5 套件組合示範](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)                                 |
| [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)                                                                                           | 示範如何從網路聊天 v3 聊天機器人遷移至 v4。                                                                                                                                                                        | [移轉示範](https://microsoft.github.io/BotFramework-WebChat/01.c.getting-started-migration)                                   |
| [`02.a.getting-started-minimal-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.a.getting-started-minimal-bundle)                                                                                 | 導入只有基本相依性的最小化 CDN。 這不包括調適型卡片、認知服務相依性或 Markdown-It 相依性。                                                                      | [最小的套件組合示範](https://microsoft.github.io/BotFramework-WebChat/02.a.getting-started-minimal-bundle)                         |
| [`02.b.getting-started-minimal-markdown`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.b.getting-started-minimal-markdown)                                                                             | 示範如何針對以最小套件組合為基礎的 Markdown-It 相依性新增 CDN。                                                                                                                                            | [具有 Markdown 的最小套件組合示範](https://microsoft.github.io/BotFramework-WebChat/02.b.getting-started-minimal-markdown)                |
| [`03.a.host-with-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react)                                                                                                               | 示範如何建立 React 元件來裝載功能完整的網路聊天。                                                                                                                                                 | [使用 React 裝載的示範](https://microsoft.github.io/BotFramework-WebChat/03.a.host-with-react)                                       |
| [`03.b.host-with-Angular`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.b.host-with-angular)                                                                                                           | 示範如何建立 Angular 元件來裝載功能完整的網路聊天。                                                                                                                                              | [使用 Angular 裝載的示範](https://stackblitz.com/github/omarsourour/ng-webchat-example)                                              |
| [`04.a.display-user-bot-initials-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.a.display-user-bot-initials-styling)                                                                           | 示範如何顯示兩個網路聊天參與者的姓名縮寫。                                                                                                                                                                | [聊天機器人姓名縮寫示範](https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/04.a.display-user-bot-initials-styling/)  |
| [`04.b.display-user-bot-images-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.b.display-user-bot-images-styling)                                                                               | 示範如何顯示兩個網路聊天參與者的圖像和姓名縮寫。                                                                                                                                                     | [使用者圖像示範](https://microsoft.github.io/BotFramework-WebChat/04.b.display-user-bot-images-styling)                           |
| [`05.a.branding-webchat-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.a.branding-webchat-styling)                                                                                             | 導入為網路聊天設定樣式的功能以符合您的品牌形象。 這個自訂樣式方法不會在網路聊天更新時損壞。                                                                                                   | [商標網路聊天示範](https://microsoft.github.io/BotFramework-WebChat/05.a.branding-webchat-styling)                            |
| [`05.b.idiosyncratic-manual-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.b.idiosyncratic-manual-styling/)                                                                                    | 示範如何手動變更樣式，這種自訂網路聊天樣式的方法更加複雜且耗時。 手動設定的樣式可能會在網路聊天更新時損壞。                                                | [特殊樣式示範](https://microsoft.github.io/BotFramework-WebChat/05.b.idiosyncratic-manual-styling)                    |
| [`05.c.presentation-mode-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.c.presentation-mode-styling)                                                                                           | 示範如何設定簡報模式，此模式會顯示聊天記錄，但不會顯示 [傳送] 方塊，而且會停用調適型卡片的互動功能。                                                                         | [簡報模式示範](https://microsoft.github.io/BotFramework-WebChat/05.c.presentation-mode-styling)                           |
| [`05.d.hide-upload-button-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.d.hide-upload-button-styling)                                                                                         | 示範如何透過樣式設定來隱藏 [檔案上傳] 按鈕。                                                                                                                                                                            | [隱藏上傳按鈕示範](https://microsoft.github.io/BotFramework-WebChat/05.d.hide-upload-button-styling)                         |
| [`06.a.cognitive-services-bing-speech-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.a.cognitive-services-bing-speech-js)                                                                           | 使用 (已過時的) 認知服務 Bing 語音 API 和 JavaScript 來導入語音轉文字和文字轉語音功能。                                                                                                      | [使用 JS 的 Bing 語音示範](https://microsoft.github.io/BotFramework-WebChat/06.a.cognitive-services-bing-speech-js)                 |
| [`06.b.cognitive-services-bing-speech-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.b.cognitive-services-bing-speech-react)                                                                     | 使用 (已過時的) 認知服務 Bing 語音 API 和 React 來導入語音轉文字和文字轉語音功能。                                                                                                           | [使用 React 的 Bing 語音示範](https://microsoft.github.io/BotFramework-WebChat/06.b.cognitive-services-bing-speech-react)           |
| [`06.c.cognitive-services-speech-services-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.c.cognitive-services-speech-services-js)                                                                   | 使用認知服務語音服務 API 來導入語音轉文字和文字轉語音功能。                                                                                                                                  | [使用 JS 的語音服務示範](https://microsoft.github.io/BotFramework-WebChat/06.c.cognitive-services-speech-services-js)         |
| [`06.d.speech-web-browser`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.d.speech-web-browser)                                                                                                         | 示範如何使用網路聊天的瀏覽器型 Web Speech API 來實作文字轉語音。 (在此範例中會連結至 W3C 標準)                                                                                                    | [Web Speech API 示範](https://microsoft.github.io/BotFramework-WebChat/06.d.speech-web-browser)                                     |
| [`06.e.cognitive-services-speech-services-with-lexical-result`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.e.cognitive-services-speech-services-with-lexical-result)                                 | 示範如何使用來自認知服務語音服務 API 的語彙結果。                                                                                                                                                 | [語彙結果示範](https://microsoft.github.io/BotFramework-WebChat/06.e.cognitive-services-speech-services-with-lexical-result) |
| [`06.f.hybrid-speech`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.f.hybrid-speech)                                                                                                                   | 示範如何使用瀏覽器型 Web Speech API 來將語音轉文字，以及使用認知服務語音服務 API 來將文字轉語音。                                                                                        | [混合語音示範](https://microsoft.github.io/BotFramework-WebChat/06.f.hybrid-speech)                                           |
| [`07.a.customization-timestamp-grouping`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.a.customization-timestamp-grouping)                                                                             | 示範如何藉由顯示或隱藏時間戳記並變更依時間的訊息群組，來自訂時間戳記。                                                                                                             | [時間戳記群組示範](https://microsoft.github.io/BotFramework-WebChat/07.a.customization-timestamp-grouping)                   |
| [`07.b.customization-send-typing-indicator`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.b.customization-send-typing-indicator)                                                                       | 示範當使用者開始在 [傳送] 方塊中輸入內容時，要如何傳送所輸入的活動。                                                                                                                                                | [使用者輸入指標示範](https://microsoft.github.io/BotFramework-WebChat/07.b.customization-send-typing-indicator)             |
| [`08.customization-user-highlighting`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)                                                                                   | 示範如何根據訊息是來自使用者還是聊天機器人，來自訂活動的樣式。                                                                                                                      | [使用者醒目提示示範](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting)                       |
| [`09.customization-reaction-buttons`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/09.customization-reaction-buttons/)                                                                                    | 針對專門用於聊天機器人需求的網路聊天，導入為其建立自訂元件的功能。 本教學課程會示範如何將反應 Emoji (例如 :thumbsup: 和 :thumbsdown:) 新增至對話活動。 | [反應按鈕示範](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)                         |
| [`10.a.customization-card-components`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)                                                                                   | 示範如何建立自訂活動卡片附件，在此例為 GitHub 存放庫卡片。                                                                                                                                  | [卡片元件示範](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components)                         |
| [`10.b.customization-password-input`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.b.customization-password-input)                                                                                     | 示範如何建立密碼輸入的自訂活動。                                                                                                                                                                      | [密碼輸入示範](https://microsoft.github.io/BotFramework-WebChat/10.b.customization-password-input)                           |
| [`11.customization-redux-actions`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/11.customization-redux-actions)                                                                                           | 進階教學課程：示範如何藉由透過聊天機器人傳送 redux 動作，將 redux 中介軟體併入網路聊天應用程式。 此範例會示範根據聊天機器人和使用者之間的活動來進行的手動樣式設定。             | [Redux 動作示範](https://microsoft.github.io/BotFramework-WebChat/11.customization-redux-actions)                               |
| [`12.customization-minimizable-web-chat`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/12.customization-minimizable-web-chat)                                                                             | 進階教學課程：示範如何將網路聊天介面新增至網站中作為可最小化的顯示/隱藏聊天方塊。                                                                                                              | [可最小化的網路聊天示範](https://microsoft.github.io/BotFramework-WebChat/12.customization-minimizable-web-chat)                 |
| [`13.customization-speech-ui`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/13.customization-speech-ui)                                                                                                   | 進階教學課程：示範如何完整自訂聊天機器人的重要元件 (在此例為語音)，以完全取代文字型文字記錄 UI，使其改為顯示簡單的語音按鈕與聊天機器人的回應。      | [語音 UI 示範](https://microsoft.github.io/BotFramework-WebChat/13.customization-speech-ui)                                       |
| [`14.customization-piping-to-redux`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/14.customization-piping-to-redux)                                                                                       | 進階教學課程：示範如何使用管線將聊天機器人的活動傳送至您自己的 Redux 存放區，並使用聊天機器人來控制您的逐頁檢視聊天機器人活動與 Redux。                                                                          | [用管線傳送至 Redux 示範](https://microsoft.github.io/BotFramework-WebChat/14.customization-piping-to-redux)                           |
| [`15.a.backchannel-piggyback-on-outgoing-activities`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.a.backchannel-piggyback-on-outgoing-activities)                                                     | 進階教學課程：示範如何將自訂資料新增至每個傳出活動。                                                                                                                                                | [Backchannel 附掛示範](https://microsoft.github.io/BotFramework-WebChat/15.a.backchannel-piggyback-on-outgoing-activities) |
| [`15.b.incoming-activity-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.b.incoming-activity-event)                                                                                               | 進階教學課程：示範如何將所有傳入活動轉送至 JavaScript 事件以便進一步處理。                                                                                                                | [傳入活動示範](https://microsoft.github.io/BotFramework-WebChat/15.b.incoming-activity-event)                             |
| [`15.c.programmatic-post-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.c.programmatic-post-activity)                                                                                         | 進階教學課程：示範如何以程式設計方式傳送訊息。                                                                                                                                                             | [以程式設計方式貼文示範](https://microsoft.github.io/BotFramework-WebChat/15.c.programmatic-post-activity)                       |
| [`15.d.backchannel-send-welcome-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.d.backchannel-send-welcome-event)                                                                                 | 進階教學課程：示範如何傳送歡迎使用事件與用戶端功能，例如瀏覽器語言。                                                                                                                        | [歡迎使用事件示範](https://microsoft.github.io/BotFramework-WebChat/15.d.backchannel-send-welcome-event)                          |
| [`16.customization-selectable-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/16.customization-selectable-activity)                                                                               | 進階教學課程：示範如何將自訂的按一下行為新增至每個活動。                                                                                                                                                  | [可選取的活動示範](https://microsoft.github.io/BotFramework-WebChat/16.customization-selectable-activity)                   |
| [`17.chat-send-history`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/17.chat-send-history)                                                                                                               | 進階教學課程：示範如何儲存使用者輸入，並允許使用者回頭逐一瀏覽先前傳送的訊息。                                                                                                      | [聊天傳送歷程記錄示範](https://microsoft.github.io/BotFramework-WebChat/17.chat-send-history)                                     |
| [`18.customization-open-url`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/18.customization-open-url)                                                                                                     | 進階教學課程：示範如何自訂開啟 URL 的行為。                                                                                                                                                             | [自訂開啟 URL 示範](https://microsoft.github.io/BotFramework-WebChat/18.customization-open-url)                               |
| [`19.a.single-sign-on-for-enterprise-apps`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps)                                                                         | 示範如何使用 OAuth 來對企業應用程式使用單一登入                                                                                                                                                              | [使用 OAuth 進行單一登入示範](https://microsoft.github.io/BotFramework-WebChat/19.a.single-sign-on-for-enterprise-apps)         |
 

## <a name="web-chat-api-reference"></a>網路聊天 API 參考

有幾個屬性可供傳遞至網路聊天 React 元件 (`<ReactWebChat>`) 或 `renderWebChat()` 方法。 請隨意檢查使用 [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378) 來開始的原始程式碼。 以下是可用屬性的簡短說明。

| 屬性                   | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | 仿造 [Redux 中介軟體](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)的中介軟體鏈結，可讓開發人員在活動目前現有的 DOM 上新增 DOM 元件。 中介軟體簽章如下：`options => next => card => children => next(card)(children)`。                                                                                                                                                                                                                                           |
| `activityRenderer`         | `activityMiddleware` 的「扁平化」版本，類似於 Redux 中的[存放區加強程式](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer)概念。                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | 傳入自訂的調適型卡片主機組態。請務必確認主機組態具有所使用調適型卡片的版本。 如需詳細資訊，請參閱[自訂主機組態](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238)。                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | 中介軟體鏈結，可讓開發人員在附件上新增其自有的自訂 HTML 元素。 其簽章如下：`options => next => card => next(card)`。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | `attachmentMiddleware` 的「扁平化」版本。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | 中介軟體鏈結，可讓開發人員修改卡片的動作，例如調適型卡片或建議的動作。 中介軟體簽章如下：`cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | 用於具現化 Direct Line 物件的 Factory 方法。 Azure Government 使用者應該使用 `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });` 來變更端點。 參數的完整清單為：`conversationId`、`domain`、`fetch`、`pollingInterval`、`secret`、`streamUrl`、`token`、`watermark` `webSocket`。                                                                                                                                                                                                                         |
| `createStore`              | 中介軟體鏈結，可讓開發人員修改存放區動作。 中介軟體簽章如下：`createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | 指定 DirectLine 物件與 DirectLine 權杖。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | 停用網路聊天的 UI (也就是針對簡報模式)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | 指定語音 (Bing 語音或認知服務語音服務) 的文法清單。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | 變更時間戳記群組的預設設定。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | 指出網路聊天的預設語言。 強烈建議使用四個字母的代碼 (例如 `en-US`)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | 變更預設的 Markdown 轉譯器物件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | 顯示從使用者到聊天機器人的輸入訊號，以指出使用者並未閒置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | 指定自訂存放區，舉裡來說，以便用於對聊天機器人新增程式化活動。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | 儲存網路聊天樣式自訂值的物件。 如需預設樣式選項的完整清單 (經常更新)，請參閱 [defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js) 檔案。                                                                                                                                                                                                                                                                              |
| `styleSet`                 | 非建議的覆寫樣式方式。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | 指定 userID。 有兩種方式可指定 `userID`：在屬性中，或於產生權杖呼叫時在權杖中 (`createDirectLine()`)。 如果同時使用這兩種方法來指定 userID，則會使用權杖 userID 屬性，且執行階段期間會出現 `console.warn`。 如果透過屬性提供 `userID`，但前面加上 `'dl'`，例如 `'dl_1234'`，則會擲回此值，並產生新的 `ID`。 如果未指定 `userID`，則此屬性會預設為隨機的使用者識別碼。 不建議多個使用者共用相同的使用者識別碼；因為這會共用其使用者狀態。 |
| `username`                 | 指定使用者名稱。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | 指定用於文字轉語音和語音轉文字的 Web Speech 物件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>瀏覽器相容性
網路聊天支援最新 2 個版本的新式瀏覽器，例如 Chrome、Edge 和 FireFox。
如果您在 Internet Explorer 11 中需要網路聊天，請參閱 [ES5 套件組合示範](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)。

不過，請注意：
- 網路聊天不支援比第 11 版還舊的 Internet Explorer
- Internet Explorer 不支援非 ES5 範例所示的自訂。 因為 IE11 並非新式瀏覽器，因此不支援 ES6，許多使用箭號函式和新式承諾的範例必須手動轉換為 ES5。  如果您的應用程式需要大量自訂，強烈建議您開發適用於新式瀏覽器 (如 Google Chrome 或 Edge) 的應用程式。
- 網路聊天沒有打算要支援 IE11 (ES5) 的範例。
   - 如果客戶想要手動重寫我們的其他範例以便在 IE11 中運作，建議其考慮使用 polyfill 和 transpiler (如 [`babel`](https://babeljs.io/docs/en/next/babel-standalone.html)) 將程式碼從 ES6+ 轉換為 ES5。

## <a name="how-to-test-with-web-chats-latest-bits"></a>如何使用網路聊天的最新版本進行測試

*目前只能透過 MyGet 封裝來測試未發行的功能。*

如果您想要測試尚未發行的功能或錯誤修正，請將網路聊天套件指向網路聊天的每日摘要，而不是官方的 npmjs 摘要。

目前，您可以藉由訂閱 MyGet 摘要來存取網路聊天的日報。 若要這樣做，您必須更新專案中的登錄。 **這項變更無法復原，我們的指示中會有如何還原為訂閱正式版本的說明**。

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>在 `myget.org` 訂閱最新版本

若要這樣做，請新增您的套件，然後變更專案的登錄。

1. 新增網路聊天以外的專案相依性。
1. 在專案的根目錄中，建立 `.npmrc` 檔案
1. 在檔案中新增下面這行：`registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`
1. 將網路聊天新增到您的專案相依性 `npm i botframework-webchat --save`
1. 請注意，在 `package-lock.json` 中，登錄現在指向 MyGet。 網路聊天專案已啟用上游來源 Proxy，這會將非 MyGet 套件重新導向至 `npmjs.com`。

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>在 `npmjs.com` 上重新訂閱正式版本
若要重新訂閱，您必須重設登錄。

1. 刪除 `.npmrc file`
1. 刪除根 `package-lock.json`
1. 移除 `node_modules` 目錄
1. 使用 `npm i` 重新安裝套件
1. 請注意，在 `package-lock.json` 中，登錄會再次指向 https://npmjs.com/ 。


## <a name="contributing"></a>參與

如需如何建置專案的詳細資料以及存放庫的提取要求指導方針，請參閱[參與頁面](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md)。

此專案採用 [Microsoft 開放原始碼管理辦法](https://opensource.microsoft.com/codeofconduct/)。
如需詳細資訊，請參閱[管理辦法常見問題集](https://opensource.microsoft.com/codeofconduct/faq/)，如有任何其他問題或意見請連絡 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## <a name="reporting-security-issues"></a>報告安全性問題

如有安全性問題和錯誤 (bug)，請私下透過電子郵件向 Microsoft Security Response Center (MSRC) 報告，信箱為：[secure@microsoft.com](mailto:secure@microsoft.com)。 您應該會在 24 小時內收到回應。 如果因為某些原因而未收到回應，請透過電子郵件來追蹤進展，以確保我們有收到您的原始郵件。 進一步的資訊 (包括 [MSRC PGP](https://technet.microsoft.com/security/dn606155) 機碼) 可於[Security TechCenter](https://technet.microsoft.com/security/default) 中找到。

Copyright (c) Microsoft Corporation。 著作權所有，並保留一切權利。
