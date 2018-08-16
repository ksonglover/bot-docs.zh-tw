Bot 服務會針對 Bot 提供兩種的不同主控方案。 將 Bot 原始程式碼從一個方案轉換為另一個方案，必須以手動方式進行。   

## <a name="app-service-plan"></a>App Service 方案

使用 App Service 方案的 Bot 是標準的 Azure Web 應用程式，您可以將它設定來配置可預測成本及規模的預先定義容量。 透過使用此主控方案的 Bot，您可以：

* 使用進階的瀏覽器內程式碼編輯器，在線上編輯 Bot 原始程式碼。
* 使用 Visual Studio 來對您的 C# Bot 進行下載、偵錯及重新發佈。
* 輕鬆設定適用於 Visual Studio Online 和 Github 的持續部署。
* 使用針對 Bot 建立器 SDK 準備的範例程式碼。

## <a name="consumption-plan"></a>取用方案

使用取用方案的 Bot 是在 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> 上執行的無伺服器 Bot，並使用按執行數付費的 Azure Functions 定價。 使用此主控方案的 Bot 可調整來處理大量的流量尖峰。 您可以使用基本的瀏覽器內程式碼編輯器，在線上編輯 Bot 原始程式碼。 如需取用方案 Bot 的執行階段環境詳細資訊，請參閱 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 取用和 App Service 方案</a>。
