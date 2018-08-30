---
title: 使用模式語言自訂使用者體驗 | Microsoft Docs
description: 了解如何搭配適用於 .NET 的 Bot Builder SDK 使用模式語言，來自訂 FormFlow 提示和覆寫 FormFlow 範本。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a192b69b2ffbac428d80b2fe7c3fd9180caacd4f
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905580"
---
# <a name="customize-user-experience-with-pattern-language"></a>使用模式語言自訂使用者體驗

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

當您自訂提示或覆寫預設範本時，可以使用模式語言來指定提示的內容和/或格式。 

## <a name="prompts-and-templates"></a>提示與範本

[提示][promptAttribute]會定義要傳送給使用者以要求某部分資訊或要求確認的訊息。 您可以使用 [Prompt 屬性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute)或隱含地透過 [IFormBuilder<T>.Field][field] 來自訂提示。 

表單會使用範本，自動建構提示以及其他項目 (例如說明)。 您可以使用 [Template 屬性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute)來覆寫類別或欄位的預設範本。 

> [!TIP]
> [FormConfiguration.Templates][formConfiguration] 類別會定義一組內建範本，以提供如何使用模式語言的良好範例。

## <a name="elements-of-pattern-language"></a>模式語言的元素

模式語言會使用大括號 (`{}`) 來識別將在執行階段以實際值取代的元素。 下表列出模式語言的元素。

| 元素 | 說明 |
|----|----|
| `{<format>}` | 顯示目前欄位 (要套用屬性的欄位) 的值。 |
| `{&}` | 顯示目前欄位的描述 (除非另有指定，否則，這是欄位的名稱)。 |
| `{<field><format>}` | 顯示具名欄位的值。 | 
| `{&<field>}` | 顯示具名欄位的描述。 |
| <code>{&#124;&#124;}</code> | 顯示目前的選項，這可能是欄位的目前值、「無喜好設定」或列舉的值。 |
| `{[{<field><format>} ...]}` | 使用[分隔符號][separator]和 [LastSeparator][lastSeparator] 來分隔清單中的個別值，以顯示具名欄位的值清單。 |
| `{*}` | 每一行均顯示一個作用中欄位；每一行都會包含欄位描述和目前的值。 | 
| `{*filled}` | 每一行均顯示一個包含實際值的作用中欄位；每一行都會包含欄位描述和目前的值。 |
| `{<nth><format>}` | 一般 C# 格式規範，適用於範本的第 n 個引數。 如需可用的引數清單，請參閱 [TemplateUsage][templateUsage]。 |
| `{?<textOrPatternElement>...}` | 條件式替代。 如果所有參考到的模式元素都有值，則會替代值並使用整個運算式。 |

對於上方所列的元素：

- `<field>` 預留位置是您表單類別中的欄位名稱。 例如，如果您的類別包含名稱為 `Size` 的欄位，您可以指定`{Size}` 作為模式元素。 

- 模式元素內的省略符號 (`"..."`) 指出元素可能包含多個值。

- `<format>` 預留位置是一個 C# 格式規範。 例如，如果類別包含名稱為 `Rating` 且類型為 `double` 的欄位，則您可以指定 `{Rating:F2}` 作為模式元素來顯示兩位數的精確度。

- `<nth>` 預留位置會參考範本的第 n 個引數。

### <a name="pattern-language-within-a-prompt-attribute"></a>Prompt 屬性中的模式語言

這個範例會使用 `{&}` 元素來顯示 `Sandwich` 欄位的描述，以及使用 `{||}` 元素來顯示該欄位的選項清單。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

以下是輸出提示：

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

## <a name="formatting-parameters"></a>格式化參數

提示和範本均支援這些格式化參數。

| 使用量 | 說明 |
|----|----|
| `AllowDefault` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷表單是否應該將欄位的目前值顯示為可能的選項。 若為 `true`，即會將目前的值顯示為可能的值。 預設值為 `true`。 |
| `ChoiceCase` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷是否要將每個選項的文字正規化 (例如，每個字組的第一個字母是否為大寫)。 預設值為 `CaseNormalization.None`。 如需可能的值，請參閱 [CaseNormalization][caseNormalization]。 |
| `ChoiceFormat` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷是否要將選項清單顯示為編號清單或項目符號清單。 如果要顯示為編號清單，請將 `ChoiceFormat` 設定為 `{0}` (預設值)。 如果要顯示為項目符號清單，則將 `ChoiceFormat` 設定為 `{1}`。 |
| `ChoiceLastSeparator` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷內嵌的選項清單是否要在最後一個選項之前包括分隔符號。 |
| `ChoiceParens` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷內嵌的選項清單是否要顯示於括號內。 若為 `true`，選項清單就會顯示於括號內。 預設值為 `true`。 |
| `ChoiceSeparator` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷內嵌的選項清單是否要在每個選項 (最後一個選項除外) 之前包括分隔符號。 | 
| `ChoiceStyle` | 適用於 <code>{&#124;&#124;}</code> 模式元素。 判斷選項清單是否要以內嵌方式顯示或每一行顯示。 預設值是 `ChoiceStyleOptions.Auto`，其會判斷在執行階段，是否要以內嵌方式或在清單中顯示選項。 如需可能的值，請參閱 [ChoiceStyleOptions][choiceStyleOptions]。 |
| `Feedback` | 僅適用於提示。 判斷表單是否會回應使用者的選擇，以指出表單已了解該選項。 預設值是 `FeedbackOptions.Auto`，只有在不了解某部分的使用者輸入時才會加以回應。 如需可能的值，請參閱 [FeedbackOptions][feedbackOptions]。 |
| `FieldCase` | 判斷是否要將欄位描述的文字正規化 (例如，每個字組的第一個字母是否為大寫)。 預設值是 `CaseNormalization.Lower`，會將描述轉換成小寫。 如需可能的值，請參閱 [CaseNormalization][caseNormalization]。 |
| `LastSeparator` | 適用於 `{[]}` 模式元素。 判斷項目陣列是否要在最後一個項目之前包括分隔符號。 |
| `Separator` | 適用於 `{[]}` 模式元素。 判斷項目陣列是否要在陣列中的每個項目 (最後一個項目除外) 之前包括分隔符號。 |
| `ValueCase` | 判斷是否要將欄位值的文字正規化 (例如，每個字組的第一個字母是否為大寫)。 預設值是 `CaseNormalization.InitialUpper`，其會將每個字組的第一個字母轉換成大寫。 如需可能的值，請參閱 [CaseNormalization][caseNormalization]。 |

### <a name="prompt-attribute-with-formatting-parameter"></a>具有格式化參數的 Prompt 屬性 

這個範例會使用 `ChoiceFormat` 參數，來指定應該將選項清單顯示為項目符號清單。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

以下是輸出提示：

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他資源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 自訂表單](bot-builder-dotnet-formflow-formbuilder.md)
- [將表單內容當地語系化](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 結構描述定義表單](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Builder SDK 參考</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
