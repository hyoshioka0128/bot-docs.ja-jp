---
title: .NET ボットと Direct Line App Service 拡張機能
titleSuffix: Bot Service
description: .NET ボットが Direct Line App Service 拡張機能を使って機能するようにする
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 01/16/2020
ms.openlocfilehash: bda857cc3d0b79369760cf5f5bd396649943b8b7
ms.sourcegitcommit: 2412f96ad8f74dfa615c71f566c5befffb920658
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/25/2020
ms.locfileid: "82158799"
---
# <a name="configure-net-bot-for-extension"></a>拡張機能のための .NET ボットの構成

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

この記事では、**名前付きパイプ**を使って機能するようにボットを更新する方法、およびボットがホストされている **Azure App Service** リソースで Direct Line App Service 拡張機能を有効にする方法について説明します。 こちらの関連記事「[Direct Line App Service 拡張機能に接続する .NET クライアントを作成する](bot-service-channel-directline-extension-net-client.md)」もお読みください。


## <a name="prerequisites"></a>前提条件

次に説明する手順を実行するには、**Azure App Service** リソースと、関連する **App Service** が Azure にある必要があります。

## <a name="enable-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

このセクションでは、ボットのチャネル構成からのキーと、ボットがホストされている **Azure App Service** リソースを使用して Direct Line App Service 機能拡張を有効にする方法について説明します。

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を使用するように .NET ボットを更新する

> [!NOTE]   
> `Microsoft.Bot.Builder.StreamingExtensions` プレビュー パッケージは非推奨になりました。 SDK v4.8 には、[ストリーミング コード](https://github.com/microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder/Streaming)が含まれるようになりました。 以前にボットでプレビュー パッケージを使用していた場合は、次の手順を実行する前にそれらを削除する必要があります。 

1. Visual Studio でボット プロジェクトを開きます。
2. プロジェクトが Bot Builder SDK のバージョン4.8 以降を使用していることを確認します。
3. アプリが **Bot Framework NamedPipe** を使用することを許可します。
    - `Startup.cs` ファイルを開きます。
    - ``Configure`` メソッドで、``UseNamedPipes`` にコードを追加します。

    ```csharp

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseHsts();
        }

        app.UseDefaultFiles();
        app.UseStaticFiles();

        // Allow the bot to use named pipes.
        app.UseNamedPipes(System.Environment.GetEnvironmentVariable("APPSETTING_WEBSITE_SITE_NAME") + ".directline");

        app.UseMvc();
    }
    ```

4. `Startup.cs` ファイルを保存します。
5. `appsettings.json` ファイルを開き、次の値を入力します。
    1. `"MicrosoftAppId": "<secret Id>"`
    2. `"MicrosoftAppPassword": "<secret password>"`

    これらの値は、**appid** と、サービス登録グループに関連付けられた **appSecret** です。

6. ボットを Azure App Service に**発行**します。

### <a name="gather-your-direct-line-extension-keys"></a>Direct Line 拡張機能キーを収集する

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、目的の **Azure Bot Service** リソースを見つけます。
1. **[チャネル]** をクリックして、ボットのチャネルを構成します。
1. **Direct Line** チャネルがまだ有効になっていない場合は、クリックして有効にします。
1. 既に有効になっている場合は、[Connect to channels]\(チャネルに接続\) テーブルの Direct Line 行で **[編集]** リンクをクリックします。
1. App Service 拡張機能キーのセクションまで下にスクロールします。
1. **[表示] リンク**をクリックしていずれかのキーを表示し、その値をコピーして保存します。 この値は次のセクションで使用します。

![App Service 拡張機能キー](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、ボットをホストしている、またはホストする予定の Web アプリの **Azure App Service** リソース ページを見つけます
1. **[Configuration]\(構成\)** をクリックします。 *[アプリケーションの設定]* セクションで、次の新しい設定を追加します。

    |名前|値|
    |---|---|
    |DirectLineExtensionKey|<App_Service_Extension_Key>|
    |DIRECTLINE_EXTENSION_VERSION|latest|

    ここで、*App_Service_Extension_Key* は、以前に保存した値です。

1. *[構成]* セクション内で、 **[全般設定]** セクションをクリックし、 **[Web ソケット]** を有効にします。
1. **[保存]** をクリックして設定を保存します。 これにより、Azure App Service が再起動されます。

## <a name="confirm-direct-line-app-extension-and-the-bot-are-initialized"></a>Direct Line アプリ拡張機能とボットが初期化されていることを確認します

ブラウザーで、 https://<your_app_service>.azurewebsites.net/.bot に移動します。
すべて正しければ、ページは JSON コンテンツ `{"v":"123","k":true,"ib":true,"ob":true,"initialized":true}` を返します。 これは、**すべてが正常に動作している**場合に取得される情報です。ここでは、次のようになります。

- **v** によって、Direct Line App Service 拡張機能 (ASE) のビルドバージョンが表示されます。
- **k** によって、Direct Line ASE がその構成から App Service 拡張機能のキーを読み取れるかどうかを決定します。
- **initialized** によって、Direct Line ASE が App Service 拡張機能キーを使用して Azure Bot Service からボット メタデータをダウンロードできるかどうかを決定します。
- **ib** によって、Direct Line ASE がボットとの受信接続を確立できるかどうかを決定します。
- **ob** によって、Direct Line ASE がボットとの送信接続を確立できるかどうかを決定します。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [.NET クライアントを作成する](./bot-service-channel-directline-extension-net-client.md)
