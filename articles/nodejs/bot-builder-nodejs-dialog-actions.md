---
title: ユーザー アクションを処理する | Microsoft Docs
description: Bot Framework SDK for Node.js を使用して、特定のキーワードを含むユーザー入力をボットが聞き取って処理できるようにすることで、ユーザー アクションを処理する方法について説明します。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7ca595b1c24769addfbdf7975c48d3a052c4a2de
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54226007"
---
# <a name="handle-user-actions"></a>ユーザー アクションを処理する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

ユーザーは通常、「ヘルプ」、「キャンセル」、「やり直し」などのキーワードを使用して、ボット内の特定の機能にアクセスしようとします。 ユーザーは会話の途中で、ボットが別の応答を期待しているときにこれを行います。 **アクション**を実装することによって、そのような要求をより適切に処理するようにボットを設計できます。 ハンドラーは「ヘルプ」、「キャンセル」、「やり直し」などの指定されたキーワードのユーザー入力を検証し、適切に応答します。 

![ユーザーの応答](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>アクションの種類

ダイアログにアタッチできるアクションの種類を次の表に一覧で示します。 各アクション名のリンクをクリックすると、そのアクションについて詳しく説明したセクションに移動します。

| Action | Scope (スコープ) | 説明 |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | グローバル | ダイアログ スタックをクリアし、それ自体をスタックの底にプッシュするアクションをダイアログにバインドします。 この既定の動作を上書きするには、`onSelectAction` オプションを使用します。 |
| [customAction](#bind-a-customaction) | グローバル | ダイアログ スタックに影響を与えずに情報を処理したり、アクションを実行したりできるカスタム アクションをボットにバインドします。 このアクションの機能をカスタマイズするには、`onSelectAction` オプションを使用します。 |
[beginDialogAction](#bind-a-begindialogaction) | コンテキストに応じた判断 | トリガーされると別のダイアログを開始するアクションをダイアログにバインドします。 開始するダイアログはスタックにプッシュされ、終了すると消えます。 |
[reloadAction](#bind-a-reloadaction) | コンテキストに応じた判断 | トリガーされるとダイアログの再読み込みを発生させるアクションをダイアログにアタッチします。 `reloadAction` を使用して、「やり直し」のようなユーザーの発話を処理できます。 |
[cancelAction](#bind-a-cancelaction) | コンテキストに応じた判断 | トリガーされるとダイアログをキャンセルするアクションをダイアログにバインドします。 `cancelAction` を使用して、「キャンセル」や「気にしないで」のようなユーザーの発話を処理できます。 |
[endConversationAction](#bind-an-endconversationaction) | コンテキストに応じた判断 | トリガーされるとユーザーとの会話を終了するアクションをダイアログにバインドします。 `endConversationAction` を使用して、「さようなら」のようなユーザーの発話を処理できます。 |

## <a name="action-precedence"></a>アクションの優先順位 

ボットがユーザーからの発話を受信すると、ボットはダイアログ スタックにあるすべての登録済みアクションと発話を照合します。 マッチングはスタックの一番上から始まってスタックの底まで進みます。 何も一致しない場合、すべてのグローバル アクションの `matches` オプションと発話を照合します。

アクションの優先順位は、異なるコンテキストで同じコマンドを使用する場合に重要です。 たとえば、ボットの「ヘルプ」コマンドを一般的なヘルプとして使用できます。 「ヘルプ」は個々のタスクに対しても使用できますが、これらのヘルプ コマンドは各タスクのコンテキストに依存します。 この点を詳しく説明する作業サンプルについては、「[ユーザー入力に応答する](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input)」を参照してください。

## <a name="bind-actions-to-dialog"></a>アクションをダイアログにバインドする

ユーザーの発話またはボタンのクリックのどちらかがアクションを*トリガー*することができ、これは[ダイアログ](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html)に関連付けられます。
*matches* が指定されている場合、アクションは、そのアクションをトリガーする単語またはフレーズをユーザーが言うのを聞き取ります。  `matches` オプションには正規表現または[認識エンジン][RecognizeIntent]の名前を渡すことができます。
アクションをボタンのクリックにバインドするには、[CardAction.dialogAction()][CardAction] を使用してアクションをトリガーします。

アクションは*連鎖可能*なため、必要な数のアクションをダイアログにバインドできます。

### <a name="bind-a-triggeraction"></a>triggerAction のバインド

[triggerAction][triggerAction] をダイアログにバインドするには、次の手順を実行します。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

`triggerAction` をダイアログにバインドすると、これがボットに登録されます。 トリガーされると、`triggerAction` はダイアログ スタックをクリアし、トリガーされたダイアログをスタックにプッシュします。 このアクションは、[会話のトピック](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation)を切り替えたり、任意のスタンドアロン タスクの要求をユーザーに許可したりするために使用するのが最適です。 このアクションがダイアログ スタックをクリアする動作をオーバーライドする場合は、`onSelectAction` オプションを `triggerAction` に追加します。

次のコード スニペットは、ダイアログ スタックを消去せずに、グローバル コンテキストから一般的なヘルプを提供する方法を示しています。

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

この場合、`triggerAction` は (`orderDinner` ダイアログではなく) `help` ダイアログ自体にアタッチされます。 `onSelectAction` オプションを使用すると、ダイアログ スタックをクリアせずにこのダイアログを開始できます。 これにより、「ヘルプ」、「バージョン情報」、「サポート」などのグローバルな要求を処理できます。`onSelectAction` オプションは、`session.beginDialog` メソッドを明示的に呼び出して、トリガーされたダイアログを開始します。 トリガーされるダイアログの ID は `args.action` によって提供されます。 ("help" のような) ダイアログ ID をこのメソッドに手動でコーディングしないでください。これを守らないと、実行時エラーが発生する場合があります。 `orderDinner` タスク自体のコンテキスト ヘルプ メッセージをトリガーする場合は、代わりに `beginDialogAction` を `orderDinner` ダイアログにアタッチすることを検討してください。

### <a name="bind-a-customaction"></a>customAction のバインド

他の種類のアクションと異なり、`customAction` には既定のアクションが定義されていません。 このアクションで何を実行するかは開発者が定義します。 `customAction` を使用することの利点は、ダイアログ スタックを操作せずにユーザーの要求を処理するオプションがあることです。 `customAction` がトリガーされると、`onSelectAction` オプションは新しいダイアログをスタックにプッシュせずに要求を処理できます。 アクションが完了すると、スタックの一番上にあるダイアログに制御が戻され、ボットは動作を継続できます。

`customAction` を使用して、「今の外の気温は何度?」、「パリは今何時?」、「今日の夕方 5 時に牛乳を買うからリマインドして」のような、一般的でちょっとしたアクション要求を提供できます。これらは、ボットがスタックを操作せずに実行できる一般的なアクションです。

`customAction` に関するもう一つの重要な違いは、ダイアログではなくボットにバインドすることです。

次のコード サンプルは、リマインダー設定要求を聞く `bot` に `customAction` をバインドする方法を示しています。

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>beginDialogAction のバインド

`beginDialogAction` をダイアログにバインドすると、アクションがダイアログに登録されます。 このメソッドは、トリガーされると別のダイアログを開始します。 このアクションの動作は、[beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog) メソッドの呼び出しに似ています。 新しいダイアログはダイアログ スタックの一番上にプッシュされるため、現在のタスクを自動的に終了させることはありません。 新しいタスクが終了すると、現在のタスクが継続されます。 

次のコード スニペットは、[beginDialogAction][beginDialogAction] をダイアログにバインドする方法を示しています。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

新しいダイアログに追加の引数を渡す必要がある場合、[`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) オプションをアクションに追加できます。

上記のサンプルを使用して、`dialogArgs` によって渡された引数を受け入れるようにサンプルを変更できます。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>reloadAction のバインド

`reloadAction` をダイアログにバインドすると、ダイアログに登録されます。 このアクションをダイアログにバインドすると、アクションがトリガーされたときにダイアログが再開するようになります。 このアクションをトリガーすることは、[replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog) メソッドの呼び出しに似ています。 これは、「やり直し」のようなユーザーの発話を処理するロジックの実装や、[ループ](bot-builder-nodejs-dialog-replace.md#repeat-an-action)を作成する目的に役立ちます。

次のコード スニペットは、[reloadAction][reloadAction] をダイアログにバインドする方法を示しています。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

再読み込みされるダイアログに追加の引数を渡す必要がある場合、[`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) オプションをアクションに追加できます。 このオプションは `args` パラメーターに渡されます。 再読み込みアクション時に引数を受け取るように上記のサンプル コードを記述し直すと、次のようになります。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>cancelAction のバインド

`cancelAction` をバインドすると、ダイアログに登録されます。 トリガーされると、このアクションはダイアログを突然終了します。 ダイアログが終了すると、親のダイアログとコードが再開し、元のダイアログがキャンセルされた (`canceled`) ことを示します。 このアクションによって、「気にしないで」や「キャンセル」などの発話を処理できます。 代わりにプログラムによってダイアログをキャンセルする必要がある場合は、「[ダイアログを取り消す](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog)」を参照してください。 *再開されるコード*の詳細については、「[Prompt results](bot-builder-nodejs-dialog-prompt.md#prompt-results)」(結果のプロンプト) を参照してください。 

次のコード スニペットは、[cancelAction][cancelAction] をダイアログにバインドする方法を示しています。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>endConversationAction のバインド

`endConversationAction` をバインドすると、ダイアログに登録されます。 トリガーされると、このアクションはユーザーとの会話を終了します。 このアクションをトリガーすることは、[endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation) メソッドの呼び出しに似ています。 会話が終了すると、Bot Framework SDK for Node.js はダイアログ スタックと保持されていた状態データをクリアします。 保持される状態データの詳細については、「[状態データの管理](bot-builder-nodejs-state.md)」を参照してください。

次のコード スニペットは、[endConversationAction][endConversationAction] をダイアログにバインドする方法を示しています。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>中断の確認

すべてではないにしても、これらのアクションのほとんどは会話の通常のフローを中断させます。 多くは中断を伴い、慎重に扱う必要があります。 たとえば、`triggerAction`、`cancelAction`、または `endConversationAction` はダイアログ スタックをクリアします。 ユーザーが誤ってこれらのアクションのいずれかをトリガーした場合、ユーザーはタスクをもう一度やり直す必要があります。 ユーザーが実際にこれらのアクションをトリガーする意図があったことを確認するために、`confirmPrompt` オプションをこれらのアクションに追加することができます。 `confirmPrompt` は、現在のタスクを本当にキャンセルまたは終了してもよいかユーザーに確認します。 これにより、ユーザーはアクションを取り消してプロセスを継続できます。

次のコード スニペットは、ユーザーが本当に注文プロセスをキャンセルしたいのか確認するための [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt) 付きの [cancelAction][cancelAction] を示しています。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

このアクションがトリガーされると、ユーザーに "Are you sure?"(本当にキャンセルしますか?) と確認します。 ユーザーは、アクションを本当に実行する場合は "Yes"、アクションを取り消して元の処理を続ける場合は "No" と答える必要があります。

## <a name="next-steps"></a>次の手順

**アクション**を使用して、ユーザーの要求を予測し、ボットでそれらの要求を適切に扱えるようにすることができます。 これらのアクションの多くは、現在の会話のフローを中断させます。 ユーザーが会話フローを中断および再開できるようにするには、中断する前にユーザー状態を保存する必要があります。ユーザー状態を保存、および状態データを管理する方法について理解を深めます。

> [!div class="nextstepaction"]
> [状態データの管理](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction
