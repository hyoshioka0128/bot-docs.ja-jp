---
title: ダイアログを使用して複雑な会話フローを管理する | Microsoft Docs
description: Bot Builder SDK for Node.js でダイアログを使用して複雑な会話フローを管理する方法について説明します。
keywords: 複雑な会話フロー, 繰り返し, ループ, メニュー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236362"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>ダイアログを使用して複雑な会話フローを管理する

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローの両方を管理できます。 [単純な会話フロー](bot-builder-dialog-manage-conversation-flow.md)では、ユーザーは "*ウォーターフォール*" の最初のステップから始め、最後のステップまで進むと会話のやり取りが終了します。 この記事では、ダイアログを使用して、分岐およびループできる部分を含むより複雑な会話を管理します。 そのために、ダイアログ コンテキストの _replace_ メソッドを使用し、ダイアログの異なる部分の間で引数を渡します。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

"*ダイアログ スタック*" をより細かく制御できるように、**ダイアログ** ライブラリには _replace_ メソッドが用意されています。 このメソッドを使用すると、現在のダイアログをスタックから取り出し、新しいダイアログをスタックの一番上にプッシュして、その新しいダイアログを開始できます。 この機能により、より複雑な会話を提供することが可能になります。 これらの手法を使用して、複雑な会話を任意に作成できます。 会話の複雑さが増してダイアログの管理が難しくなった場合は、制御ロジックの独自のフローを作成してユーザーの会話を追跡できます。

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

この記事では、ゲストがレストランを予約するときやルーム サービスの食事を注文するときに使用できるホテル ボットのダイアログを作成します。 最上位のダイアログでは、この 2 つのオプションをゲストに提供します。 ゲストがレストランを予約する場合は、最上位のダイアログでレストラン予約ダイアログが開始されます。 ゲストがディナーを注文する場合は、最上位のダイアログでディナー注文ダイアログが開始されます。 ディナー注文ダイアログでは、まずメニューからフード アイテムを選択するようゲストに求め、次に部屋番号の入力を求めます。 フード アイテムの選択もダイアログであるため、ゲストはディナーの注文を処理する前に複数のアイテムを選択できます。

次の図は、この記事で作成するダイアログとその相互関係を示しています。

