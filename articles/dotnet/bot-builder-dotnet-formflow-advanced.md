---
title: FormFlow 的進階功能 | Microsoft Docs
description: 了解如何使用 FormFlow 和適用於 .NET 的 Bot Framework SDK 來自訂使用者體驗。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d04e13babef847a44438e1a748990d7405478fa2
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225953"
---
# <a name="advanced-features-of-formflow"></a>FormFlow 的進階功能

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[FormFlow 的基本功能](bot-builder-dotnet-formflow.md)描述基本的 FormFlow 實作，可提供相當通用的使用者體驗。 若要使用 FormFlow 來提供更客製化的使用者體驗，您可以指定初始表單狀態；新增商務邏輯，以管理欄位之間的相互依存性並處理使用者輸入；以及使用屬性來自訂提示、覆寫範本、指定選擇性欄位、比對使用者輸入，並驗證使用者輸入。 

## <a name="specify-initial-form-state-and-entities"></a>指定初始表單狀態和實體

當您啟動 [FormDialog][formDialog] 時，可以選擇性地傳遞狀態的執行個體。 如果您將狀態的執行個體傳入，則 FormFlow 依預設會略過任何欄位中已包含值的步驟；系統將不會提示使用者輸入這些欄位。 若要強制表單提示使用者輸入所有欄位 (即使欄位已包含初始狀態中的值)，當您啟動 `FormDialog` 時，請傳入 [FormOptions.PromptFieldsWithValues][promptFieldsWithValues]。 如果欄位包含一個初始值，則提示會使用該值作為預設值。

