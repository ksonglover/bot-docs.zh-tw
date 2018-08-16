## <a name="next-steps"></a>後續步驟
在您將 Bot 部署至雲端，並使用 Bot Framework 模擬器測試 Bot 確認部署成功後，Bot 發佈程序的下一個步驟將取決於您是否已使用 Bot Framework 註冊 Bot。

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>如果您已經使用 Bot Framework 註冊 Bot：

1. 請返回 <a href="https://dev.botframework.com" target="_blank">Bot Framework 入口網站</a>並[更新 Bot 的 設定資料](~/bot-service-manage-settings.md)以指定 Bot 的**傳訊端點**。

2. [將 Bot 設定為在一或多個通道上執行](~/bot-service-manage-channels.md)。

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>如果您尚未使用 Bot Framework 註冊 Bot：

1. [使用 Bot Framework 註冊 Bot](~/bot-service-quickstart-registration.md)。

2. 在您部署應用程式的組態設定中，更新 Microsoft 應用程式識別碼和 Microsoft 應用程式密碼的值，以指定註冊程序期間針對您的 Bot 所產生的 **appID** 和**密碼**值。 若要尋找 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

3. [將 Bot 設定為在一或多個通道上執行](~/bot-service-manage-channels.md)。