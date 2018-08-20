## <a name="application-settings-and-messaging-endpoint"></a>應用程式設定和傳訊端點

### <a name="verify-application-settings"></a>確認應用程式設定

若要讓 Bot 在雲端中正常運作，您必須確定其應用程式設定正確無誤。 如果您有 **appID** 和 **password**，請在部署過程中，更新應用程式組態設定中的 `Microsoft AppId` 和 `Microsoft App Password` 值。 若要尋找 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

如果您尚未向 Bot Framework 註冊您的 Bot (且因此還沒有 **appID** 和 **password**)，您可以使用這些設定的暫時性預留位置值來部署您的 Bot。
其後，在您[註冊您的 Bot](~/bot-service-quickstart-registration.md)之後，請使用在註冊期間為 Bot 產生的 **appID** 和 **password** 值來更新已部署的應用程式設定。

### <a id="messagingEndpoint"></a>確認傳訊端點

您已部署的 Bot 必須要有可從 Bot Framework Connector 服務接收訊息的**傳訊端點**。

> [!NOTE]
> 當您將 Bot 部署至 Azure 時，系統會自動為您的應用程式設定 SSL，藉以啟用 Bot Framework 所需的**傳訊端點**。
> 如果您部署了其他雲端服務，請務必確認您的應用程式已設定 SSL，讓 Bot 能夠有**傳訊端點**。
