---
title: プロアクティブ メッセージの送信 | Microsoft Docs
description: Bot Builder SDK for .NET を使用してプロアクティブ メッセージを送信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f338a405ccc74e66606c0eba5604776edb1fc1ff
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303156"
---
# <a name="send-proactive-messages"></a>プロアクティブ メッセージの送信
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>プロアクティブ メッセージのタイプ 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>アドホック プロアクティブ メッセージの送信

以下のコード サンプルは、Bot Builder SDK for .NET を使用してアドホック プロアクティブ メッセージを送信する方法を示しています。

ユーザーにアドホック メッセージを送信できるようにするには、ボットはまず現在の会話からユーザーに関する情報を収集し、保存する必要があります。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> この例では、わかりやすくするために、ユーザー データの格納方法は指定しません。 ボットが後でデータを取得できる限り、データの格納方法は問題となりません。

データが格納されたので、ボットは簡単にデータを取得し、アドホック プロアクティブ メッセージを構築して、送信することができます。 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> ボットが以前に格納された会話 ID を指定した場合、メッセージは、クライアント上の既存の会話ウィンドウ内のユーザーに配信される可能性があります。 ボットが新しい会話 ID を生成した場合、クライアントが複数の会話ウィンドウをサポートしていれば、メッセージは、クライアント上の新しい会話ウィンドウ内のユーザーに配信されます。 

## <a name="send-a-dialog-based-proactive-message"></a>ダイアログ ベースのプロアクティブ メッセージの送信

以下のコード サンプルは、Bot Builder SDK for .NET を使用してダイアログ ベースのプロアクティブ メッセージを送信する方法を示しています。

ユーザーにダイアログ ベースのプロアクティブ メッセージを送信できるようにするには、ボットはまず現在の会話から情報を収集し、保存する必要があります。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

メッセージを送信するとき、ボットは新しいダイアログを作成し、ダイアログ スタックの先頭に追加します。 新しいダイアログは、会話を制御し、プロアクティブ メッセージを配信して、終了してから、スタック内の前のダイアログ ボックスに制御を返します。 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog` は、会話が終了するまで制御します。 そのタスクが完了したら、`context.Done` を呼び出して終了し、前のダイアログに制御を返します。 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>サンプル コード

Bot Builder SDK for .NET を使用してプロアクティブ メッセージを送信する方法を示す完全なサンプルについては、GitHub で<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">プロアクティブ メッセージのサンプル</a>を参照してください。 プロアクティブ メッセージのサンプル内で、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> はアドホック プロアクティブ メッセージを送信する方法を示し、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a> はダイアログ ベースのプロアクティブ メッセージを送信する方法を示しています。 

## <a name="additional-resources"></a>その他のリソース

- [会話フローの設計と制御](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">プロアクティブ メッセージのサンプル (GitHub)</a>

