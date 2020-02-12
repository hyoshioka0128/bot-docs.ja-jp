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
ms.openlocfilehash: c49a632696ea0708f817f733aff7b2743600ccb6
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071700"
---
# <a name="configure-net-bot-for-extension"></a>拡張機能のための .NET ボットの構成

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

この記事では、**名前付きパイプ**を使って機能するようにボットを更新する方法、およびボットがホストされている **Azure App Service** リソースで Direct Line App Service 拡張機能を有効にする方法について説明します。

## <a name="prerequisites"></a>前提条件

次に説明する手順を実行するには、**Azure App Service** リソースと、関連する **App Service** が Azure にある必要があります。

## <a name="enable-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

このセクションでは、ボットのチャネル構成からのキーと、ボットがホストされている **Azure App Service** リソースを使用して Direct Line App Service 機能拡張を有効にする方法について説明します。

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を使用するように .NET ボットを更新する

> [!NOTE]
> `Microsoft.Bot.Builder.StreamingExtensions` はプレビュー パッケージであり、更新されません。 SDK v4.7 には、[ストリーミング コード](https://github.com/microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder/Streaming)が含まれているため、ストリーミング パッケージを別途自分でインストールする必要はありません。

1. Visual Studio でボット プロジェクトを開きます。
2. **Streaming Extension NuGet** パッケージをプロジェクトに追加します。
    1. プロジェクトで、 **[依存関係]** を右クリックし、 **[NuGet パッケージの管理]** を選択します。
    2. *[参照]* タブで **[プレリリースを含める]** をクリックして、プレビュー パッケージを表示します。
    3. **Microsoft.Bot.Builder.StreamingExtensions** パッケージを選択します。
    4. **[インストール]** をクリックしてパッケージをインストールし、使用許諾契約書を読んで同意します。
3. アプリが **Bot Framework NamedPipe** を使用することを許可します。
    - `Startup.cs` ファイルを開きます。
    - ``Configure`` メソッドで、``UseBotFrameworkNamedPipe`` にコードを追加します。

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
        app.UseNamedPipes();

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
1. **[表示] リンク**をクリックしていずれかのキーを表示し、その値をコピーします。

![App Service 拡張機能キー](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、ボットをホストしている、またはホストする予定の Web アプリの **Azure App Service** リソース ページを見つけます
1. **[Configuration]\(構成\)** をクリックします。 *[アプリケーションの設定]* セクションで、次の新しい設定を追加します。

    |Name|Value|
    |---|---|
    |DirectLineExtensionKey|<App_Service_Extension_Key_From_Section_1>|
    |DIRECTLINE_EXTENSION_VERSION|latest|

1. *[構成]* セクション内で、 **[全般設定]** セクションをクリックし、 **[Web ソケット]** を有効にします。
1. **[保存]** をクリックして設定を保存します。 これにより、Azure App Service が再起動されます。

## <a name="confirm-direct-line-app-extension-and-the-bot-are-initialized"></a>Direct Line アプリ拡張機能とボットが初期化されていることを確認します

ブラウザーで、 https://<your_app_service>.azurewebsites.net/.bot に移動します。
すべて正しければ、ページは JSON コンテンツ `{"k":true,"ib":true,"ob":true,"initialized":true}` を返します。 これは、**すべてが正常に動作している**場合に取得される情報です。ここでは、次のようになります。

- **k** によって、Direct Line App Service 拡張機能 (ASE) がその構成から App Service 拡張機能のキーを読み取れるかどうかを決定します。
- **initialized** によって、Direct Line ASE が App Service 拡張機能キーを使用して Azure Bot Service からボット メタデータをダウンロードできるかどうかを決定します。
- **ib** によって、Direct Line ASE がボットとの受信接続を確立できるかどうかを決定します。
- **ob** によって、Direct Line ASE がボットとの送信接続を確立できるかどうかを決定します。