![この記事で使用するダイアログの図](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>分岐する方法

ダイアログ コンテキストでは "_ダイアログ スタック_" が保持され、スタックのダイアログごとに、次のステップが追跡されます。 _begin_ メソッドは、ダイアログをスタックの一番上にプッシュし、_end_ メソッドは一番上のダイアログをスタックから取り出します。

ダイアログでは、ダイアログ コンテキストの _begin_ メソッドを呼び出し、新しいダイアログの ID を指定することで、同じダイアログ セット内で新しいダイアログ (分岐) を開始できます。 元のダイアログは引き続きスタックに存在しますが、ダイアログ コンテキストの _continue_ メソッドの呼び出しは、スタックの一番上のダイアログ ("_アクティブ ダイアログ_") にのみ送信されます。 ダイアログがスタックから取り出されると、ダイアログ コンテキストは、スタックのウォーターフォールの次のステップで、中断されたところから再開されます。

そのため、発生する可能性のある一連のダイアログの中から、開始するダイアログを条件付きで選択できるステップを 1 つのダイアログに含めることによって、会話フロー内に分岐を作成できます。

## <a name="how-to-loop"></a>ループする方法

ダイアログ コンテキストの _replace_ メソッドを使用すると、スタックの一番上のダイアログを置き換えることができます。 古いダイアログの状態が破棄され、新しいダイアログが最初から開始されます。 このメソッドを使用して、ダイアログをそれ自体に置き換えることでループを作成できます。 ただし、[状態を保持](bot-builder-howto-v4-state.md)するには、_replace_ メソッドの呼び出し時にダイアログの新しいインスタンスに情報を渡す必要があります。 次の例では、ディナー注文カートがダイアログの状態に保存されます。`orderPrompt` ダイアログが繰り返されると、新しいダイアログの状態で引き続きアイテムを追加できるように、現在のダイアログの状態が渡されます。 これらの情報を他の状態に保存する場合は、「[ユーザー データを保持する](bot-builder-tutorial-persist-user-inputs.md)」をご覧ください。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>ホテル ボットのダイアログを作成する

このセクションでは、前述のホテル ボットの会話を管理するためのダイアログを作成します。

### <a name="install-the-dialogs-library"></a>ダイアログ ライブラリをインストールする

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

基本的な EchoBot テンプレートから始めます。 手順については、[.NET のクイック スタート](~/dotnet/bot-builder-dotnet-quickstart.md)をご覧ください。

ダイアログを使用するには、プロジェクトまたはソリューション用の `Microsoft.Bot.Builder.Dialogs` NuGet パッケージをインストールします。
次に、必要に応じてコード ファイル内の using ステートメントでダイアログ ライブラリを参照します。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`botbuilder-dialogs` ライブラリは、NPM からダウンロードできます。 `botbuilder-dialogs` ライブラリをインストールするには、次の NPM コマンドを実行します。

```cmd
npm install --save botbuilder-dialogs
```

**ダイアログ**をボットで使用するには、それをボット コードに含めます。 たとえば、**app.js** ファイルに次のコードを追加します。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>ダイアログ セットを作成する

この例のすべてのダイアログの追加先となる "_ダイアログ セット_" を作成します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** クラスを作成し、必要な using ステートメントを追加します。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

**DialogSet** からクラスを派生させ、このダイアログ セットのダイアログ、プロンプト、状態情報の識別に使用する ID とキーを定義します。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

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

**ダイアログ**をボットで使用するには、それをボット コードに含めます。

**app.js** ファイルからライブラリを参照します。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

次に、ダイアログ セットを作成します。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>セットにプロンプトを追加する

**ChoicePrompt** を使用して、ディナーの注文とレストランの予約のどちらを行うかをゲストにたずね、ディナー メニューからオプションを選択するよう求めます。 また、**NumberPrompt** を使用して、ゲストがディナーを注文したときに部屋番号の入力を求めます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** コンストラクターに、2 つのプロンプトを追加します。

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

この 2 つのプロンプトをダイアログ セットに追加します。

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
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
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
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

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

この情報を格納する **dinnerMenu** 定数を作成します。

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

このダイアログは、`ChoicePrompt` を使用してメニューを表示し、ユーザーがオプションを選択するまで待機します。 ユーザーが `Order dinner` または `Reserve a table` のいずれかを選択すると、該当する選択肢のダイアログが開始されます。そのダイアログが完了したら、最後のステップでダイアログを単に終了し、ユーザーに何が起こっているのか疑問に思わせるのではなく、この `mainMenu` ダイアログで `mainMenu` ダイアログをもう一度開始してそれ自体を繰り返します。 この単純な構造により、ボットは常にメニューを表示することができ、ユーザーは選択内容を常に把握できます。

**MainMenu** ダイアログには、次のウォーターフォール ステップが含まれています。

* ウォーターフォールの最初のステップでは、ダイアログの状態を初期化またはクリアし、ゲストにあいさつして、利用可能なオプション (`Order dinner` または `Reserve a table`) から選択するよう求めます。
* 2 番目のステップでは、ゲストの選択内容を取得し、その選択に関連付けられた子ダイアログを開始します。 子ダイアログが終了すると、このダイアログが次のステップで再開されます。
* 最後のステップでは、**DialogContext.Replace** メソッドを使用して、このダイアログを新しいインスタンスに置き換えます。これにより、ウェルカム ダイアログがループ メニューに効果的に変わります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>ディナー注文ダイアログを作成する

ディナー注文ダイアログでは、"ディナー注文サービス" にゲストを迎え、次のセクションで説明するフード選択ダイアログを即座に開始します。 重要なのは、ゲストがサービスに注文の処理を依頼すると、フード選択ダイアログから注文のアイテムのリストが返されることです。 プロセス全体を完了するために、このダイアログでは、フードの届け先の部屋番号の入力を求め、終了する前に確認メッセージを送信します。 ゲストが注文をキャンセルした場合、このダイアログは部屋番号の入力を求めずに即座に終了します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** クラスに、ゲストのディナーの注文を追跡するために使用できるデータ構造を追加します。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

**HotelDialog** コンストラクターに、ディナー注文ダイアログを追加します。

* 最初のステップでは、ウェルカム メッセージを送信し、フード選択ダイアログを開始します。
* 2 番目のステップでは、フード選択ダイアログから注文カートが返されたかどうかを確認します。
  * 返された場合は、ゲストに部屋番号の入力を求め、カート情報を保存します。
  * 返されなかった場合は、ゲストが注文をキャンセルしたとみなし、**DialogContext.End** を呼び出してこのダイアログを終了します。
* 最後のステップでは、部屋番号を記録し、終了する前に確認メッセージを送信します。 このステップで、注文がボットから処理サービスに転送されます。

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);
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

