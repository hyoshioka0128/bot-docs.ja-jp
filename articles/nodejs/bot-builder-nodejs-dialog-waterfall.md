---
title: ウォーターフォールを使用した会話のステップの定義 | Microsoft Docs
description: Bot Builder SDK for Node.js でウォーターフォールを使用して会話のステップを定義する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ac8e01a81e3f8acca3aa6976eec47017876c3042
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302268"
---
# <a name="define-conversation-steps-with-waterfalls"></a>ウォーターフォールを使用した会話のステップの定義

会話とは、ユーザーとボットの間で交換される一連のメッセージのことです。 ボットの目的が一連のステップでユーザーを誘導することにある場合は、ウォーターフォールを使用して会話のステップを定義できます。

[ウォーターフォール](bot-builder-nodejs-dialog-overview.md)はダイアログ特有の実装であり、ユーザーから情報を収集したり、一連のタスクをユーザーに案内したりするときに最も一般的に使用されます。 タスクは、最初の関数の結果が次の関数の入力として渡される、関数の配列として実装されます。 各関数は、通常、プロセス全体の 1 つのステップを表します。 各ステップで、ボットはユーザーに入力を要求し、応答を待って、その結果を次のステップに渡します。

この記事は、ウォーターフォールのしくみと、ウォーターフォールを使用して[会話フローを管理する](bot-builder-nodejs-dialog-manage-conversation.md)方法を理解するのに役立ちます。

## <a name="conversation-steps"></a>会話のステップ

通常、会話では、ユーザーとボットの間でプロンプト/応答が何回か交換されます。 それぞれのプロンプト/応答の交換は、会話を進める 1 ステップです。 会話は、わずか 2 ステップのウォーターフォールを使用して作成することができます。

たとえば、次の `greetings` ダイアログを考えてみましょう。 ウォーターフォールの最初のステップでユーザーの名前を要求し、2 番目のステップでその応答を使用してユーザーを名前で迎えます。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

これを可能にしているのが、プロンプトの使用です。 Bot Builder SDK for Node.js には、ユーザーにさまざまな種類の情報を要求するための各種の組み込み[プロンプト](bot-builder-nodejs-dialog-prompt.md)が用意されています。

次のサンプル コードは、プロンプトを使用して 4 ステップのウォーターフォール全体でユーザーからさまざまな情報を収集するダイアログを示しています。

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
        builder.Prompts.text(session, "How many people are in your party?");
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

この例では、既定のダイアログに、それぞれがウォーターフォールのステップを表す 4 つの関数があります。 各ステップは、ユーザーに入力を要求し、その結果を次に処理されるステップに送信します。 このプロセスは最後のステップが実行されるまで続き、最終的に予約を確認してダイアログを終了します。

次のスクリーンショットは、[Bot Framework Emulator](~/bot-service-debug-emulator.md) で実行されたこのボットの結果を示しています。

![ウォーターフォールを使用して会話フローを管理する](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>複数のウォーターフォールを使用して会話を作成する

会話内に複数のウォーターフォールを使用することで、ボットに必要などのような会話構造も定義することができます。 たとえば、1 つのウォーターフォールを使用して会話フローを管理し、追加のウォーターフォールを使用してユーザーから情報を収集することができます。 各ウォーターフォールは、ダイアログにカプセル化され、`session.beginDialog` 関数を呼び出すことによって呼び出すことができます。

会話内で複数のウォーターフォールを使用する方法を次のサンプル コードに示します。 `greetings` ダイアログ内のウォーターフォールは会話のフローを管理し、`askName` ダイアログ内のウォーターフォールはユーザーから情報を収集します。

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>ウォーターフォールのステップを進める

ウォーターフォールのステップは、配列内に定義されている関数の順に進行します。 ウォーターフォール内の最初の関数は、ダイアログに渡された引数を受け取ることができます。 ウォーターフォール内のすべての関数は、`next` 関数を使用して、ユーザーに入力を要求することなく次のステップに進むことができます。

次のサンプル コードは、ユーザーにユーザー プロファイルの情報を入力してもらうためのプロセスを進めるダイアログ内で `next` 関数を使用する方法を示しています。 ボットは、(必要に応じて) ウォーターフォールの各ステップでユーザーに情報の入力を要求します。ユーザーの応答がある場合、応答はウォーターフォールの後続のステップで処理されます。 `ensureProfile` ダイアログのウォーターフォールの最後のステップでダイアログが終了し、入力されたプロファイル情報が呼び出し元のダイアログに返されます。この呼び出し元のダイアログから、ユーザーにパーソナライズされた挨拶が送信されます。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> この例では、データを格納するために、`dialogData` と `userData` の 2 つの異なるデータ バッグを使用しています。 ボットが複数の計算ノードに分散されている場合、ウォーターフォールの各ステップが異なるノードによって処理される可能性があります。 ボット データの保存の詳細については、「[Manage state data (状態データの管理)](bot-builder-nodejs-state.md)」を参照してください。

## <a name="end-a-waterfall"></a>ウォーターフォールを終了する

ウォーターフォールを使用して作成したダイアログは明示的に終了する必要があります。そうでないと、ボットはウォーターフォールを無制限に繰り返します。 ウォーターフォールは、次のメソッドのいずれかを使用して終了できます。

* `session.endDialog`: 呼び出し元のダイアログに返すデータがない場合は、このメソッドを使用してウォーターフォールを終了します。

* `session.endDialogWithResult`: 呼び出し元のダイアログに返すデータがある場合は、このメソッドを使用してウォーターフォールを終了します。 返される `response` 引数は、JSON オブジェクトまたは任意の JavaScript プリミティブ データ型です。 例: 
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`: ウォーターフォールの終了が会話の終了を表す場合は、このメソッドを使用してウォーターフォールを終了します。

これらの 3 つのメソッドのいずれかを使用してウォーターフォールを終了する代わりに、`endConversationAction` トリガーをダイアログにアタッチすることができます。 例: 

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>次の手順

ウォーターフォールを使用すると、"*プロンプト*" を使用してユーザーから情報を収集できます。 ユーザーに入力を要求する方法について詳しく見ていきましょう。

> [!div class="nextstepaction"]
> [ユーザーに入力を要求する](bot-builder-nodejs-dialog-prompt.md)
