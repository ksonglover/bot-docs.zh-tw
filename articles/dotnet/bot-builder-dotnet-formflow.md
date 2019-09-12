---
title: FormFlow 的基本功能 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK 中的 FormFlow 引導交談流程。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4e9aa1b7bffd55518bd4ef03512d873ac48b49a7
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297872"
---
# <a name="basic-features-of-formflow"></a>FormFlow 的基本功能

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[交談方塊](bot-builder-dotnet-dialogs.md)功能強大且彈性，處理訂購三明治等引導式交談時可能會很費工。 在交談中的每個時間點，接下來有可能會發生許多不同的情況。 例如，您可能需要釐清問題、提供協助、返回或顯示進度。 使用適用於 .NET 的 Bot Framework SDK 中的 **FormFlow**，即可大幅簡化管理這類引導式交談的程序。 

FormFlow 可根據您指定的指導方針，自動產生管理引導式交談時必須使用的交談方塊。 雖然使用 FormFlow 會讓您無法建立並管理專屬交談方塊，因而犧牲部分彈性；但使用 FormFlow 設計引導式交談可大幅減少您自己開發 Bot 的時間。 此外，您也可使用 FormFlow 產生的對話方塊及其他類型對話方塊組合，建構自己的 Bot。 例如，FormFlow 對話方塊可引導使用者完成表單填寫的整個程序，而 [LuisDialog][LuisDialog] 可評估使用者輸入內容以判斷其意圖。

本文說明如何建立使用 FormFlow 基本功能的 Bot，以向使用者收集資訊。

## <a id="forms-and-fields"></a>表單和欄位

若要使用 FormFlow 建立 Bot，您必須指定 Bot 要向使用者收集的資訊。 例如，如果 Bot 的目標是取得使用者的三明治訂單，則您定義的表單中必須包含可讓 Bot 填寫訂單之資料的欄位。 您可建立 C# 類別，其中包含一或多個公用屬性，代表 Bot 將向使用者收集的資料。 每個屬性都必須屬於以下資料類型之一：

- 整數 (sbyte、byte、short、ushort、int、uint、long、ulong)
- 浮點數 (float、double)
- 字串
- Datetime
- 列舉
- 列舉的清單

任何資料類型皆可為 Null，您可在沒有值之欄位的模型中使用。 如果表單欄位是以不得為 Null 的列舉屬性為基礎，則列舉中的值 **0** 即代表 **null** (即指出該欄位沒有值)，而您應從 **1** 開始列舉值。 FormFlow 會忽略所有其他屬性類型及方法。

針對複雜物件，您必須為最上層 C# 類別建立表單，再為複雜物件建立另一份表單。 您可使用一般[對話方塊](bot-builder-dotnet-dialogs.md)語意，將表單合併在一起。 您亦可實作 [Advanced.IField][iField] 或使用 [Advanced.Field][field]，並在其中填入字典，藉以直接定義表單。 

> [!NOTE]
> 您可使用 C# 類別或 JSON 結構描述定義表單。 本文說明如何使用 C# 類別定義表單。 如需有關使用 JSON 結構描述的詳細資訊，請參閱[使用 JSON 結構描述定義表單](bot-builder-dotnet-formflow-json-schema.md)。

## <a name="simple-sandwich-bot"></a>範例三明治 Bot

請參考此三明治 Bot 範例，其旨在取得使用者的三明治訂單。 

### <a id="create-class"></a> 建立表單

`SandwichOrder` 類別可定義表單，而列舉則定義建置三明治的選項。 此類別亦包含靜態 `BuildForm` 方法，採用 [FormBuilder][formBuilder] 建立表單，並定義簡單的歡迎訊息。 

若要使用 FormFlow，您必須先匯入 `Microsoft.Bot.Builder.FormFlow` 命名空間。

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>將表單連接至架構 

若要將表單連接至架構，您必須將其新增至控制器。 在此範例中，`Conversation.SendAsync` 方法會呼叫靜態的 `MakeRootDialog` 方法，而後者則會呼叫 `FormDialog.FromForm` 方法以建立 `SandwichOrder` 表單。 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>了解其運作方式