您也可以傳入 [LUIS](https://luis.ai/) 實體以繫結至狀態。 如果 `EntityRecommendation.Type` 是前往 C# 類別中欄位的路徑，則 `EntityRecommendation.Entity` 會通過要繫結至您欄位的辨識器。 FormFlow 會略過任何繫結至實體欄位的步驟；系統將不會提示使用者輸入這些欄位。 

## <a name="add-business-logic"></a>新增商務邏輯 

若要處理表單欄位之間的相互依存性，或在取得或設定欄位值的程序期間套用特定邏輯，您可以指定驗證函式內的商務邏輯。 驗證函式可讓您管理狀態，並傳回 [ValidateResult][validateResult] 物件，當中可能包含： 

- 意見反應字串，會描述值無效的原因
- 已轉換的值
- 用於釐清值的一組選擇

此程式碼範例會顯示 `Toppings` 欄位的驗證函式。 如果欄位的輸入包含 `ToppingOptions.Everything` 列舉值，則函式可確保 `Toppings` 欄位值包含配料的完整清單。

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

除了驗證函式之外，您還可以新增 [Term](#match-user-input-using-the-terms-attribute) 屬性，以符合 "everything" 或 "not" 等使用者運算式。

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

此程式碼片段會使用如上所示的驗證函式，來顯示當使用者要求 "everything but Jalapenos" 時，Bot 和使用者之間的互動。 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>FormFlow 屬性

您可以將這些 C# 屬性新增至您的類別，來自訂 FormFlow 對話方塊的行為。

| 屬性 | 目的 |
|----|----| 
| [描述][describeAttribute] | 變更欄位或值在範本或卡片中的顯示方式 |
| [數值][numericAttribute] | 限制數值欄位的接受值 |
| [選用][optionalAttribute] | 將欄位標示為選用 |
| [模式][patternAttribute] | 定義規則運算式來驗證字串欄位 |
| [提示][promptAttribute] | 定義欄位的提示 |
| [範本][templateAttribute] | 定義要用來在提示中產生提示或值的範本 |
| [字詞][termsAttribute] | 定義符合欄位或值的輸入字詞 |

## <a name="customize-prompts-using-the-prompt-attribute"></a>使用 Prompt 屬性來自訂提示

系統會針對表單中的每個欄位自動產生預設提示，但您可以使用 `Prompt` 屬性，指定任何欄位的自訂提示。 例如，如果 `SandwichOrder.Sandwich` 欄位的預設提示是「請選取三明治」您可以新增 `Prompt` 屬性來指定該欄位的自訂提示。

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

這個範例會使用[模式語言](bot-builder-dotnet-formflow-pattern-language.md)，在執行階段以動態方式將表單資料填入提示：`{&}` 會取代為欄位的描述，而 `{||}` 會取代為列舉中的選擇清單。 

> [!NOTE]
> 根據預設，欄位的描述會產生自欄位的名稱。 若要指定欄位的自訂描述，請新增 `Describe` 屬性。

此程式碼片段會顯示上述範例中所指定的自訂提示。

```console
What kind of sandwich would you like?
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

`Prompt` 屬性也可以指定影響表單顯示提示方式的參數。 例如，`ChoiceFormat` 參數會決定表單轉譯選擇清單的方式。

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

在此範例中，`ChoiceFormat` 參數的值表示選擇應該顯示為項目符號清單 (而不是已編號的清單)。

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>使用 Template 屬性自訂提示

`Prompt` 屬性可讓您自訂單一欄位的提示，而 `Template` 屬性可讓您取代 FormFlow 用來自動產生提示的預設範本。 此程式碼範例會使用 `Template` 屬性，重新定義表單處理所有列舉欄位的方式。 此屬性會指出使用者只能選取其中一個項目、使用[模式語言](bot-builder-dotnet-formflow-pattern-language.md)來設定提示的文字，並指定表單每一行應該只顯示一個項目。 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

此程式碼片段會顯示 `Bread` 欄位和 `Cheese` 欄位的結果提示。

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

如果您使用 `Template` 屬性來取代 FormFlow 用來產生提示的預設範本，可能需要將一些變化插入表單產生的提示和訊息。 若要這樣做，您可以使用[模式語言](bot-builder-dotnet-formflow-pattern-language.md)來定義多個文字字串，每當表單必須顯示提示或訊息時，就會隨機從可用的選項中進行選擇。

此程式碼範例會重新定義 [TemplateUsage.NotUnderstood][notUnderstood] 範本，以指定兩個不同變化的訊息。 當 Bot 因不了解使用者的輸入而需要通訊時，會從兩個文字字串中隨機選取一個來判斷訊息內容。 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

此程式碼片段會顯示 Bot 與使用者之間產生互動的範例。 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>使用 Optional 屬性將欄位指定為選擇性

若要將欄位指定為選擇性，請使用 `Optional` 屬性。 這個程式碼範例會指定 `Cheese` 欄位是選擇性的。

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

如果欄位是選擇性的，且未指定任何值，則目前的選擇會顯示為「無喜好設定」。

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

如果欄位是選擇性的，且使用者已指定值，則「無喜好設定」會顯示為清單中的最後一個選擇。

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>使用 Terms 屬性來比對使用者輸入

當使用者將訊息傳送至使用 FormFlow 建置的 Bot 時，Bot 就會比對輸入與字詞清單，以嘗試識別使用者輸入的意義。 根據預設，字詞清單是透過將這些步驟套用至欄位或值所產生的： 

1. 在出現大小寫變更和底線 (_) 時中斷。
2. 產生每個 <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-gram</a> 直到長度上限。
3. 將 "s?" 新增 到每個字的結尾 (以支援複數)。 

例如，"AngusBeefAndGarlicPizza" 值會產生下列字詞： 

- 'angus?'
- 'beefs?'
- 'garlics?'
- 'pizzas?'
- 'angus? beefs?'
- 'garlics? pizzas?'
- 'angus beef and garlic pizza'

若要覆寫此預設行為，並定義字詞清單，用來比對使用者輸入與欄位或欄位中的值，請使用 `Terms` 屬性。 例如，您可以使用 `Terms` 屬性 (與規則運算式) 來說明使用者很可能將 "rotisserie" 這個單字拼錯的事實。 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

您可以藉由使用 `Terms` 屬性，來增加使用者輸入符合其中一個有效選項的可能性。 在此範例中的 `Terms.MaxPhrase` 參數會使 `Language.GenerateTerms` 產生字詞的其他變化。 

當使用者拼錯 "Rotisserie" 時，此程式碼片段會顯示 Bot 與使用者之間產生的互動。

```console
What kind of sandwich would you like?
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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>使用 Numeric 屬性或 Pattern 屬性來驗證使用者輸入

若要限制數值欄位的允許值範圍，請使用 `Numeric` 屬性。 此程式碼範例會使用 `Numeric` 屬性，來指定 `Rating` 欄位的輸入必須是介於 1 到 5 之間的數字。 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

若要指定特定欄位值所需的格式，請使用 `Pattern` 屬性。 此程式碼範例會使用 `Pattern` 屬性，來指定 `PhoneNumber` 欄位值所需的格式。

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>總結

本文已說明如何透過指定初始表單狀態；新增商務邏輯，以管理欄位之間的相互依存性並處理使用者輸入；以及使用屬性來自訂提示、覆寫範本、指定選擇性欄位、比對使用者輸入，並驗證使用者輸入，使用 FormFlow 來提供客製化的使用者體驗。 如需使用 FormFlow 來自訂使用者體驗的其他方式相關資訊，請參閱[使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)。

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他資源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)
- [將表單內容當地語系化](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 結構描述來定義表單](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式語言來自訂使用者體驗](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
