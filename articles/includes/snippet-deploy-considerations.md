## <a name="application-settings-and-messaging-endpoint"></a>アプリケーション設定とメッセージング エンドポイント

### <a name="verify-application-settings"></a>アプリケーション設定を確認する

ボットがクラウドで正常に機能するように、アプリケーション設定が正しいことを確認する必要があります。 **アプリ ID** と**パスワード**がある場合は、デプロイ プロセスの一環としてアプリケーションの構成設定の `Microsoft AppId` と `Microsoft App Password` の値を更新します。 自分のボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

まだボットを Bot Framework に登録していない (したがって、**アプリ ID** と**パスワード**がまだない) 場合は、これらの設定を一時的なプレースホルダー値にしてボットをデプロイできます。
そして、[ボットを登録](~/bot-service-quickstart-registration.md)した後、デプロイされているアプリケーションの設定を、登録の間にボットに対して生成された**アプリ ID** と**パスワード**の値で更新します。

### <a id="messagingEndpoint"></a> メッセージング エンドポイントを確認する

デプロイされるボットには、Bot Framework Connector Service からメッセージを受信できる**メッセージング エンドポイント**が必要です。

> [!NOTE]
> ボットを Azure にデプロイすると、SSL がアプリケーション用に自動的に構成され、それにより Bot Framework で必要な**メッセージング エンドポイント**が有効になります。
> 別のクラウド サービスにデプロイする場合は、ボットが**メッセージング エンドポイント**を備えるように、アプリケーションが SSL 用に構成されることを確認してください。
