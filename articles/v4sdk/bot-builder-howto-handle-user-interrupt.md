---
title: ユーザーによる割り込みを処理する | Microsoft Docs
description: ユーザーによる割り込みを処理し、会話フローを転送する方法について説明します。
keywords: 割り込む, 割り込み, トピックの切り替え, 中断
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fff4f8e2a4d2d86cf440bee7ab40216e93a8c8c5
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756749"
---
# <a name="handle-user-interruptions"></a>ユーザーによる割り込みを処理する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーによる割り込みの処理は、堅牢なボットの重要な側面です。 自分では対象のユーザーが定義済み会話フローの手順に従っていると思っていても、ユーザーの気が変わったり、プロセスの途中で質問に答える代わりに、質問してきたりする可能性は大いにあります。 このような場合、お使いのボットでは、ユーザーの入力がどのように処理されるのでしょうか。 ユーザー エクスペリエンスはどのようになるのでしょうか。 ユーザーの状態データはどのように維持されるのしょうか。

これらの質問に正解はありません。各状況が、お使いのボットが処理するように設計されたシナリオに固有であるためです。 このトピックでは、ユーザーによる割り込みの一般的な処理方法をいくつか取り上げて、それをご自身のボットで実装する方法を提案します。

## <a name="handle-expected-interruptions"></a>予期される割り込みの処理

手続き型会話フローに含まれるコア ステップに従うようにユーザーを導く必要があり、こうしたステップと異なるユーザー アクションはすべて、割り込みになる可能性があります。 通常のフローには、予測できる割り込みがいくつかあります。

**レストラン予約**: レストラン予約ボットのコア ステップには、ユーザーに日時、人数、および予約名を聞く、といった動作があります。 このプロセスでは、次のような割り込みを予測できます。

* `cancel`: プロセスを終了する。
* `help`: このプロセスに関する追加ガイダンスを提供する。
* `more info`: ヒント/提案、または別の予約方法を提供する (連絡先の電子メール アドレス、電話番号など)。
* `show list of available tables`:(そのようなオプションがある場合) ユーザーが希望する日時に予約できるレストランの一覧を表示する。

**ディナーの注文**: ディナー注文ボットのコア ステップには、メニュー項目の一覧を示す、ユーザーが自分のカートに項目を追加できるようにする、などの動作があります。 このプロセスでは、次のような割り込みを予測できます。

* `cancel`: 注文の処理を終了する。
* `more info`: 各メニュー項目の食に関する詳細情報を提供する。
* `help`: システムの使用方法に関するヘルプを提供する。
* `process order`: 注文を処理する。

これらの情報は、**推奨されるアクション**の一覧またはヒントとしてユーザーに示すことができます。これによりユーザーは少なくともどのコマンドがボットによって認識され、ユーザーが送信できるかを把握できます。

たとえば、ディナー注文フローでは、メニュー項目と共に、予測される割り込みを提供できます。 この場合、メニュー項目は `choices` の配列として送信されます。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
            "more info", "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

ご利用の注文ロジックでは、文字列の一致または正規表現を使用してそれらを確認できます。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

まず、注文が追跡されるようにヘルパーを定義する必要があります

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

次に、ダイアログをご自身のボットに追加します。

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");

                // In production, you may want to store something more helpful,
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");

                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);

            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);

            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);
```

---

## <a name="handle-unexpected-interruptions"></a>予期されない割り込みの処理

ご自身のボットが実行するように設計されたもの以外の割り込みも発生します。
すべての割り込みを予期することは無理ですが、ご自身のボットが処理できるようにプログラミングできる割り込みのパターンはいくつかあります。

### <a name="switching-topic-of-conversations"></a>会話のトピックの切り替え

ユーザーがある会話の途中で、別の会話に切り替えたくなったらどうなるでしょうか。 たとえば、お使いのボットでレストランの予約と、ディナーの注文ができるとします。
_レストラン予約_フローで、ユーザーは "予約の人数は何人ですか" という質問に答えず、"ディナーを注文する" というメッセージを送信します。 この場合、ユーザーは途中で気が変わり、ディナー注文の会話に参加したくなったのです。 この割り込みはどのように処理すべきでしょうか。

トピックをディナー注文フローに切り替えることも、元の話題にとどまり、人数を答える必要があることをユーザーに伝え、入力を促すこともできます。 トピックを切り替えることを許可する場合は、ユーザーが中断した場所から後で作業を続行できるように操作内容を保存するか、入力されたすべての情報を削除するかを決めなければなりません。削除した場合、ユーザーは次回予約するときに、すべての操作を最初からやり直す必要があります。 ユーザー状態データの管理の詳細については、「[Save state using conversation and user properties (会話およびユーザー プロパティを使用した状態の保存)](bot-builder-howto-v4-state.md)」を参照してください。

### <a name="apply-artificial-intelligence"></a>人工知能を適用する

予想外の割り込みの場合は、ユーザーの意図の推測を試みることができます。 これを行うには、QnAMaker、LUIS、ご自身のカスタム ロジックなどの AI サービスを使用します。そして、ユーザーが求めているものに関するボットの推測を提案します。

たとえば、レストラン予約フローの途中で、ユーザーが "ハンバーガーを注文したい" と言ったとします。 これは、ボットがこの会話フローから処理方法を判断できるものではありません。 現在のフローは注文とは何の関係もありません。ボットのもう一方の会話コマンドは "ディナーを注文" です。このため、ボットでは、この入力をどのように処理すべきかを判断できません。 たとえば、LUIS を適用すると、ユーザーが食べ物を注文したがっていることが認識されるようにモデルをトレーニングできます (例: LUIS で "orderFood" 意図を返すことができます)。 これにより、"食べ物を注文されたいようですね。 ディナー注文プロセスに切り替えますか" という応答を、ボットで返すことができます。 LUIS のトレーニングとユーザー意図の検出の詳細については、[言語理解のための LUIS の使用](bot-builder-howto-v4-luis.md)に関するページをご覧ください。

### <a name="default-response"></a>既定の応答

その他すべてが失敗した場合は、何も行わない、あるいは何が起こっているかとユーザーに思わせたままにするのではなく、汎用的な既定の応答を送信できます。 既定の応答では、ユーザーが本来の会話に戻ることができるように、ボットで認識されるコマンドをユーザーに伝える必要があります。

ボット ロジックの末尾にあるコンテキスト**応答**フラグでは、ターンの途中にボットによってユーザーに何か送り返されていないかどうかを確認することができます。 ボットによってユーザーの入力が処理されているにもかかわらず、応答がない場合は、ボットではその入力の扱い方を判断できない可能性があります。 この場合は、それをキャッチして、既定のメッセージをユーザーに送信できます。

このボットでは、既定で `mainMenu` ダイアログがユーザーに表示されます。 ボットでこのような状況が発生したときのユーザーのエクスペリエンスは、ご自身で決めることができます。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---
