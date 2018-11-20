---
redirect_url: /bot-framework/bot-builder-send-welcome-message
ms.openlocfilehash: ef281fd1b6539484c06f68caffbd4e87ec8acc2b
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332902"
---
<a name="--"></a><!--
---
標題：歡迎使用者 | Microsoft Docs 描述：了解如何設計 Bot，以提供親切的使用者體驗。
關鍵字：概觀, 設計, 使用者體驗, 歡迎, 個人化體驗 作者：dashel ms.author: dashel manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 8/30/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="welcoming-the-user"></a>歡迎使用者

建立任何 Bot 時的主要目標就是讓您的使用者參與有意義的對話。 達成此目標的最佳方式之一就是確保從使用者第一次連線的那一刻，他們就了解您 Bot 的主要用途和功能，以及您 Bot 的建立原因。

## <a name="show-your-purpose"></a>顯示您的目的

想像一下以下的使用者體驗：使用者連線到標示「芝加哥觀光資訊」的 Bot，並希望在芝加哥找到旅館資訊。 他們根本不知道，您的 Bot 實際上適合於芝加哥式披薩愛好者，只提供餐廳秘訣。 一旦明白他們的問題並未得到正確回答，這位使用者會快速退出，並且認為您的網站提供了不良的線上體驗。 以足夠的資訊來歡迎您的使用者，讓他們了解您 Bot 的主要用途和功能，即可避免這類不良的使用者體驗。 

![歡迎訊息](./media/welcome_message.png)

讀取您的歡迎訊息時，如果 Bot 並未提供使用者所需的資訊類型，他們即可在經歷令人沮喪的互動前快速地繼續前進。
每當使用者第一次與您的 Bot 互動時，應該就會產生歡迎訊息。 若要達成此目的，您可以監視 Bot 的 [活動] 類型並監看新的連線。 視通道而定，每個新連線都可以產生最多兩個對話更新活動。

- 當使用者的 Bot 連線到對話時產生一個。
- 當使用者加入對話時產生一個。

每當偵測到新的對話更新時，只會產生歡迎訊息，但是透過各種通道存取您的 Bot 時，可能會導致無法預期的結果。

## <a name="design-for-different-channels"></a>針對不同頻道進行設計

雖然您可完全控制您的 Bot 所呈現的資訊，但您可能無法控制不同通道實作該資訊的呈現方式。 當使用者最初連結至該通道時，有些通道會建立一個對話更新，而只有在收到來自使用者的初始輸入訊息之後，才會建立不同的對話更新。 當使用者最初連線至通道時，其他通道會產生這兩個活動。 如果您只監看對話更新事件，並且在具有兩個對話更新活動的通道上顯示歡迎訊息，使用者可能會收到下列訊息：

![雙重歡迎訊息](./media/double_welcome_message.png)

只針對第二個對話更新事件，產生初始歡迎訊息，可以避免此重複訊息。 在下列兩種情況下，可以偵測到第二個事件：
- 已發生對話更新事件。
- 新的成員 (使用者) 已加入對話。

此外，務必考量使用者的輸入何時實際上可能包含實用資訊，而這也可能因通道而異。 如果通道在 Bot 初始連線時產生這兩個對話更新活動，則可將使用者的第一次輸入成功地評估為您的歡迎訊息提示的答案，如下所示的交談。

![單一輸入回應](./media/single_input_response.png)

不過，如果通道先等候初始使用者輸入，再產生第二個對話更新事件，則上面使用的完全相同程式碼會改為產生以下使用者體驗。

![單一輸入錯誤回應](./media/single_input_wrong_response.png)

若要確保使用者在所有可能的通道上擁有良好的體驗，最佳做法就是不預期使用者的初始對話輸入中有實用資訊。 相反地，將初始輸入視為「捨棄」資料，而在接收時，提示使用者提供繼續對話所需的資訊。 不論最初用來存取 Bot 的通道為何，這項技術都會產生一致的使用者體驗。

![捨棄第一次輸入](./media/no_first_input_response.png)

## <a name="personalize-the-user-experience"></a>將使用者體驗個人化

持續提示使用者提供他們已經提供的資訊，會讓他們覺得不受歡迎且不近人情。 如果 Bot 為先前造訪過的使用者保存資訊，先檢查此資訊，而如果適用，使用您在使用者前次造訪時儲存的名稱來歡迎他們是個不錯的做法。 

![歡迎回來訊息](./media/welcome_back.png)

在此情況下，您的對話邏輯會略過提示輸入名稱，並繼續進行下一個對話活動。

您的 Bot 也可以透過即時回應非預期的使用者輸入 (例如終止對話的要求)，將使用者的體驗個人化。

![回應結束要求](./media/respond_to_exit.png)

讓您的互動保持即時對話，可協助使用者體驗更親切且愉悅的 Bot 互動。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [將歡迎訊息傳送給使用者](bot-builder-send-welcome-message.md)

-->
