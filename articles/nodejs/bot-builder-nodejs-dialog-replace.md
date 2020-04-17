---
title: ダイアログの置換 - Bot Service
description: Bot Framework SDK for Node.js を使用して会話フローの入力と管理を再確認するためのダイアログを置換する方法を説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 79083626f60564d8fa6a500cf7c45805e4488e22
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75790906"
---
# <a name="replace-dialogs"></a>ダイアログの置換

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ダイアログを置換する機能は、ユーザー入力を検証したり、会話中にアクションを繰り返したりする必要がある場合に便利です。 Bot Framework SDK for Node.js では、[`session.replaceDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) メソッドを使用してダイアログを置換できます。 このメソッドを使用すると、呼び出し元に戻らずに、現在のダイアログを終了して、新しいダイアログに置き換えることができます。 

## <a name="create-custom-prompts-to-validate-input"></a>カスタム プロンプトを作成して入力を検証する

Bot Framework SDK for Node.js には、`Prompts.time` や `Prompts.choice` など、一部の種類の[プロンプト](bot-builder-nodejs-dialog-prompt.md)に対する入力検証が含まれています。 `Prompts.text` に応答して受信するテキスト入力を検証するには、独自の検証ロジックとカスタム プロンプトを作成する必要があります。 

定義する特定の値、パターン、範囲、または条件に準拠する必要がある場合に、入力を検証することもできます。 入力の検証に失敗した場合、ボットでは、`session.replaceDialog` メソッドを使って、情報をもう一度入力するようユーザーに求めることができます。

次のコード サンプルは、ユーザーが入力した電話番号を検証するカスタム プロンプトを作成する方法を示しています。

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

この例では、ユーザーは最初に自分の電話番号を入力するように求められます。 検証ロジックにより、ユーザーが入力した数字の範囲に一致する正規表現が使用されます。 入力に 10 桁または 11 桁の数字が含まれている場合は、その数字が応答で返されます。 それ以外の場合は、`session.replaceDialog` メソッドが実行され、ユーザーに再入力を求める `phonePrompt` ダイアログが繰り返されます。このダイアログでは、必要な入力形式に関するより具体的なガイダンスが示されます。

`session.replaceDialog` メソッドを呼び出すときに、繰り返すダイアログの名前と、引数の一覧を指定します。 この例では、引数の一覧に `{ reprompt: true }` が含まれます。この引数によって、最初のプロンプトか再プロンプトかに応じて、異なるプロンプト メッセージがボットによって表示されますが、ご利用のボットで必要とされる可能性がある引数を指定することもできます。 

## <a name="repeat-an-action"></a>アクションを繰り返す

会話中、ユーザーが特定のアクションを複数回実行できるように、ダイアログを繰り返し表示したい場合があります。 たとえば、お使いのボットによってさまざまなサービスが提供される場合に、最初に表示するサービス メニューで、ユーザーがサービスの要求操作を行い、次に表示するサービス メニューで、ユーザーが別のサービスを要求できるようにする場合があります。 これを実現するには、['session.endConversation'](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) メソッドで会話を終了するのではなく、[`session.replaceDialog`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) メソッドを使用して、サービス メニューを再表示します。 

次の例は、`session.replaceDialog` メソッドを使用して、この種類のシナリオを実装する方法を示しています。 最初に、サービスのメニューが定義されます。

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

`mainMenu` ダイアログは既定のダイアログによって呼び出されるため、メニューは会話の開始時にユーザーに表示されます。 さらに、ユーザーが「メイン メニュー」と入力したときにも必ずメニューが表示されるように、[`triggerAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) が `mainMenu` ダイアログにアタッチされます。 表示されたメニューでユーザーがオプションを選択すると、そのユーザーの選択に対応するダイアログが、`session.beginDialog` メソッドを使用して呼び出されます。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

この例では、ユーザーがオプション 1 を選択して、ルーム サービスのディナーを注文すると、`orderDinner` ダイアログが呼び出されます。ユーザーは、ここでディナーの注文操作を行います。 操作の最後に、ボットによって注文が確認され、`session.replaceDialog` メソッドを使ってメイン メニューが再表示されます。

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

`orderDinner` ダイアログには、ユーザーが操作の途中でいつでも、注文を "最初からやり直す" か "取り消す" ことができるように 2 つのトリガーがアタッチされています。 

最初のトリガーは [`reloadAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction) です。これにより、ユーザーは「やり直し」と入力して送信することで、注文操作を最初からやり直すことができます。 トリガーが発話 "やり直し" と一致すると、`reloadAction` によってダイアログが最初から再開されます。 

2 番目のトリガーは [`cancelAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction) です。これにより、ユーザーは「取り消し」と入力して送信することで、注文操作を中止できます。 このトリガーでは、メイン メニューは自動的には表示されませんが、代わりに「"メイン メニュー" を入力して続行」と言うことで、次の操作をユーザーに示すメッセージが送信されます。

### <a name="dialog-loops"></a>ダイアログをループさせる

上記の例でユーザーが選択できる項目は注文ごとに 1 つだけです。 つまり、ユーザーがメニューから 2 つの項目を注文したい場合は、最初の項目についてすべての注文操作を完了してから、2 つ目の項目に対して、注文操作をもう一度最初から最後まで繰り返さなければなりません。 