只要以 C# 類別定義表單，並將其連接至架構，即可啟用 FormFlow 自動管理 Bot 和使用者之間的交談。 下列範例互動示範了 FormFlow 基本功能建立的  Bot 功能。 在每個互動中， **>** 符號皆指出使用者輸入回應的時間點。 

#### <a name="display-the-first-prompt"></a>顯示第一個提示 

此表單會填入 `SandwichOrder.Sandwich` 屬性。 表單將自動產生提示「請選取三明治」，而提示中的「三明治」一詞是從屬性名稱 `Sandwich` 衍生而來。 `SandwichOptions` 列舉可定義向使用者顯示的選擇，每個列舉值都可自動根據大小寫和底線文字細分變更。

```console
Please select a sandwich
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

#### <a name="provide-guidance"></a>提供指引

使用者可在交談中任何時間點輸入「協助」，以取得填寫表單的指引。 例如，如果使用者在三明治提示中輸入「協助」，則 Bot 將在回應中提供此指引。 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>繼續下一個提示

如果使用者在回應第一個三明治提示時輸入「2」，Bot 可接著顯示下個屬性提示，其由此表單定義：`SandwichOrder.Length`。

```console
Please select a sandwich
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>返回上一個提示 

如果使用者在交談中的此時間點輸入「返回」，則 Bot 將返回上一個提示。 提示會顯示使用者目前的選項 (「黑森林火腿」)；使用者可輸入其他數目以修改選項，或輸入「C」確認選取項。

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>釐清使用者輸入值

如果使用者以文字回應 (而非用數字) 並指出選項，則 Bot 可進一步釐清文字，自動詢問使用者其輸入的內容是否包含其他選項。 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

如果使用者輸入值未直接符合任何有效選項，Bot 可自動提示使用者確認。

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

如果使用者輸入值指出某屬性的多個選擇，而 Bot 無法理解其中任何指定選項，此時就會自動提示使用者以進一步釐清。

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>顯示目前狀態

如果使用者在訂單中的任何時間點輸入「狀態」，則 Bot 的回應將指出已指定和待指定的值。 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>確認選取項目

使用者完成表單後，Bot 便會要求使用者確認其選取項目。

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

如果使用者在回應中輸入「否」，則 Bot 便會讓使用者修改先前的選取項目。 如果使用者在回應中輸入「是」，則系統將視為完成表單，控制項隨即傳回至呼叫對話方塊。 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>處理結束和例外狀況

如果使用者在表單中輸入「結束」，或交談中任何時間點發生例外狀況，則 Bot 必須知道事件發生於哪個步驟、事件發生時的表單狀態，以及事件發生前使用者完成了表單的哪些步驟。 表單會透過 `FormCanceledException<T>` 類別傳回此資訊。 

此程式碼範例會顯示如何快取例外狀況，並根據發生的事件顯示訊息。 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>總結

本文說明了如何使用 FormFlow 的基本功能建立可執行以下工作的 Bot：

- 自動產生並管理交談
- 提供清楚的指引和協助
- 了解數字和文字項目
- 向使用者提供意見反應，說明目前已理解和待釐清的資訊 
- 必要時詢問釐清問題 
- 允許使用者在步驟之間往返修改

雖然在某些情況下，基本的 FormFlow 功能已足夠，但您仍應考量在 Bot 中整合其他進階功能的潛在好處。 如需詳細資訊，請參閱 [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)和[使用 FormBuilder 自訂表單](bot-builder-dotnet-formflow-formbuilder.md)。

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>後續步驟

FormFlow 簡化對話方塊的開發作業。 FormFlow 的進階功能可讓您自訂 FormFlow 物件的運作方式。

> [!div class="nextstepaction"]
> [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>其他資源

- [使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)
- [將表單內容當地語系化](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 結構描述來定義表單](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式語言來自訂使用者體驗](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
