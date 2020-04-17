---
title: ダイアログで会話フローを管理する - Bot Service
description: Bot Framework SDK for Node.js のダイアログを使用してボットとユーザー間の会話を管理する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cd16586ec6e412d37f34bbe8784fdc86e837d9b7
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791134"
---
# <a name="manage-conversation-flow-with-dialogs"></a>ダイアログを使用して会話フローを管理する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

会話フローの管理は、ボットを構築する上で最も重要なタスクです。 ボットは、主要なタスクを的確に実行し、中断を適切に処理できなければなりません。 Bot Framework SDK for Node.js を使用することで、ダイアログを使用して会話フローを管理することができます。

ダイアログは、プログラムの関数に似ています。 それは、一般的に特定の操作を実行するように設計されており、必要に応じて何度でも呼び出すことができます。 複数のダイアログを連鎖させて、ボットで処理するほぼすべての会話フローを処理できます。 Bot Framework SDK for Node.js には、[プロンプト](bot-builder-nodejs-dialog-prompt.md)と[ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md)のような組み込み機能があり、会話の流れを管理するのに役立ちます。

この記事では、ダイアログを使用してボットに中断を処理させたり適切にフローを再開させたりすることのできる、簡単な会話フローと複雑な会話フローの両方を管理する方法について、いくつかの例で説明します。 例は、次のシナリオに基づいています。 

1. ボットがディナー予約をします。
2. ボットは、予約中にいつでも "ヘルプ" 要求を処理できます。
3. ボットは、予約の現在のステップの状況依存の "ヘルプ" を処理できます。
4. ボットは、複数のトピックの会話を処理できます。

## <a name="manage-conversation-flow-with-a-waterfall"></a>ウォーターフォールで会話フローを管理する

[ウォーターフォール](bot-builder-nodejs-dialog-waterfall.md)というダイアログによって、ボットは一連のタスクをユーザーに簡単に実行させることができます。 この例では、予約ボットが予約要求を処理できるように、ユーザーに一連の質問を行います。 ボットは、ユーザーに次の情報の入力を求めます。

1. 予約の日付と時刻
2. パーティーの人数
3. 予約者の名前

