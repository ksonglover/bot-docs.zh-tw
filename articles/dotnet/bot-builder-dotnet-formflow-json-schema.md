---
title: 使用 JSON 結構描述和 FormFlow 來定義表單 | Microsoft Docs
description: 了解如何使用 JSON 結構描述和 FormFlow 搭配適用於 .NET 的 Bot Framework SDK 來定義表單。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a29f376afa4a9d3027960407f688cbef76b35473
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298798"
---
# <a name="define-a-form-using-json-schema"></a>使用 JSON 結構描述來定義表單

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

如果您透過 FormFlow 建立 Bot 時使用了 [C# 類別](bot-builder-dotnet-formflow.md#create-class)來定義表單，表單會衍生自 C# 中類型的靜態定義。 或者，您也可以改用 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 結構描述</a> (英文) 來定義表單。 使用 JSON 結構描述所定義的表單完全由料驅動；只要更新結構描述即可變更表單 (因而變更 Bot 的行為)。 

JSON 結構描述會描述 <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> (英文) 內的欄位，且包含控制提示、範本和字詞的註釋。 若要使用 JSON 結構描述搭配 FormFlow，您必須將 `Microsoft.Bot.Builder.FormFlow.Json` NuGet 套件新增至專案中，並匯入 `Microsoft.Bot.Builder.FormFlow.Json` 命名空間。

## <a name="standard-keywords"></a>標準關鍵字 

FormFlow 可支援下列標準 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 結構描述</a> (英文) 關鍵字：

| 關鍵字 | 說明 | 
|----|----|
| type | 會定義欄位包含的資料類型。 |
| 列舉 | 會定義欄位的有效值。 |
| minimum | 定義欄位允許的最小數值 (如 [NumericAttribute][numericAttribute] 中所述)。 |
| maximum | 定義欄位允許的最大數值 (如 [NumericAttribute][numericAttribute] 中所述)。 |
| 必要 | 會定義哪些欄位為必要。 |
| 模式 | 驗證字串值 (如 [PatternAttribute][patternAttribute] 中所述)。 |

## <a name="extensions-to-json-schema"></a>JSON 結構描述延伸模組

FormFlow 可延伸標準 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 結構描述</a> (英文) 以支援數個額外屬性。

### <a name="additional-properties-at-the-root-of-the-schema"></a>位於結構描述根目錄的其他屬性

| 屬性 | 值 |
|----|----|
| OnCompletion | 含引數 `(IDialogContext context, JObject state)` 的 C# 指令碼用來完成表單。 |
| 參考 | 要包含在指令碼中的參考。 例如： `[assemblyReference, ...]` 。 路徑應為絕對或相對於目前的目錄。 根據預設，指令碼會包含 `Microsoft.Bot.Builder.dll`。 |
| 匯入 | 要包含在指令碼中的匯入。 例如： `[import, ...]` 。 根據預設，指令碼會包含 `Microsoft.Bot.Builder`、`Microsoft.Bot.Builder.Dialogs`、`Microsoft.Bot.Builder.FormFlow`、`Microsoft.Bot.Builder.FormFlow.Advanced`、`System.Collections.Generic` 和 `System.Linq` 命名空間。 |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>位於結構描述根目錄，或作為類型屬性同儕節點的其他屬性

| 屬性 | 值 |
|----|----|
| 範本 | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| Prompt | `{ Patterns:[string, ...] <args>}` |

若要在 JSON 結構描述中指定範本和提示，請使用 [TemplateAttribute][templateAttribute] 和 [PromptAttribute][promptAttribute] 所定義的相同詞彙。 結構描述中的屬性名稱和值，應符合基礎 C# 列舉中的屬性名稱和值。 例如，此結構描述程式碼片段定義的範本，會覆寫 `TemplateUsage.NotUnderstood` 範本並指定 `TemplateBaseAttribute.ChoiceStyle`： 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>作為類型屬性同儕節點的其他屬性

|   屬性   |          內容           |                                                   說明                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            布林             |                                  指出欄位是否為 `DateTime` 欄位。                                  |
|   說明   |      字串或物件       |                  如 [DescribeAttribute][describeAttribute] 中所述的欄位說明。                  |
|    詞彙     |       `[string,...]`        |                  如 TermsAttribute 中所述，用於比對欄位值的規則運算式。                  |
|  MaxPhrase   |             int             |                  透過 `Language.GenerateTerms(string, int)` 執行字詞以將其展開。                   |
|    值    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    Active    |           script            | 含引數 `(JObject state)->bool` 的 C# 指令碼用來測試欄位、訊息或確認是否為作用中。  |
|   驗證   |           script            |      含引數 `(JObject state, object value)->ValidateResult` 的 C# 指令碼用來驗證欄位值。      |
|    定義    |           script            |        含引數 `(JObject state, Field<JObject> field)` 的 C# 指令碼用來以動態方式定義欄位。        |
|     下一頁     |           script            | 含引數 `(object value, JObject state)` 的 C# 指令碼用來在填入欄位後判斷下一個步驟。 |
|    先前    |          `[confirm          |                                                  message, ...]`                                                  |
|    後續     |          `[confirm          |                                                  message, ...]`                                                  |
| 相依性 |        [string, ...]        |                           此欄位、訊息或確認所相依的欄位。                           |

透過在 **Before** 屬性或 **After** 屬性值內使用含引數 `(JObject state)` 的 C# 指令碼，或隨機選取的一組模式搭配選用的範本引數，使用 `{Confirm:script|[string, ...], ...templateArgs}` 來定義確認。

透過在 **Before** 屬性或 **After** 屬性值內使用含引數 `(JObject state)` 的 C# 指令碼，或隨機選取的一組模式搭配選用的範本引數，使用 `{Message:script|[string, ...] ...templateArgs}` 來定義訊息。

## <a name="scripts"></a>指令碼

上述屬性中有數個屬性包含指令碼作為屬性值。 指令碼可以是 C# 程式碼的任何程式碼片段，您通常可以在方法的主體中找到。 您可以使用 **References** 屬性和/或 **Imports** 屬性來新增參考。 特殊的全域變數包括：

| 變數 | 說明 |
|----|----|
| 選項 | 指令碼要執行的內部分派。 |
| state | 針對所有指令碼繫結的 `JObject` 表單狀態。 |
| ifield | 針對訊息/確認提示建立器以外的所有指令碼，`IField<JObject>` 允許對目前欄位進行推理。 |
| value | 要為 **Validate** 驗證的物件值。 |
| field | `Field<JObject>` 允許動態更新 **Define** 中的欄位。 |
| context | `IDialogContext` 內容允許將結果張貼在 **OnCompletion** 中。 |

透過 JSON 結構描述定義的欄位具有與任何其他欄位相同的功能，可用程式設計方式延伸或覆寫定義。 也可用相同方式進行當地語系化。

## <a name="json-schema-example"></a>JSON 結構描述範例

定義表單最簡單的方式是直接在 JSON 結構描述中定義所有項目，包括任何 C# 程式碼。 此範例會示範已標註三明治 Bot 的 JSON 結構描述，如[使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)中所述。

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>使用 JSON 結構描述實來作 FormFlow

若要使用 JSON 結構描述實來作 FormFlow，請使用 `FormBuilderJson`，這樣可支援與 `FormBuilder` 相同的 Fluent 介面。 此程式碼範例會示範如何實作已標註三明治 Bot 的 JSON 結構描述，如[使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)中所述。

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他資源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 自訂表單](bot-builder-dotnet-formflow-formbuilder.md)
- [將表單內容當地語系化](bot-builder-dotnet-formflow-localize.md)
- [使用模式語言來自訂使用者體驗](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
