---
title: ダイアログを使用して複雑な会話フローを管理する | Microsoft Docs
description: Bot Builder SDK for Node.js でダイアログを使用して複雑な会話フローを管理する方法について説明します。
keywords: 複雑な会話フロー, 繰り返し, ループ, メニュー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326569"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>ダイアログを使用して複雑な会話フローを管理する

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

前回の記事では、ダイアログ ライブラリを使用して単純な会話を管理する方法を紹介しました。 [単純な会話フロー](bot-builder-dialog-manage-conversation-flow.md)では、ユーザーは "*ウォーターフォール*" の最初のステップから始め、最後のステップまで進むと会話のやり取りが終了します。 この記事では、ダイアログを使用して、分岐およびループできる部分を含むより複雑な会話を管理します。 そのために、ダイアログ コンテキストとウォーターフォール ステップ コンテキストに定義されているさまざまなメソッドを使用し、ダイアログを構成する要素間で引数の受け渡しを行います。

ダイアログの背景情報について詳しくは、「[ダイアログ ライブラリ](bot-builder-concept-dialog.md)」を参照してください。

"*ダイアログ スタック*" をより細かく制御できるように、**ダイアログ** ライブラリには _replace dialog_ メソッドが用意されています。 このメソッドを使用すると、会話の状態とフローを維持したまま、現在アクティブなダイアログと別のダイアログとを入れ替えることができます。 _begin dialog_ メソッドと _replace dialog_ メソッドで分岐とループを必要に応じて使えば、より複雑な対話を作成することができます。 会話の複雑さが増してウォーターフォール ダイアログの管理が難しくなった場合は、[コンポーネント ダイアログ](bot-builder-compositcontrol.md)を使用する方法や、基本 `Dialog` クラスをベースにカスタムのダイアログ管理クラスを作成する方法について調べてみてください。

この記事で作成するサンプル ダイアログは、ホテルのレストランのテーブル予約とルーム サービスへの食事の注文という一般的なホテル サービスを利用する際にゲストが利用できるホテル コンシェルジュ ボットを想定しています。  これらの機能はそれぞれ、ダイアログ セット内のダイアログとして、それらを相互に関連付けるメニューと共に作成されます。

これらの 2 つの選択肢は、ボットの最上位のダイアログでゲストに提供されます。 ゲストがテーブルを予約する場合は、最上位のダイアログで _begin dialog_ 非同期メソッドを使用してテーブル予約ダイアログが開始されます。 ゲストがルーム サービスを注文する場合は、最上位のダイアログでディナー注文ダイアログが開始されます。

ディナー注文ダイアログでは、まずメニューからフード アイテムを選択するようゲストに求め、次に部屋番号の入力を求めます。 フード アイテムの選択 "_も_" ダイアログです。つまり、ゲストがメニューからアイテムを選択する際、ディナーの注文を確定するまで複数回にわたって利用されます。

次の図は、この記事で作成するダイアログとその相互関係を示しています。