次のコード サンプルは、ウォーターフォールを使用してユーザーを一連のプロンプトに誘導する方法を示しています。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.number(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;
        
        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

このボットの基本的な機能は、既定のダイアログに表示されます。 既定のダイアログは、ボットの作成時に定義されます。 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

また、ボットの作成プロセス中に、使用する[データ ストレージ](bot-builder-nodejs-state.md)を設定できます。 たとえば、メモリ内ストレージを使用するには次のように設定します。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

既定のダイアログは、ウォーターフォールのステップを定義する関数の配列として作成されます。 この例では 4 つの関数があり、ウォーターフォールには 4 つのステップがあります。 各ステップは 1 つのタスクを実行し、その結果は次のステップで処理されます。 このプロセスは最後のステップまで続き、最後のステップで予約が確定してダイアログが終了します。

次のスクリーン ショットは、[Bot Framework Emulator](../bot-service-debug-emulator.md) で実行されているこのボットの結果を示しています。

![ウォーターフォールを使用して会話フローを管理する](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>ユーザーに入力を要求する

この例の各ステップでは、プロンプトを使用して、ユーザー入力を要求しています。 プロンプトは、ユーザー入力を要求し、応答を待ってから、ウォーターフォールの次のステップへの応答を返す特殊な種類のダイアログです。 ボットで使用できるさまざまな種類のプロンプトについては、[ユーザーに入力を要求する](bot-builder-nodejs-dialog-prompt.md)を参照してください。

この例では、ボットは `Prompts.text()` を使用して、ユーザーからの自由形式の応答をテキスト形式で要求します。 ユーザーは任意のテキストで応答することができ、ボットは応答を処理する方法を決定する必要があります。 `Prompts.time()` は [Chrono](https://github.com/wanasit/chrono) ライブラリを使用して文字列から日付と時刻の情報を解析します。 これにより、ボットは日付と時刻を指定するための自然言語をより理解することができます。 次に例を示します。"2017 年 6 月 6 日の午後 9 時"、"今日の午後 7 時 30 分"、"次の月曜日の午後 6 時" といった具合です。

> [!TIP] 
> ユーザーが入力した時刻は、ボットがホストされているサーバーのタイム ゾーンに基づいて UTC 時間に変換されます。 サーバーは、ユーザーとは別のタイム ゾーンに配置されている場合があるため、必ずタイム ゾーンを考慮してください。 日付と時刻をユーザーのローカル時刻に変換するには、ユーザーが属しているタイム ゾーンを確認することを検討してください。

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>複数のダイアログで会話フローを管理する

会話フローを管理するためのもう 1 つの手法は、ウォーターフォールと複数のダイアログの組み合わせを使用することです。 ウォーターフォールを使用すると、ダイアログで機能を連動させることができます。一方ダイアログを使用すると、会話をいつでも再利用できる小さな機能に分割できます。

たとえば、ディナー予約のボットを考えてみましょう。 次のコード サンプルは、ウォーターフォールと複数のダイアログを使用するように、以前の例を書き直したものです。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

このボットの実行結果は、ウォーターフォールだけが使用されていた以前のボットとまったく同じです。 ただし、プログラムとしては、次の 2 つの根本的な違いがあります。

1. 既定のダイアログは、会話フローを管理するためのものです。
2. 会話の各ステップのタスクは、別のダイアログによって管理されています。 この場合、ボットには 3 つの情報が必要であったため、ユーザーに 3 回ダイアログを表示します。 各プロンプトは、それ自身のダイアログ内に含まれています。

この手法では、タスクのロジックから会話フローを分離できます。 これにより、必要に応じて別の会話フローによってダイアログを再利用することができます。 

## <a name="respond-to-user-input"></a>ユーザー入力に応答する

一連のタスクにユーザーを誘導する過程で、ユーザーが質問をしたり、回答前に別の情報を要求したりする場合、そうした要求をどのように処理しますか? たとえば、ユーザーが会話に入っている状況に関係なく、ユーザーが「ヘルプ」、「サポート」、または「キャンセル」を入力した場合、ボットはどのように応答しますか? ユーザーがステップに関する別の情報が必要である場合はどうなりますか? ユーザーの気が変わり、現在のタスクを破棄して全く別のタスクを開始する場合はどうなりますか?

Bot Framework SDK for Node.js により、ボットでは現在のダイアログのスコープのグローバル コンテキスト内またはローカル コンテキスト内の特定の入力をリッスンできます。 こうした入力は[アクション](bot-builder-nodejs-dialog-actions.md)と呼ばれ、アクションによってボットは `matches` 句に基づいてユーザー入力をリッスンすることができます。 特定のユーザー入力にどのように対応するかを決定するのはボット次第です。

### <a name="handle-global-action"></a>グローバル アクションを処理する

ボットが会話の任意の時点でアクションを処理できるようにするには、`triggerAction` を使用します。 トリガーを使用すると、入力が指定された用語に一致するときにボットが特定のダイアログを呼び出すことができます。 たとえば、グローバル "ヘルプ" オプションをサポートする場合は、ヘルプ ダイアログを作成し、"ヘルプ" に一致する入力をリッスンする `triggerAction` を添付することができます。

次のコード サンプルは、`triggerAction` をダイアログに添付して、ユーザーが「ヘルプ」と入力したときにダイアログを呼び出すように指定する方法を示しています。

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

既定では、`triggerAction` が実行されると、ダイアログ スタックが消去され、トリガーされたダイアログが新しい既定のダイアログになります。 この例では、`triggerAction` が実行されるとダイアログ スタックが消去され、`help` ダイアログが新しい既定のダイアログとしてスタックに追加されます。 これが目的の動作でない場合は、`onSelectAction` オプションを `triggerAction` に追加できます。 `onSelectAction` オプションは、ボットがダイアログ スタックを消去せずに新しいダイアログを開始できるようにします。これにより、会話は一時的にリダイレクトされ、後で中断したところから再開できます。

次のコード サンプルは、`triggerAction` で `onSelectAction` オプションを使用して、`help` ダイアログを既存のダイアログ スタックに追加する方法を示しています (ダイアログ スタックは消去されません)。

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

この例では、ユーザーが「ヘルプ」と入力すると、`help` ダイアログが会話を制御します。 `triggerAction` に `onSelectAction` オプションが含まれているため、`help` ダイアログは既存のダイアログ スタックにプッシュされ、スタックは消去されません。 `help` ダイアログが終了すると、ダイアログ スタックから削除され、`help` コマンドで中断されたところから会話が再開されます。

### <a name="handle-contextual-action"></a>コンテキスト アクションを処理する

前の例では、ユーザーが会話の任意の時点で「ヘルプ」と入力すると、`help` ダイアログが呼び出されました。 したがって、ユーザーのヘルプ要求の具体的なコンテキストがないため、ダイアログには一般的なヘルプのガイダンスのみが表示されます。 しかし、ユーザーが会話の特定のポイントに関するヘルプを要求したい場合はどうなるでしょうか? その場合は、現在のダイアログのコンテキスト内で `help` ダイアログをトリガーする必要があります。

たとえば、ディナー予約のボットを考えてみましょう。 パーティーの参加者数を尋ねられ、パーティーの最大規模を知りたい場合はどうなるでしょう? このシナリオに対処するには、`askForPartySize` ダイアログに `beginDialogAction` を添付して、ユーザーが「ヘルプ」と入力するのをリッスンします。

次のコード サンプルは、`beginDialogAction`を使用して状況依存のヘルプをダイアログに添付する方法を示しています。

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

この例では、ユーザーが「ヘルプ」と入力するたびに、ボットが `partySizeHelp` ダイアログをスタックにプッシュします。 そのダイアログは、ヘルプ メッセージをユーザーに送信し、ダイアログを終了し、制御を `askForPartySize` ダイアログに戻し、ユーザーに対してパーティーの規模をもう一度要求します。

この状況依存のヘルプは、ユーザーが `askForPartySize` ダイアログ内にいるときにのみ実行されることに留意することが重要です。 それ以外のときは、`triggerAction` からの一般的なヘルプ メッセージが代わりに実行されます。 つまり、ローカルの `matches` 句はグローバルの `matches` 句よりも常に優先されます。 たとえば、`beginDialogAction` が **ヘルプ** と一致する場合、`triggerAction` の **ヘルプ** との一致は実行されません。 詳細については、[アクションの優先順位](bot-builder-nodejs-dialog-actions.md#action-precedence)に関する記事をご覧ください。

### <a name="change-the-topic-of-conversation"></a>会話のトピックを変更する

既定では、`triggerAction` を実行するとダイアログ スタックが消去されて会話がリセットされ、指定されたダイアログから開始します。 この動作は、ボットが会話のトピックを別のトピックに切り替える必要がある場合によく使用されます。たとえば、ディナー予約を取っているユーザーが、ホテルの部屋にディナーの配達を注文するよう決定する場合などです。 

次のサンプル ビルドは以前の例に基づいており、ボットによってユーザーがディナーを予約したり、ディナーの配達を注文したりすることができるようになっています。 このボットでは、既定のダイアログは、`Dinner Reservation` と `Order Dinner` の 2 つのオプションをユーザーに提示するグリーティング ダイアログです。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

前の例の既定のダイアログにあったディナー予約のロジックは、`dinnerReservation` という独自のダイアログに含まれることになりました。 `dinnerReservation` のフローは、前に説明した複数ダイアログのバージョンと変わりません。 唯一の違いは、ダイアログに `triggerAction` が添付されていることです。 このバージョンでは、`confirmPrompt` は、新しいダイアログを呼び出す前に、会話のトピックを変更するかどうか確認するようにユーザーに要求します。 ダイアログ スタックがクリアされると、ユーザーは新しい会話トピックに誘導され、進行中の会話トピックは破棄されるため、こうしたシナリオでは `confirmPrompt` を含めることをお勧めします。

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

2 つ目の会話トピックは、`orderDinner` ダイアログでウォーターフォールを使用して定義されます。 このダイアログは、単にディナー メニューを表示し、注文が指定されたら部屋番号を入力するようユーザーに表示します。 ダイアログに `triggerAction` が添付され、ユーザーが「ディナー注文」と入力したときにダイアログが呼び出されるように指定されます。また、会話のトピックを変更したいことを示す場合は、ユーザーが選択を確認するように要求されます。

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

ユーザーが会話を開始し、`Dinner Reservation` か `Order Dinner` のいずれかを選択すると、いつでも考えを変えることができます。 たとえば、ユーザーがディナー予約の途中で「ディナー注文」と入力した場合、ボットは「現在の要求をキャンセルします。 よろしいですか?」と確認します。 ユーザーが「いいえ」と入力すると、要求はキャンセルされ、ユーザーはディナー予約プロセスを続行できます。 ユーザーが「はい」と入力すると、ボットはダイアログ スタックをクリアし、会話の制御を `orderDinner` ダイアログに移します。

## <a name="end-conversation"></a>会話を終了する

上記の例では、ダイアログは `session.endDialog` または `session.endDialogWithResult` のいずれかによって閉じられます。どちらもダイアログを終了し、スタックからダイアログを削除して、呼び出しダイアログに制御を戻します。 ユーザーが会話の終わりに達した場合は、会話が終了したことを示すために `session.endConversation` を使用する必要があります。

[`session.endConversation`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) メソッドは会話を終了し、必要に応じてメッセージをユーザーに送信します。 たとえば、前の例の `orderDinner` ダイアログでは、次のコード サンプルで示すように、`session.endConversation`を使用して会話を終了できます。

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

`session.endConversation` を呼び出すと、ダイアログ スタックがクリアされ、[`session.conversationData`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata) ストレージがリセットされて会話が終了します。 データ ストレージの詳細については、「[Manage state data (状態データの管理)](bot-builder-nodejs-state.md)」を参照してください。

`session.endConversation` の呼び出しは、ボットが設計された目的の会話フローをユーザーが完了した場合の適切な動作です。 また、会話中にユーザーが「キャンセル」または「さようなら」と入力した場合に、`session.endConversation` を使用して会話を終了することもできます。 これは、ダイアログに `endConversationAction` を添付し、このトリガーに「キャンセル」または「さようなら」に一致する入力をリッスンさせるだけで実行できます。

次のコード サンプルは、ユーザーが「キャンセル」または「さようなら」を入力した場合に会話を終了するため、ダイアログに `endConversationAction` を添付する方法を示しています。

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

`session.endConversation` または `endConversationAction` で会話を終了すると、ダイアログ スタックがクリアされ、ユーザーは強制的に最初からやり直しとなるので、ユーザーが本当に終了したいかどうかを確認するために、`confirmPrompt` を含める必要があります。

## <a name="next-steps"></a>次のステップ

この記事では、性質上連続する会話の管理方法について説明しています。 会話内でダイアログを繰り返したり、ループ パターンを使用する場合はどうすればよいでしょうか? スタックのダイアログ ボックスを置き換えることでそれを実行する方法を見てみましょう。

> [!div class="nextstepaction"]
> [ダイアログの置換](bot-builder-nodejs-dialog-replace.md)

