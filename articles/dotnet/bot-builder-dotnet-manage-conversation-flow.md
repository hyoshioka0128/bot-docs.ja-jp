---
title: ダイアログを使用して会話フローを管理する | Microsoft Docs
description: ダイアログと Bot Builder SDK for .NET を使用して会話をモデル化し、会話フローを管理する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 091da9c1db310491c7e2263d8e7eab51cfcc3787
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303569"
---
# <a name="manage-conversation-flow-with-dialogs"></a>ダイアログを使用して会話フローを管理する
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

この記事では、[ダイアログ](bot-builder-dotnet-dialogs.md)と Bot Builder SDK for .NET を使用して、会話フローをモデル化する方法を説明します。 

## <a name="invoke-the-root-dialog"></a>ルート ダイアログを呼び出す

最初に、ボット コントローラーは、"ルート ダイアログ" を呼び出します。 次の例は、コントローラーへの基本的な HTTP POST 呼び出しを記述して、ルート ダイアログを呼び出す方法を示しています。 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>"新しい注文" ダイアログを呼び出す

次に、ルート ダイアログ は、"新しい注文" ダイアログを呼び出します。 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> ダイアログのライフサイクル

ダイアログが呼び出されると、呼び出されたダイアログが会話フローを制御します。 すべての新しいメッセージは、ダイアログが閉じるか別のダイアログにリダイレクトするまで、そのダイアログによる処理の対象になります。 

C# で、`context.Wait()` を使用して、次回ユーザーがメッセージを送信したときに呼び出されるコールバックを指定できます。 ダイアログを閉じてスタックから削除する (これによってユーザーがスタック内の直前のダイアログに戻ります) には、`context.Done()` を使用します。 すべてのダイアログ メソッドは、`context.Wait()`、`context.Fail()`、`context.Done()` で終了するか、`context.Forward()` や `context.Call()` などのリダイレクト ディレクティブで終了する必要があります。 これらのいずれかで終わらないダイアログ メソッドはエラーになります (次回ユーザーがメッセージを送信したときに、フレームワークはどのようなアクションを実行するかわからないためです)。

## <a name="passing-state-between-dialogs"></a>ダイアログ間の状態の受け渡し

ボット状態の中に状態を格納できますが、ダイアログ クラス コンス トラクターをオーバー ロードすることでダイアログ ボックス間でデータを渡すこともできます。

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

ダイアログ コードを呼び出し、ユーザーから名前の値を渡します。

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>サンプル コード 

Bot Builder SDK for .NET でダイアログを使用して会話を管理する方法の完全なサンプルについては、GitHub の <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">マルチサイアログの基本的なサンプル</a>を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [ダイアログ](bot-builder-dotnet-dialogs.md)
- [会話フローの設計と制御](../bot-service-design-conversation-flow.md)
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">マルチダイアログの基本的なサンプル (GitHub)</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>