![この記事で使用するダイアログの図](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>分岐する方法

ダイアログ コンテキストでは "_ダイアログ スタック_" が保持され、スタックのダイアログごとに、次のステップが追跡されます。 _begin dialog_ メソッドは、ダイアログをスタックの一番上にプッシュし、_end dialog_ メソッドは一番上のダイアログをスタックから取り除きます。

分岐させたければ、一連の子ダイアログから開始するものを選択します。 この概念の詳細については、「[会話の分岐](bot-builder-concept-dialog.md#branch-a-conversation)」を参照してください。

## <a name="how-to-loop"></a>ループする方法

ダイアログ コンテキストの _replace dialog_ メソッドを使用すると、スタックの一番上のダイアログを置き換えることができます。 古いダイアログの状態が破棄され、新しいダイアログが最初から開始されます。 このメソッドを使用して、ダイアログをそれ自体に置き換えることでループを作成できます。 ただし、[ボットによって既に収集された情報を保持](bot-builder-howto-v4-state.md)するためには、その情報を、_replace dialog_ メソッドの呼び出し時にダイアログの新しいインスタンスに渡す必要があります。

次の例では、ルーム サービス注文がダイアログの状態に保存されます。`orderPrompt` ダイアログが繰り返されると、新しいダイアログの状態で引き続きアイテムを追加できるように、現在のダイアログの状態が渡されます。 ダイアログの外部にあるボット状態にこの情報を保存したい場合は、[ユーザー データを保持する](bot-builder-tutorial-persist-user-inputs.md)方法に関するページを参照してください。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>ホテル ボットのダイアログを作成する

このセクションでは、前述のホテル ボットの会話を管理するためのダイアログを作成します。

### <a name="install-the-dialogs-library"></a>ダイアログ ライブラリをインストールする

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

基本的な EchoBot テンプレートから始めます。 手順については、[.NET のクイック スタート](../dotnet/bot-builder-dotnet-sdk-quickstart.md)をご覧ください。

ダイアログを使用するには、プロジェクトまたはソリューション用の `Microsoft.Bot.Builder.Dialogs` NuGet パッケージをインストールします。
次に、必要に応じてコード ファイル内の using ステートメントでダイアログ ライブラリを参照します。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

EchoBot テンプレートをベースにして開始します。 手順については、[JavaScript のクイック スタート](../javascript/bot-builder-javascript-quickstart.md)を参照してください。



`botbuilder-dialogs` ライブラリは、npm からダウンロードできます。 `botbuilder-dialogs` ライブラリをインストールするには、次の npm コマンドを実行します。

```cmd
npm install --save botbuilder-dialogs
```

**ダイアログ**をボットで使用するには、それをボット コードに含めます。 たとえば、**index.js** ファイルに次のコードを追加します。

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

さらに次のコードを **bot.js** ファイルに追加します。

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>ダイアログ セットを作成する

この例のすべてのダイアログの追加先となる "_ダイアログ セット_" を作成します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialogs** クラスを作成し、必要な using ステートメントを追加します。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

このクラスは、**DialogSet** から派生させます。 `IStatePropertyAccessor<DialogState>` パラメーターを受け取るコンストラクターを追加します。このパラメーターは、ダイアログ セットのインスタンスの内部状態を管理するために使用します。 さらに、このダイアログ セットのダイアログ、プロンプト、状態情報の識別に使用する ID とキーを定義します。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js** ファイルに、ダイアログの状態管理に使用する状態プロパティ アクセサーを作成するためのコードと、そのアクセサーを使用して、ボットに使用するダイアログ セットを作成するためのコードを追加します。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

次に、そのボット オブジェクトを使用するように、アクティビティ処理の呼び出しを更新してください。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>セットにプロンプトを追加する

**ChoicePrompt** を使用して、ディナーの注文とレストランの予約のどちらを行うかをゲストにたずね、ディナー メニューからオプションを選択するよう求めます。 また、**NumberPrompt** を使用して、ゲストがディナーを注文したときに部屋番号の入力を求めます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialogs** コンストラクターに、2 つのプロンプトを追加します。

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットのコンストラクターを更新し、ダイアログ セットに 2 つのプロンプトを追加します。

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
```

---

### <a name="define-some-of-the-supporting-information"></a>関連情報を定義する

ディナー メニューの各オプションに関する情報が必要であるため、ここではそれを設定しましょう。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

内部静的 **Lists** クラスを作成して、この情報を含めます。 また、内部クラス **WelcomeChoice** と **MenuChoice** を作成して、各オプションに関する情報を含めます。

ここでは、最上位のウェルカム ダイアログにオプションのリストの情報を追加します。また、後でゲストにこの情報の入力を求めるときに使用する関連リストも作成します。 これは事前に行うわずかな追加作業ですが、ダイアログ コードが簡素化されます。

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** ファイルに、この情報を格納する **dinnerMenu** 定数を作成します。

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel"],
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

### <a name="create-the-welcome-dialog"></a>ウェルカム ダイアログを作成する

このダイアログでは、`ChoicePrompt` を使用してメニューが表示され、ユーザーがオプションを選択するまで待機されます。 ユーザーが `Order dinner` または `Reserve a table` を選択すると、該当するダイアログが開始されます。 その子ダイアログが完了したら、最後のステップでダイアログを単に終了し、ユーザーに何が起こっているのか疑問に思わせるのではなく、`mainMenu` ダイアログをもう一度開始して `mainMenu` ダイアログをループさせます。 この単純な構造により、ボットは常にメニューを表示することができ、ユーザーは選択内容を常に把握できます。

`mainMenu` ダイアログには、以下のウォーターフォール ステップが含まれています。

* ウォーターフォールの最初のステップでは、ダイアログの状態を初期化またはクリアし、ゲストにあいさつして、利用可能なオプション (`Order dinner` または `Reserve a table`) から選択するよう求めます。
* 2 番目のステップでは、ゲストの選択内容を取得し、その選択に関連付けられた子ダイアログを開始します。 子ダイアログが終了すると、このダイアログが次のステップで再開されます。
* 最後のステップでは、_replace dialog_ メソッドを使用して、このダイアログを新しいインスタンスに置き換えます。これにより、ウェルカム ダイアログがループ メニューに効果的に変わります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

コンストラクター内にウォーターフォール ダイアログを追加します。 その後、ウォーターフォール ステップを定義します。 ここでは、入れ子になったクラス内にそれらを定義しています。

**HotelDialogs** コンストラクター内。

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

**HotelDialogs** クラス内に、ステップの定義を追加します。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット コンストラクターに `mainMenu` ウォーターフォール ダイアログを追加します。

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>ディナー注文ダイアログを作成する

ディナー注文ダイアログでは、"ディナー注文サービス" にゲストを迎え、次のセクションで説明するフード選択ダイアログを即座に開始します。 重要なのは、ゲストがサービスに注文の処理を依頼すると、フード選択ダイアログから注文のアイテムのリストが返されることです。 プロセスを完了するために、このダイアログでは、フードの届け先の部屋番号の入力を求め、確認メッセージを送信します。 ゲストが注文をキャンセルした場合、このダイアログは部屋番号の入力を求めずに即座に終了します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** クラスに、ゲストのディナーの注文を追跡するために使用できるデータ構造を追加します。 これはダイアログの状態に保持されるため、このクラスには、シリアル化のための既定のコンストラクターが必要です。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

コンストラクター内にウォーターフォール ダイアログを追加します。 その後、ウォーターフォール ステップを定義します。 ここでは、入れ子になったクラス内にそれらを定義しています。

**HotelDialogs** コンストラクター内。

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

**HotelDialogs** クラス内に、ステップの定義を追加します。

* 最初のステップでは、ウェルカム メッセージを送信し、フード選択ダイアログを開始します。
* 2 番目のステップでは、フード選択ダイアログから注文カートが返されたかどうかを確認します。
  * 返された場合は、ゲストに部屋番号の入力を求め、カート情報を保存します。
  * 返されなかった場合は、ゲストが注文をキャンセルしたとみなし、_end dialog_ メソッドを呼び出してこのダイアログを終了します。
* 最後のステップでは、部屋番号を記録し、終了する前に確認メッセージを送信します。 このステップで、注文がボットから処理サービスに転送されます。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット コンストラクターに `orderDinner` ウォーターフォール ダイアログを追加します。

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>注文プロンプト ダイアログを作成する

フード選択ダイアログでは、注文できるディナー アイテムと 2 つの注文処理要求を含むオプションのリストをゲストに提示します。 このダイアログは、ボットに注文を処理させるか、注文をキャンセルするかをゲストが選択するまでループします。

* ゲストがディナー アイテムを選択すると、"_カート_" に追加されます。
* ゲストが注文の処理を選択した場合は、まずカートが空かどうかを確認します。
  * 空の場合は、エラー メッセージを送信し、ループを続行します。
  * それ以外の場合は、親ダイアログにカート情報を返して、このダイアログを終了します。
* ゲストが注文のキャンセルを選択した場合は、カート情報を返さずにダイアログを終了します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

コンストラクター内にウォーターフォール ダイアログを追加します。 その後、ウォーターフォール ステップを定義します。 ここでは、入れ子になったクラス内にそれらを定義しています。

**HotelDialogs** コンストラクター内。

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* 最初のステップでは、ダイアログの状態を初期化します。 ダイアログの入力引数にカート情報が含まれている場合は、それをダイアログの状態に保存します。それ以外の場合は、空のカートを作成して追加します。 次に、ゲストにディナー メニューから選択するよう促します。
* 次のステップでは、ゲストが選択したオプションを確認します。
  * 注文の処理要求の場合は、カートにアイテムが含まれているかどうかを確認します。
    * 含まれている場合は、カートの内容を返してダイアログを終了します。
    * それ以外の場合は、ゲストにエラー メッセージを送信し、ダイアログの最初からやり直します。
  * 注文のキャンセル要求の場合は、空のディクショナリを返してダイアログを終了します。
  * ディナー アイテムの場合は、アイテムをカートに追加して状態メッセージを送信し、現在の注文の状態を渡してダイアログをやり直します。

**HotelDialogs** クラス内に、ステップの定義を追加します。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット コンストラクターに `orderPrompt` ウォーターフォール ダイアログを追加します。

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

上のサンプル コードは、メインの `orderDinner` ダイアログが、`orderPrompt` という名前のヘルパー ダイアログを使用して、ユーザーの選択を処理することを示しています。 `orderPrompt` ダイアログは、フード アイテムのメニューを表示し、ユーザーにアイテムを選択するよう求め、アイテムをカートに追加して、ループ内で再度選択を促します。 これにより、ユーザーは、複数のアイテムを注文に追加できます。 ダイアログのループは、ユーザーが `Process order` または `Cancel` を選択するまで続きます。 その時点で、実行が親ダイアログ (`orderDinner` ダイアログ) に戻されます。 `orderDinner` ダイアログは、ユーザーが注文を進める場合は、最後のハウス キーピング処理を実行します。そうでない場合は、実行を終了し、親ダイアログ (例: `mainMenu`) に実行を戻します。 `mainMenu` ダイアログは、最後のステップの実行を続行します。それは、メイン メニューの選択肢を再表示することです。

---

### <a name="create-the-reserve-table-dialog"></a>レストラン予約ダイアログを作成する

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

この例を簡潔にするために、ここでは `reserveTable` ダイアログの最小限の実装のみを示します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

コンストラクター内にウォーターフォール ダイアログを追加します。 その後、ウォーターフォール ステップを定義します。 ここでは、入れ子になったクラス内にそれらを定義しています。

**HotelDialogs** コンストラクター内。

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

**HotelDialogs** クラス内に、ステップの定義を追加します。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット コンストラクターにプレースホルダーの `reserveTable` ウォーターフォール ダイアログを追加します。

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>ダイアログを呼び出すようにボットのコードを更新する

ダイアログを呼び出すようにボットのターン ハンドラー コードを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBotAccessors.cs** の名前を **BotAccessors.cs** に変更し、クラス名前を `EchoBotAccessors` から `BotAccessors` に変更します。 using ステートメントとクラス定義を更新し、このボットに必要な状態プロパティ アクセサーを追加します。

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

**Startup.cs** ファイルを更新して、`BotAccessors` をシングルトンとして構成します。

1. using ステートメントを更新します。

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. `ConfigureServices` メソッドの、ボット状態プロパティ アクセサーを登録する部分を更新します。

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

EchoWithCounterBot.cs ファイルの名前を HotelBot.cs に変更し、クラス名を EchoWithCounterBot から HotelBot に変更します。

1. ボットの初期化コードを更新します。

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. ボットのターン ハンドラーを更新して、ダイアログを実行するようにします。

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>次の手順

メニューの選択肢に "詳細" や "ヘルプ" などの他のオプションを提供することで、このボットを強化できます。 この種の割り込みを実装する方法の詳細については、「[ユーザーによる割り込みの処理](bot-builder-howto-handle-user-interrupt.md)」をご覧ください。

ここでは、ダイアログ、プロンプト、ウォーターフォールを使用して複雑な会話フローを管理する方法を説明しました。次に、1 つの大規模なダイアログ セットにすべてのダイアログをまとめるのではなく、ダイアログ (`orderDinner` ダイアログや `reserveTable` ダイアログなど) を個別のオブジェクトに分割する方法を見てみましょう。

> [!div class="nextstepaction"]
> [複合コントロールを使用してモジュラー化されたボットのロジックを作成する](bot-builder-compositcontrol.md)
