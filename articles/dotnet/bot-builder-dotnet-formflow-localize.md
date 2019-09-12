---
title: 將表單內容當地語系化 | Microsoft Docs
description: 了解如何使用 FormFlow 和適用於 .NET 的 Bot Framework SDK，將表單內容當地語系化。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 795232b401284becd940daed6bf7da8642c12efd
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297956"
---
# <a name="localize-form-content"></a>將表單內容當地語系化

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

表單的當地語系化語言是由目前執行緒的 [CurrentUICulture](https://msdn.microsoft.com/library/system.threading.thread.currentuiculture(v=vs.110).aspx) 和 [CurrentCulture](https://msdn.microsoft.com/library/system.threading.thread.currentculture(v=vs.110).aspx) 所決定。
根據預設，文化特性是衍生自目前訊息的 [地區設定]  欄位，但是您可以覆寫該預設行為。
根據您建構 Bot 的方式，當地語系化資訊可能來自最多三種不同的來源：

- **PromptDialog** 和 **FormFlow** 的內建當地語系化
- 您在表單中針對靜態字串產生的資源檔
- 您使用動態計算欄位、訊息或確認的字串所建立的資源檔

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>在表單中針對靜態字串產生資源檔

表單中的靜態字串包含表單從 C# 類別中的資訊所產生的字串，以及您指定為提示、範本、訊息或確認的字串。
系統不會將從內建範本產生的字串視為靜態字串，因為這些字串已當地語系化。
因為表單中有許多字串是自動產生的，所以直接使用一般 C# 資源字串並不可行。
相反地，您可以藉由呼叫 `IFormBuilder.SaveResources` 或使用適用於 .NET 的 Bot 建立器 SDK
 隨附的 **RView** 工具，為表單中的靜態字串產生資源檔。

### <a name="use-iformbuildersaveresources"></a>使用 IFormBuilder.SaveResources

您可以藉由在表單上呼叫 [IFormBuilder.SaveResources][saveResources]，將字串儲存至 .resx 檔案。

### <a name="use-rview"></a>使用 RView

或者，您也可以藉由使用適用於 .NET 的 Bot 建立器 SDK
 隨附的 <a href="https://aka.ms/v3-cs-RView-library" target="_blank">RView</a> 工具，建立以 .dll 或 .exe 為基礎的資源檔。
若要產生 .resx 檔案，請執行 **rview** 並且指定組件，其中包含您的靜態表單建置方法和該方法的路徑。
此程式碼片段示範如何使用 **RView** 來產生 `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` 資源檔。

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

此摘要示範部分 .resx 檔案，該檔案是藉由執行此 **rview** 命令而產生的。

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>設定您的專案

產生資源檔之後，請新增至您的專案中，然後藉由完成下列步驟來設定中性語言： 

1. 以滑鼠右鍵按一下專案，然後選取 [應用程式]  。
2. 按一下 [組件資訊]  。
3. 選取與您用來開發 Bot 的語言對應的 [中性語言]  值。

建立您的表單時，[IFormBuilder.Build][build] 方法會自動尋找包含您表單類型名稱的資源，並且使用這些資源將表單中的靜態字串當地語系化。 

> [!NOTE]
> 使用 [Advanced.Field.SetDefine][setDefine] 定義的動態計算欄位 (如同[使用動態欄位](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)中所述) 無法透過與靜態欄位相同的方式進行當地語系化，因為動態計算欄位的字串是在表單填入時建構的。 但是，您可以藉由使用一般 C# 當地語系化機制，將動態計算欄位當地語系化。

### <a name="localize-resource-files"></a>將資源檔當地語系化 

將資源檔新增至您的專案之後，您可以使用<a href="https://developer.microsoft.com/windows/develop/multilingual-app-toolkit" target="_blank">多語應用程式工具組 (MAT)</a> 將資源檔當地語系化。 安裝 **MAT**，然後藉由完成下列步驟針對專案加以啟用：

1. 在 Visual Studio 方案總管中選取您的專案。
2. 按一下 [工具]  、[多語應用程式工具組]  ，然後按一下 [啟用]  。
3. 以滑鼠右鍵按一下專案，然後選取 [多語應用程式工具組]  、[新增翻譯]  以選取翻譯。 這樣會建立業界標準的 <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> 檔案，您可以自動或手動進行翻譯。

> [!NOTE]
> 雖然這篇文章說明如何使用多語應用程式工具組來將內容當地語系化，但是您可以透過其他各種方式來實作當地語系化。

## <a name="see-it-in-action"></a>了解它是如何運作的

這個程式碼範例是在[使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)其中一個程式碼的基礎上所建置，以便如上所述進行當地語系化。 在此範例中，`DynamicSandwich` 類別 (此處未顯示) 包含動態計算欄位、訊息和確認的當地語系化資訊。

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

當 `CurrentUICulture` 是**法文**時，此程式碼片段會顯示 Bot 與使用者之間產生的互動。

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>範例程式碼

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他資源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的進階功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 來自訂表單](bot-builder-dotnet-formflow-formbuilder.md)
- [使用 JSON 結構描述來定義表單](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式語言來自訂使用者體驗](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
