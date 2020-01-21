---
title: Direct Line App Service 拡張機能用の .NET クライアントを作成する
titleSuffix: Bot Service
description: Direct Line App Service 拡張機能に接続する .NET クライアント C#
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 0ed4bbeb9a882bcf8e4dd75364211f1a3538479b
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75793222"
---
# <a name="create-net-client-to-connect-to-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能に接続する .NET クライアントを作成する

この記事では、Direct Line App Service 拡張機能に接続する .NET クライアントを C# で作成する方法について説明します。

## <a name="gather-your-direct-line-extension-keys"></a>Direct Line 拡張機能キーを収集する

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、目的の **Azure Bot Service** リソースを見つけます。
1. **[チャネル]** をクリックして、ボットのチャネルを構成します。
1. **Direct Line** チャネルがまだ有効になっていない場合は、クリックして有効にします。 
1. 既に有効になっている場合は、[Connect to channels]\(チャネルに接続\) テーブルの Direct Line 行で **[編集]** リンクをクリックします。
1. [サイト] セクションまでスクロールします。 通常、削除または名前を変更していない限り、既定のサイトがあります。
1. **[表示] リンク**をクリックしていずれかのキーを表示し、その値をコピーします。

    ![App Service 拡張機能キー](./media/channels/direct-line-extension-extension-keys-net-client.png)

> [!NOTE]
> この値は、Direct Line App Service 拡張機能に接続するために使用される Direct Line クライアントのシークレットです。 必要に応じて追加のサイトを作成でき、それらのシークレット値も使用できます。

## <a name="add-the-preview-nuget-package-source"></a>プレビュー版の Nuget パッケージのソースを追加する

C# Direct Line クライアントを作成するために必要なプレビュー版の NuGet パッケージは、NuGet フィードにあります。

1. Visual Studio で、 **[ツール]->[オプション]** メニュー項目に移動します。
1. **[NuGet パッケージ マネージャー]->[パッケージ ソース]** 項目を選択します。
1. [+] ボタンをクリックして、次の値を使って新しいパッケージを追加します。
    - 名前:DL ASE Preview
    - ソース: https://botbuilder.myget.org/F/experimental/api/v3/index.json
1. **[更新]** ボタンをクリックして値を保存します。
1. **[OK]** をクリックして、パッケージ ソースの構成を終了します。

## <a name="create-a-c-direct-line-client"></a>C# Direct Line クライアントを作成する

Direct Line App Service 拡張機能との対話は、従来の Direct Line とは異なる方法で行われます。これは、ほとんどの通信が *WebSocket* で行われるためです。 更新された Direct Line クライアントには、*WebSocket* の開始/終了、WebSocket 経由でのコマンドの送信、およびボットから返される Activity の受信のためのヘルパー クラスが含まれています。 このセクションでは、ボットと対話するための単純な C# クライアントを作成する方法について説明します。

1. Visual Studio で、新しい .NET Core 2.2 コンソール アプリケーション プロジェクトを作成します。
1. **DirectLine クライアント NuGet** をプロジェクトに追加します
    - [ソリューション] ツリーで [依存関係] をクリックします
    - **[NuGet パッケージの管理]** を選択します
    - パッケージ ソースを、上で定義したもの (DL ASE Preview) に変更します
    - パッケージ *Microsoft.Bot.Connector.Directline* バージョン v3.0.3-Preview1 以降を見つけます。
    - **[パッケージのインストール]** をクリックします。
1. クライアントを作成し、シークレットを使用してトークンを生成します。 この手順は、次に示すように **.bot/** パスが付加された、ボットで使用する必要があるエンドポイントを除き、他の C# Direct Line クライアントの構築と同じです。 末尾の **/** を忘れないでください。

    ```csharp
    string endpoint = "https://<YOUR_BOT_HOST>.azurewebsites.net/.bot/";
    string secret = "<YOUR_BOT_SECRET>";

    var tokenClient = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(secret));
    var conversation = await tokenClient.Tokens.GenerateTokenForNewConversationAsync();
    ```

1. トークンの生成から会話の参照を取得したら、この会話 ID を使用して、`DirectLineClient` の新しい `StreamingConversations` プロパティで WebSocket を開くことができます。 これを行うには、ボットが `ActivitySets` をクライアントに送信するときに呼び出されるコールバックを作成する必要があります。

    ```csharp
    public static void ReceiveActivities(ActivitySet activitySet)
    {
        if (activitySet != null)
        {
            foreach (var a in activitySet.Activities)
            {
                if (a.Type == ActivityTypes.Message && a.From.Id.Contains("bot"))
                {
                    Console.WriteLine($"<Bot>: {a.Text}");
                }
            }
        }
    }
    ```

1. これで、会話のトークン `conversationId` と `ReceiveActivities` コールバックを使用して `StreamingConversations` プロパティで WebSocket を開く準備ができました。

    ```csharp
    var client = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(conversation.Token));

    await client.StreamingConversations.ConnectAsync(
        conversation.ConversationId,
        ReceiveActivities);
    ```

1. これで、クライアントを使用して会話を開始し、`Activities` をボットに送信できるようになりました。

    ```csharp

    var startConversation = await client.StreamingConversations.StartConversationAsync();
    var from = new ChannelAccount() { Id = "123", Name = "Fred" };
    var message = Console.ReadLine();

    while (message != "end")
    {
        try
        {
            var response = await client.StreamingConversations.PostActivityAsync(
                startConversation.ConversationId,
                new Activity()
                {
                    Type = "message",
                    Text = message,
                    From = from
                });
        }
        catch (OperationException ex)
        {
            Console.WriteLine(
                $"OperationException when calling PostActivityAsync: ({ex.StatusCode})");
        }
        message = Console.ReadLine();
    }
    ```
