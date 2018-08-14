---
title: Bot Builder SDK for .NET を使用したボットの作成 | Microsoft Docs
description: ボットの強力な構築フレームワークである Bot Builder SDK for .NET を使用してボットを作成します。
keywords: Bot Builder SDK、ボットの作成、クイック スタート、使用の開始
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fcb21fa38750c09f110a3c71f763a941c4437979
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304073"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET を使用したボットの作成
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

<a href="https://github.com/Microsoft/BotBuilder" target="_blank">Bot Builder SDK for .NET</a> は、Visual Studio と Windows を使用してボットを開発するための使いやすいフレームワークです。 SDK では、C# を活用することで、.NET 開発者が使い慣れた方法で強力なボットを作成できるようにします。


このチュートリアルでは、Bot Application テンプレートと Bot Builder SDK for .NET を使用してボットを構築し、Bot Framework Emulator でテストする方法を説明します。

## <a name="prerequisites"></a>前提条件
1. Visual Studio [2017](https://www.visualstudio.com/)。

2. Visual Studio で、すべての[拡張機能](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension)を最新のバージョンに更新します。

3. [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) のボット テンプレート。

> [!TIP]
> Visual Studio 2017 のプロジェクト テンプレートのディレクトリは通常 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\` にあり、項目テンプレートのディレクトリは `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\` にあります。

## <a name="create-your-bot"></a>ボットの作成

Visual Studio を開き、新しい C# プロジェクトを作成します。 新しいプロジェクトの **[簡易エコー ボット アプリケーション]** テンプレートを選択します。

![Visual Studio でのプロジェクトの作成](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio では、[IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264) をダウンロードしてインストールすることが必要な場合があります。 

Bot Application テンプレートのおかげで、プロジェクトには、このチュートリアルでボットを作成するのに必要なすべてのコードが含まれています。 実際には、追加のコードを記述する必要はありません。 ただし、ボットのテストに進む前に、Bot Application テンプレートに提供されているコードの一部を簡単に見てみましょう。

> [!TIP] 
> 必要に応じて、[NuGet パッケージ](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)を更新します。

## <a name="explore-the-code"></a>コードを調べる

まず、**Controllers\MessagesController.cs** 内の `Post` メソッドがユーザーからメッセージを受信して、ルート ダイアログを呼び出します。

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

ルート ダイアログは、メッセージを処理して応答を生成します。 **Dialogs\RootDialog.cs** 内の `MessageReceivedAsync` メソッドは、ユーザーのメッセージをオウム返しする応答を送信します。このメッセージは、'You sent' というテキストで始まり、'which was *##* characters' というテキストで終わります。ここで *##* はユーザーのメッセージの文字数を表します。

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>ボットのテスト

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>ボットの起動

エミュレーターをインストールした後、アプリケーション ホストとしてブラウザーを使用して、Visual Studio でボットを起動します。
この Visual Studio のスクリーンショットは、実行ボタンがクリックされたときにボットが Microsoft Edge で起動することを示しています。

![Visual Studio でのプロジェクトの実行](../media/connector-getstarted-start-bot-locally.png)

実行ボタンをクリックすると、Visual Studio はアプリケーションを構築し、localhost に展開して、Web ブラウザーを起動してアプリケーションの **default.htm** ページを表示します。
たとえば、Microsoft Edge に表示されるアプリケーションの **default.htm** ページは次のとおりです。

![Visual Studio での localhost を実行しているボット](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> プロジェクト内の **default.htm** ファイルを変更して、ボット アプリケーションの名前と説明を指定することができます。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

この時点では、ボットはローカルで実行されています。
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. 新しいボット構成を作成します。 アドレス バーに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。

2. **[保存および接続]** をクリックします。 **[Microsoft アプリ ID]** と **[Microsoft アプリ パスワード]** を指定する必要はありません。 現段階では、これらのフィールドは空白のままにしておくことができます。 この情報は、後ほど、[ボットを登録](~/bot-service-quickstart-registration.md)するときに取得します。

### <a name="test-your-bot"></a>ボットのテスト

ボットがローカルで実行され、エミュレーターに接続されたので、エミュレーターにいくつかのメッセージを入力してボットをテストします。
送信した各メッセージにボットがそのメッセージをオウム返しして応答することが分かります。このメッセージは、'You sent' というテキストで始まり、'which was *##* characters' というテキストで終わります。ここで *##* は送信したメッセージの文字数の合計です。


> [!TIP]
> エミュレーターで、会話の任意の吹き出しをクリックします。 メッセージの詳細は詳細ウィンドウに JSON 形式で表示されます。

これで、Bot Application テンプレートと Bot Builder SDK for .NET を使用して、ボットが正常に作成されました。

## <a name="next-steps"></a>次の手順

このクイック スタートでは、Bot Application テンプレートと Bot Builder SDK for .NET を使用して、単純なボットを作成し、Bot Framework Emulator を使用してボットの機能を確認します。

次に、Bot Builder SDK for .NET の主要概念を確認します。

> [!div class="nextstepaction"]
> [Bot Builder SDK for .NET の主要概念](bot-builder-dotnet-concepts.md)