次の例は、別のダイアログに進むようにディナー メニューをリファクタリングすることにより、前のボットを改善する方法を示しています。 これを行うことで、ボットによってループ内でディナー メニューを繰り返すことができ、ユーザーは 1 回の注文で複数の項目を選択できます。

最初に、メニューに "チェック アウト" オプションを追加します。 このオプションを使用すると、ユーザーは項目の選択プロセスを終了し、チェック アウト処理を続行できます。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

次に、ボットでメニューを繰り返し表示できるように、また、ユーザーが複数の項目を注文に追加できるように、注文のプロンプトが独自のダイアログに進むようにリファクタリングします。

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

この例では、現在の会話 `session.conversationData.orders` にスコープが限定されたボット データ ストアに、注文が格納されています。 変数は新規注文ごとに新しい配列で再初期化されます。また、ユーザーが選択するすべての項目が、ボットによってそれぞれ `orders` 配列に追加され、価格は、チェックアウトの `Price` 変数に格納されている合計に追加されます。 ユーザーが自分の注文の項目を選択し終えたら、"チェック アウト" と言えば、残りの注文操作を続行できます。

> [!NOTE]
> ボット データ ストレージの詳細については、「[Manage state data (状態データの管理)](bot-builder-nodejs-state.md)」を参照してください。 

最後に、`orderDinner` ダイアログ内にあるウォーターフォールの 2 番目の手順を更新し、確認によって注文を処理します。 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>ダイアログを取り消す

`session.replaceDialog` メソッドを使用すると "*現在の*" ダイアログを新しいダイアログに置き換えることができますが、ダイアログ スタックの下の方にあるダイアログを置換することはできません。 ダイアログ スタック内の "*現在の*" ダイアログではないダイアログを置換するには、代わりに [`session.cancelDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) メソッドを使用します。 

`session.cancelDialog` メソッドを使用すると、ダイアログ スタックにおけるダイアログの場所に関係なく、ダイアログを終了できるほか、必要に応じて、代わりの新しいダイアログを呼び出すことができます。 `session.cancelDialog` メソッドを呼び出すには、取り消すダイアログの ID を指定し、必要に応じて、代わりの呼び出すダイアログの ID を指定します。 たとえば、次のコード スニペットでは `orderDinner` ダイアログが取り消され、これが `mainMenu` ダイアログで置き換えられます。

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

`session.cancelDialog` メソッドが呼び出されると、ダイアログ スタックが後方検索され、最初に検出されたダイアログが取り消されます。これにより、そのダイアログとその子ダイアログ (存在する場合) がダイアログ スタックから削除されます。 その後、制御が呼び出し元ダイアログに返されます。これにより、[`ResumeReason.notCompleted`](http://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted) と同じ [`results.resumed`](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) コードがないかどうかをチェックして、取り消しを検出できます。

`session.cancelDialog` メソッドを呼び出すとき、取り消すダイアログの ID を指定する代わりに、取り消すダイアログの 0 から始まるインデックスを指定できます。ここではインデックスはダイアログ スタックにおけるダイアログの位置を表します。 たとえば、次のコード スニペットは、現在アクティブなダイアログ (インデックス = 0) を終了し、`mainMenu` ダイアログを代わりに起動します。 `mainMenu` ダイアログはダイアログ スタックの位置 0 で呼び出されるため、新しい既定のダイアログになります。

```javascript
session.cancelDialog(0, 'mainMenu');
```

上記の「[ダイアログ ループ](#dialog-loops)」で取り上げたサンプルについて考えましょう。 ユーザーが項目選択メニューに達したとき、そのダイアログ (`addDinnerItem`) は、ダイアログ スタック内の 4 番目のダイアログです (`[default dialog, mainMenu, orderDinner, addDinnerItem]`)。 `addDinnerItem` 内からユーザーが自分の注文を取り消せるようにするにはどすればよいでしょうか。 `cancelAction` トリガーを `addDinnerItem` ダイアログにアタッチすると、ユーザーは前のダイアログ (`orderDinner`) に戻されるだけです。これでは、ユーザーが再度 `addDinnerItem` ダイアログに戻ってしまいます。

`session.cancelDialog` メソッドが役に立つのはここです。 [ダイアログのループの例](#dialog-loops)から始まって、"注文の取り消し" を明示的なオプションとしてディナー メニューに追加します。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

次に、"注文の取り消し" 要求が確認されるように `addDinnerItem` ダイアログを更新します。 "取り消し" が検出された場合は、`session.cancelDialog` メソッドを使用して既定のダイアログ (スタックのインデックス 0 にあるダイアログ) を取り消し、代わりに `mainMenu`ダイアログを呼び出します。 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

このように `session.cancelDialog` メソッドを使用すると、お使いのボットに必要なすべての会話フローを実装することができます。

## <a name="next-steps"></a>次のステップ

おわかりのように、スタックでダイアログを置換するには、さまざまな種類の**アクション**を使用してそのタスクを実行します。 アクションによって、会話フローを柔軟に管理できるようになります。 **アクション**、およびユーザー アクションをより効果的に処理する方法について見ていきましょう。

> [!div class="nextstepaction"]
> [ユーザー アクションを処理する](bot-builder-nodejs-dialog-actions.md)
