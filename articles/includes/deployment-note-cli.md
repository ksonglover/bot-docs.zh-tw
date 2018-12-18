如果您使用 LUIS 等服務，則也必須傳遞 `luisAuthoringKey`。 如果您想要在 Azure 中使用現有的資源群組，請使用 `groupName` 引數搭配上述命令。

強烈建議您使用 `verbose` 選項來協助排解在 Bot 部署期間可能發生的問題。 搭配 `msbot clone services` 命令使用的其他選項如下所述：

| 引數    | 說明 |
|--------------|-------------|
| `folder`     | `bot.receipe` 檔案的位置。 依預設，receipe 檔案在 `DeploymentsScript/MSBotClone` 中建立。 請勿修改這個檔案。|
| `location`   | 用來建立 Bot 服務資源的地理位置。 例如，eastus、westus、westus2 等等。|
| `proj-file`  | 若為 C# Bot，這是 .csproj 檔案。 若為 JS Bot，這是您本機 Bot 的啟動專案檔案名稱 (例如 index.js)。|
| `name`       | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格。|

建立 Azure 資源之前，系統會提示您完成驗證。 依照畫面上顯示的指示完成這個步驟。

請注意，以上步驟需要_數秒到數分鐘_的時間才能完成，而且在 Azure 中建立的資源會修飾其名稱。 若要深入了解名稱修飾，請參閱 GitHub 存放庫中的[問題編號 796](https://github.com/Microsoft/botbuilder-tools/issues/796)。
