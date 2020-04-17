---
title: scorable を使用したグローバル メッセージ ハンドラー
description: Bot Framework SDK for .NET 内の scorable を使用すると、より柔軟なダイアログを作成できます。
author: matthewshim-ms
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 101de0274c0daf4cbeb003703e299e0598f12a8a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "70297295"
---
# <a name="global-message-handlers-using-scorables"></a>scorable を使用したグローバル メッセージ ハンドラー

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ユーザーは、ボットが別の応答を期待しているときに、会話の途中で "ヘルプ"、"キャンセル"、"やり直し" などの言葉を使用してボット内の特定の機能にアクセスしようとします。 scorable ダイアログを使用して、このような要求を適切に処理するようにボットを設計できます。

scorable ダイアログはすべての受信メッセージを監視し、メッセージが何らかの方法で実行可能かどうかを判断します。 scorable であるメッセージには、各 scorable ダイアログによって [0 - 1] の範囲のスコアが割り当てられます。 最高スコアを決定する scorable ダイアログがダイアログ スタックの一番上に追加され、ユーザーに応答が渡されます。 scorable ダイアログの実行が完了すると、会話は中断したところから続行されます。

scorable を使用すると、通常のダイアログで見つけた通常の会話フローをユーザーが "中断" できるようにして、より柔軟な会話を作成できます。

## <a name="create-a-scorable-dialog"></a>scorable ダイアログを作成する

まず新しい[ダイアログ](bot-builder-dotnet-dialogs.md)を定義します。 次のコードでは、`IDialog` インターフェイスから派生したダイアログを使用しています。

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
scorable ダイアログを作成するには、`ScorableBase` 抽象クラスから継承したクラスを作成します。 次のコードは `SampleScorable` クラスを示しています。

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
`ScorableBase` 抽象クラスは `IScorable` インターフェイスから継承します。 クラスには以下の `IScorable` メソッドを実装する必要があります。

- `PrepareAsync` は、scorable インスタンスで呼び出される最初のメソッドです。 受信メッセージ アクティビティを受け取り、ダイアログの状態を分析して設定します。この状態は、`IScorable` インターフェイスの他のすべてのメソッドに渡されます。

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- `HasScore` メソッドは state プロパティを確認して、scorable ダイアログでメッセージのスコアが提供されているかどうかを判断します。 このメソッドから false が返されると、メッセージは scorable ダイアログで無視されます。

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `HasScore` から true が返された場合にのみ、`GetScore` はトリガーされます。 このメソッドでロジックをプロビジョニングし、0 - 1 の範囲でメッセージのスコアを決定します。

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- `PostAsync` メソッドで、scorable クラスに対して実行するコア アクションを定義します。 すべての scorable ダイアログは受信メッセージを監視し、scorable の GetScore メソッドに基づいて有効なメッセージにスコアを割り当てます。 最も高いスコア (0 - 1.0 の範囲) を決定する scorable クラスが、その scorable の `PostAsync` メソッドをトリガーします。

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- スコア付けプロセスが完了した後、`DoneAsync` が呼び出されます。 スコープ設定されたリソースを破棄するには、このメソッドを使用します。

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>IScorable サービスを登録するモジュールを作成する

次に、`SampleScorable` クラスをコンポーネントとして登録する `Module` を定義します。 これにより `IScorable` サービスがプロビジョニングされます。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>モジュールを登録する  

このプロセスの最後の手順は、`SampleScorable` をボットの Conversation Container に適用することです。 この手順で、Bot Framework のメッセージ処理パイプライン内に scorable サービスが登録されます。 次のコードは、**Global.asax.cs** のボット アプリの初期化中に `Conversation.Container`を更新する方法を示しています。

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>その他のリソース
* [グローバル メッセージ ハンドラーのサンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers)
* [シンプルな scorable ボットのサンプル](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample)
* [ダイアログの概要](bot-builder-dotnet-dialogs.md)
* [AutoFac](https://autofac.org/)
