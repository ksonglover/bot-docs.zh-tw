---
title: 使用 FormBuilder 來自訂表單 | Microsoft Docs
description: 了解如何針對適用於 .NET 的 Bot 建立器 SDK 使用 FormBuilder，以動態方式變更及自訂對話流程和內容。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c4e60f76ecebfa01664500b8343d60ccff0064c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997695"
---
# <a name="customize-a-form-using-formbuilder"></a>使用 FormBuilder 來自訂表單

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[FormFlow 的基本功能](bot-builder-dotnet-formflow.md)描述基本的 FormFlow 實作，可提供相當通用的使用者體驗，而 [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)則描述如何使用商務邏輯和屬性來自訂使用者體驗。 本文說明如何藉由指定表單執行步驟的序列，並以動態方式定義欄位值、確認和訊息，使用 [FormBuilder][formBuilder] 進一步自訂使用者體驗。 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>以動態方式定義欄位值、確認和訊息

您可以使用 FormBuilder，以動態方式定義欄位值、確認和訊息。

### <a name="dynamically-define-field-values"></a>以動態方式定義欄位值 

三明治 Bot 的設計宗旨是將免費飲料或餅乾加入指定長條三明治的任何訂單中，會使用 `Sandwich.Specials` 欄位來儲存可用項目的資料。 在此案例中，`Sandwich.Specials` 欄位的值必須根據訂單是否包含長條三明治，以動態方式為每筆訂單進行設定。 

`Specials` 欄位已指定為選用，且代表無偏好選擇的文字已指定為「無」。

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

此程式碼範例會示範如何以動態方式設定 `Specials` 欄位的值。 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

在此範例中，[Advanced.Field.SetType][setType] 方法會指定欄位類型 (`null` 表示列舉欄位)。 [Advanced.Field.SetActive][setActive] 方法可指定只有在三明治長度為 `Length.FootLong` 時，才會啟用欄位。 最後，[Advanced.Field.SetDefine][setDefine] 方法會指定可定義欄位的非同步委派。 委派已傳遞至目前的狀態物件和以動態方式定義的 [Advanced.Field][field]。 此委派會使用欄位的 Fluent 方法，以動態方式定義值。 在此範例中，值為字串，且 `AddDescription` 和 `AddTerms` 方法會指定每個值的說明和字詞。

> [!NOTE]
> 若要以動態方式定義欄位值，您可以自己實作 [Advanced.IField][iField]，或使用 [Advanced.FieldReflector][FieldReflector] 類別來簡化程序，如上述範例所示。 

### <a name="dynamically-define-messages-and-confirmations"></a>以動態方式定義訊息和確認

您也可以使用 FormBuilder 以動態方式定義訊息和確認。 只有在表單中先前的步驟為非使用中或已完成時，才會執行每個訊息和確認。 

此程式碼範例會示範動態產生的確認如何計算三明治成本。 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>使用 FormBuilder 來自訂表單

此程式碼範例會使用 FormBuilder 來定義表單步驟、[驗證選取項目](bot-builder-dotnet-formflow-advanced.md#add-business-logic)，並[以動態方式定義欄位值和確認](#dynamically-define-field-values-confirmations-and-messages)。 根據預設，表單中的步驟會以列示的序列執行。 不過，若欄位已包含值，或在已指定明確的導覽時，系統可能會略過相關步驟。 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

在此範例中，表單會執行下列步驟：

- 顯示歡迎訊息。 
- 填入 `SandwichOrder.Sandwich`。 
- 填入 `SandwichOrder.Length`。 
- 填入 `SandwichOrder.Bread`。 
- 填入 `SandwichOrder.Cheese`。 
- 如果使用者選取 `ToppingOptions.Everything`，會填入 `SandwichOrder.Toppings` 並新增遺漏值。 -. 顯示確認選取配料的訊息。 
- 填入 `SandwichOrder.Sauces`。 
- [以動態方式定義](#dynamically-define-field-values) `SandwichOrder.Specials` 的欄位值。 
- [以動態方式定義](#dynamically-define-messages-and-confirmations)三明治成本確認。 
- 填入 `SandwichOrder.DeliveryAddress` 並[確認](bot-builder-dotnet-formflow-advanced.md#add-business-logic)產生的字串。 如果地址不是以數字開頭，表單會傳回訊息。 
- 以自訂提示填入 `SandwichOrder.DeliveryTime`。 
- 確認訂單。 
- 新增任何已在類別中定義，但未由 `Field` 明確參考的其餘欄位。 (如果範例未呼叫 `AddRemainingFields` 方法，則表單不會包含任何未明確參考的欄位。) 
- 顯示感謝訊息。 
- 定義 `OnCompletionAsync` 處理常式以處理訂單。 

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他資源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)
- [將表單內容當地語系化](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 結構描述來定義表單](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式語言來自訂使用者體驗](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot 建立器 SDK 參考</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