**HotelDialog** コンストラクターに、フード選択ダイアログを追加します。

* 最初のステップでは、ダイアログの状態を初期化します。 ダイアログの入力引数にカート情報が含まれている場合は、それをダイアログの状態に保存します。それ以外の場合は、空のカートを作成して追加します。 次に、ゲストにディナー メニューから選択するよう促します。
* 次のステップでは、ゲストが選択したオプションを確認します。
  * 注文の処理要求の場合は、カートにアイテムが含まれているかどうかを確認します。
    * 含まれている場合は、カートの内容を返してダイアログを終了します。
    * それ以外の場合は、ゲストにエラー メッセージを送信し、ダイアログの最初からやり直します。
  * 注文のキャンセル要求の場合は、空のディクショナリを返してダイアログを終了します。
  * ディナー アイテムの場合は、アイテムをカートに追加して状態メッセージを送信し、現在の注文の状態を渡してダイアログをやり直します。

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

上のサンプル コードは、メインの `orderDinner` ダイアログが、`orderPrompt` という名前のヘルパー ダイアログを使用して、ユーザーの選択を処理することを示しています。 `orderPrompt` ダイアログは、フード アイテムのメニューを表示し、ユーザーにアイテムを選択するよう求め、アイテムをカートに追加して、ループ内で再度選択を促します。 これにより、ユーザーは、複数のアイテムを注文に追加できます。 ダイアログのループは、ユーザーが `Process order` または `Cancel` を選択するまで続きます。 その時点で、実行が親ダイアログ (`orderDinner` ダイアログ) に戻されます。 `orderDinner` ダイアログは、ユーザーが注文を進める場合は、最後のハウス キーピング処理を実行します。そうでない場合は、実行を終了し、親ダイアログ (例: `mainMenu`) に実行を戻します。 `mainMenu` ダイアログは、最後のステップの実行を続行します。それは、メイン メニューの選択肢を再表示することです。

---
### <a name="create-the-reserve-table-dialog"></a>レストラン予約ダイアログを作成する

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

この例を簡潔にするために、ここでは `reserveTable` ダイアログの最小限の実装のみを示します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>次の手順

メニューの選択肢に "詳細" や "ヘルプ" などの他のオプションを提供することで、このボットを強化できます。 この種の割り込みを実装する方法の詳細については、「[ユーザーによる割り込みの処理](bot-builder-howto-handle-user-interrupt.md)」をご覧ください。

ここでは、ダイアログ、プロンプト、ウォーターフォールを使用して複雑な会話フローを管理する方法を説明しました。次に、1 つの大規模なダイアログ セットにすべてのダイアログをまとめるのではなく、ダイアログ (`orderDinner` ダイアログや `reserveTable` ダイアログなど) を個別のオブジェクトに分割する方法を見てみましょう。

> [!div class="nextstepaction"]
> [複合コントロールを使用してモジュラー化されたボットのロジックを作成する](bot-builder-compositcontrol.md)
