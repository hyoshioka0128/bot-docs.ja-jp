---
title: ユーザーによる割り込みを処理する | Microsoft Docs
description: ユーザーによる割り込みを処理し、会話フローを転送する方法について説明します。
keywords: 割り込む, 割り込み, トピックの切り替え, 中断
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/20/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 74d4bb07274643d61da332d6ee1cdfb1a14372dc
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998429"
---
# <a name="handle-user-interruptions"></a>ユーザーによる割り込みを処理する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

割り込みの処理は、堅牢なボットの重要な側面です。

自分では対象のユーザーが定義済み会話フローの手順に従っていると思っていても、ユーザーの気が変わったり、プロセスの途中で質問に答える代わりに、質問してきたりする可能性は大いにあります。 このような場合、お使いのボットでは、ユーザーの入力がどのように処理されるのでしょうか。 ユーザー エクスペリエンスはどのようになるのでしょうか。 ユーザーの状態データはどのように維持されるのしょうか。 割り込みの処理は、ボットがこのような状況に対処できるよう確実に準備されていることを意味します。

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

ダイアログ セットを **DialogSet** のサブクラスとして定義します。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

メニューを記述するために、内部クラスをいくつか定義します。

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

基本的な EchoBot テンプレートから始めます。 手順については、[JavaScript のクイック スタート](~/javascript/bot-builder-javascript-quickstart.md)をご覧ください。

`botbuilder-dialogs` ライブラリは、NPM からダウンロードできます。 `botbuilder-dialogs` ライブラリをインストールするには、次の npm コマンドを実行します。

```cmd
npm install --save botbuilder-dialogs
```

**bot.js** ファイルで、参照するクラスを参照し、ダイアログに使用する識別子を定義します。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

注文ダイアログに表示するオプションを定義します。

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

ご利用の注文ロジックでは、文字列の一致または正規表現を使用してそれらを確認できます。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

まず、注文が追跡されるようにヘルパーを定義する必要があります。

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

必要な ID を追跡するために、定数をいくつか追加します。

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

コンストラクターを更新して、選択プロンプトとウォーターフォール ダイアログを追加します。 また、ウォーターフォール ステップを実装するメソッドも定義します。

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

ボット コンストラクターでダイアログを定義します。 このコードでは "_最初に_" 割り込みの有無の確認と処理を行ってから、次の論理ステップに進むことに注意してください。

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

ダイアログを呼び出し、注文処理の結果を表示するようにオン ターン ハンドラーを更新します。

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
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

予想外の割り込みの場合は、ユーザーの意図の推測を試みることができます。 これを行うには、QnA Maker、LUIS、ご自身のカスタム ロジックなどの AI サービスを使用します。そして、ユーザーが求めているものに関するボットの推測を提案します。

たとえば、レストラン予約フローの途中で、ユーザーが "ハンバーガーを注文したい" と言ったとします。 これは、ボットがこの会話フローから処理方法を判断できるものではありません。 現在のフローは注文とは何の関係もありません。ボットのもう一方の会話コマンドは "ディナーを注文" です。このため、ボットでは、この入力をどのように処理すべきかを判断できません。 たとえば、LUIS を適用すると、ユーザーが食べ物を注文したがっていることが認識されるようにモデルをトレーニングできます (例: LUIS で "orderFood" 意図を返すことができます)。 これにより、"食べ物を注文されたいようですね。 ディナー注文プロセスに切り替えますか" という応答を、ボットで返すことができます。 LUIS のトレーニングとユーザー意図の検出の詳細については、[言語理解のための LUIS の使用](bot-builder-howto-v4-luis.md)に関するページをご覧ください。

### <a name="default-response"></a>既定の応答

その他すべてが失敗した場合は、何も行わない、または何が起こっているかとユーザーに思わせたままにするのではなく、既定の応答を送信する必要があります。 既定の応答では、ユーザーが本来の会話に戻ることができるように、ボットで認識されるコマンドをユーザーに伝える必要があります。

ボット ロジックの末尾にあるコンテキスト**応答**フラグでは、ターンの途中にボットによってユーザーに何か送り返されていないかどうかを確認することができます。 ボットによってユーザーの入力が処理されているにもかかわらず、応答がない場合は、ボットではその入力の扱い方を判断できない可能性があります。 この場合は、それをキャッチして、既定のメッセージをユーザーに送信できます。

ボットでは既定で `mainMenu` ダイアログがユーザーに表示される場合、 ボットでこのような状況が発生したときのユーザーのエクスペリエンスは、ご自身で決めることができます。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>グローバルな割り込みの処理

上記の例では、特定のダイアログ内において特定のターンで発生する可能性のある割り込みを処理しています。 グローバルな割り込み (いつでも発生する可能性のある割り込み) を処理したい場合はどうなるでしょうか。

これは、ボットのメイン ハンドラーに割り込み処理ロジック (受信 `turnContext` を処理し、それにどう対処するかを決定する関数) を配置することで実現できます。

以下の例では、ボットが実行する "_最初の処理_" として、受信メッセージのテキストを調べて、ユーザーがヘルプを必要としているか、キャンセルと望んでいるかを確認します。この 2 つは、ボットで発生する非常に一般的な割り込みです。 この確認が完了すると、ボットは `dc.continueDialog()` を呼び出して、まだ保留中のアクティブなダイアログをすべて処理します。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

ダイアログにユーザーの応答を送信する前に、割り込みを処理することができました。

このスニペットでは、ダイアログ セットに `helpDialog` と `mainMenu` が存在していることを前提としています。

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---
