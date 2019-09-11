---
title: 網路聊天自訂 | Microsoft Docs
description: 了解如何自訂 Bot Framework 網路聊天。
keywords: bot framework、網路聊天, 聊天, 範例, react, 參考
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: 9e61e7e9d7bc6ba06d4c239cbf8b0dc0c4e7e16e
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299090"
---
# <a name="web-chat-customization"></a>網路聊天自訂

本文詳述如何自訂網路聊天範例以符合您的聊天機器人。

## <a name="integrate-web-chat-into-your-website"></a>將網路聊天整合到您的網站

請遵循[概觀](bot-builder-webchat-overview.md)頁面上的指示，將網路聊天控制項整合到您的網站。

## <a name="customizing-styles"></a>自訂樣式

最新版的網路聊天控制項提供豐富的自訂選項：您可以變更色彩、大小、元素位置、新增自訂元素，以及與裝載的網頁互動。 下面幾個範例會示範如何自訂網路聊天 UI 的這些元素。

您可以在 [`defaultStyleOptions.js` 檔案](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js)上找到可於網路聊天中輕鬆修改的完整設定清單。

這些設定會產生_樣式集_，這是使用 [glamor](https://github.com/threepointone/glamor) 所增強的一組 CSS 規則。 您可以在 [`createStyleSet.js` 檔案](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js)上找到樣式集內所產生的 CSS 樣式完整清單。

## <a name="set-the-size-of-the-web-chat-container"></a>設定網路聊天容器的大小

現在，您可以使用 `styleSetOptions` 調整網路聊天容器的大小。 下列範例具有 `paleturquoise` 的 `body` 背景色彩 (背景為白色的部分)，其可用來顯示網路聊天容器。

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

結果如下︰

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>變更字型或色彩

您不必使用預設背景色彩和聊天泡泡內部所用的字型，而是可以自訂這些項目以符合目標網頁的樣式。 下列程式碼片段可讓您變更來自使用者和聊天機器人的訊息背景色彩。

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

如果您需要進行一些簡單的樣式設定，則可以透過 `styleOptions` 來設定。 樣式選項是一組可直接修改的預先定義樣式，而且網路聊天會據此來計算整個樣式表。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>手動變更 CSS

除了色彩，您還可以修改用來呈現訊息的字型：

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

若要進行更深入的樣式設定，您也可以藉由直接設定 CSS 規則來手動修改樣式集。

> CSS 規則會與 DOM 樹狀結構緊密結合，因此這些規則可能需要更新才能與較新版的網路聊天搭配運作。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>變更對話方塊內的聊天機器人虛擬人偶

最新版的網路聊天支援虛擬人偶，您可以使用 `botAvatarInitials` 和 `userAvatarInitials` 屬性來加以自訂。

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

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
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing avatar initials when rendering Web Chat
               botAvatarInitials: 'BF',
               userAvatarInitials: 'WC'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

在 `renderWebChat` 程式碼內，我們新增了 `botAvatarInitials` 和 `userAvatarInitials`：

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials` 會設定左側虛擬人偶內的文字。 如果將文字設定為假的值，則會隱藏聊天機器人這端的虛擬人偶。 相反地，`userAvatarInitials` 會設定右側的虛擬人偶文字。

## <a name="custom-rendering-activity-or-attachment"></a>自訂呈現活動或附件

在使用最新版網路聊天時，您也可以呈現網路聊天未原生支援的活動或附件。 活動和附件會透過仿照 [Redux 中介軟體](https://redux.js.org/api/applymiddleware)的可自訂管線來傳送。 管線有足夠的彈性，因此您可以輕鬆執行下列工作：

-  裝飾現有活動/附件
-  新增活動/附件
-  取代現有活動/附件 (或加以移除)
-  將中介軟體連接形成菊輪鍊

### <a name="show-github-repository-as-an-attachment"></a>將 GitHub 存放庫顯示為附件

如果您想要顯示一疊 GitHub 存放庫卡片，則可以為 GitHub 存放庫建立新的 React 元件，並將其新增為附件的中介軟體。

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

完整範例可在[自訂卡片元件範例](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)中找到。

在此範例中，我們會新增稱為 `GitHubRepositoryAttachment` 的新 React 元件：

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

接著，我們會建立中介軟體，以在聊天機器人傳送內容類型為 `application/vnd.microsoft.botframework.samples.github-repository` 的附件時，呈現新的 React 元件。 否則，其會藉由呼叫 `next(card)` 在中介軟體上繼續進行。

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

聊天機器人所傳來的活動看起來如下所示：

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
