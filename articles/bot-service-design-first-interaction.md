---
title: 設計 Bot 的首次使用者互動 | Microsoft Docs
description: 了解如何提供絕佳的首次使用者體驗，以及如何設計 Bot 以取得成功。
keywords: first impression, beginning, language vs menu, 第一印象, 開始, 語言與功能表
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f59a7acbdb7d580aebeef6ffe81d8b15505aed90
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999805"
---
# <a name="design-a-bots-first-user-interaction"></a>設計 Bot 的首次使用者互動

## <a name="first-impressions-matter"></a>第一印象非常重要

使用者與 Bot 之間的首次互動，對於使用者體驗而言是非常重要的。 設計 Bot 時，請記得 Bot 對使用者的第一則訊息不應該僅僅是一句「您好」而已。 在建置應用程式時，您應該將第一個畫面設計成能夠提供重要的[瀏覽](bot-service-design-navigation.md)提示。使用者應該要能直覺地了解像是功能表的所在位置及其運作方式、應該去哪裡尋求協助、隱私權原則的內容為何之類的項目。 當您設計 Bot 時，使用者與 Bot 之間的首次互動應該要能提供上述類型的資訊。 

## <a name="language-versus-menus"></a>語言與功能表 

請考量下列兩種設計：

### <a name="design-1"></a>設計 1

![Bot](~/media/bot-service-design-first-interaction/hello1.png)


### <a name="design-2"></a>設計 2

![Bot](~/media/bot-service-design-first-interaction/hello2.png)

一開始，讓 Bot 詢問像是 "How can I help you?" (我可以如何協助您？) 這樣的開放式問題 通常是不建議的。 假設您的 Bot 能夠執行一百種不同的工作，使用者應該不太可能自行猜出大部分的工作。 這是因為您的 Bot 並沒有告訴使用者它可以做什麼，所以他們怎麼可能會知道呢？

功能表能為此問題提供簡單的解決方案。 首先，透過列出可用的選項，您的 Bot 便能將自己的功能傳達給使用者。 再來，功能表可讓使用者無需自行輸入太多內容，而只需要按一下即可。 最後，使用功能表將能縮小 Bot 可從使用者接收之輸入的範圍，並大幅簡化您的自然語言模型。 

> [!TIP]
> 功能表對於設計能提供絕佳使用者體驗的 Bot 來說，是一個非常重要的工具，因此請不要因為它們「不夠聰明」而棄之不用。 您依舊可以在搭配功能表設計 Bot 的情況下支援自由格式輸入。 如果使用者回應初始功能表的方式是透過輸入，而非選取功能表項目，您的 Bot 可以嘗試剖析使用者的文字輸入。 

或者，在 Bot 具有特定功能的情況下，您可以詢問更具針對性的問題來引導使用者。 例如，如果您的 Bot 是負責接收三明治訂單，您的第一個互動便可以是「您好！ 我可以幫您點三明治。 請問您想要哪一種麵包？ 我們有白麵包、全麥麵包或黑麵包。」 如此一來，使用者便能透過交談中所提供的瀏覽提示了解回應的方式。

## <a name="other-considerations"></a>其他考量

除了提供直覺且能輕鬆瀏覽的首次互動之外，一個設計完善的 Bot 也應該讓使用者能輕鬆存取其隱私權原則和使用規定。 

> [!TIP]
> 如果您的 Bot 會收集使用者的個人資料，請務必將此作法告知使用者，並描述該資料的後續使用方式。

## <a name="next-steps"></a>後續步驟

您已熟悉設計使用者與 Bot 之間首次互動的幾個基本原則，請繼續深入了解[設計交談流程](~/bot-service-design-conversation-flow.md)。
