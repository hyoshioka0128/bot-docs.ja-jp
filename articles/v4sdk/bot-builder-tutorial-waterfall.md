---
title: ユーザーに質問する | Microsoft Docs
description: Bot Builder SDK で、ウォーターフォール モデルを使用してユーザーに複数の入力を要求する方法について説明します。
keywords: ウォーターフォール、ダイアログ、質問する、プロンプト
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 5/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 84c86e9cc295cd2a1dce0857fa7ab629c219dab1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997479"
---
# <a name="ask-the-user-questions"></a>ユーザーに質問する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットは、基本的に、ユーザーとの会話で構成されます。 この会話は、短い場合やより複雑な場合、質問をする場合や質問に答える場合など、[多くの形式](bot-builder-conversations.md)を取ることができます。 会話の構成要素はいくつかの要因に依存しますが、それらはすべて会話を必要とします。

このチュートリアルでは、簡単な質問への回答から複数ターンのボットまでの、会話の構築について説明します。 使用する例はテーブルの予約に関するものですが、メニューの注文、よくある質問への回答、予約の実行など、複数ターンの会話を通してさまざまなことを実行するボットを想定することができます。

対話型のチャット ボットは、ユーザー入力に応答するか、またはユーザーに特定の入力を要求できます。 このチュートリアルでは、`Dialogs` の一部である `Prompts` ライブラリを使用してユーザーに質問する方法を示します。 [ダイアログ](../bot-service-design-conversation-flow.md)は会話の構造を定義するコンテナーであると考えることができ、ダイアログ内のプロンプトは固有の[ハウツー記事](bot-builder-prompts.md)でより詳細に説明されています。

## <a name="prerequisite"></a>前提条件

このチュートリアル内のコードは、[作業開始](~/bot-service-quickstart.md)エクスペリエンスを通して作成した**基本ボット**に基づきます。

## <a name="get-the-package"></a>パッケージを入手する

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Nuget パケット マネージャーから **Microsoft.Bot.Builder.Dialogs** パッケージをインストールします。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
ボットのプロジェクト フォルダーに移動し、NPM から `botbuilder-dialogs` パッケージをインストールします。

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>パッケージをボットにインポートする

# <a name="ctabcstab"></a>[C#](#tab/cstab)

ボット コードにダイアログとプロンプトの両方への参照を追加します。

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js** ファイルを開き、ボット コードに `botbuilder-dialogs` ライブラリを含めます。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

これにより、ユーザーに質問するために使用する `DialogSet` および `Prompts` ライブラリにアクセスできるようになります。 `DialogSet` は単に、**ウォーターフォール** パターンで構築するダイアログのコレクションです。 これは単純に、あるダイアログの後に別のダイアログが続くことを示します。

## <a name="instantiate-a-dialogs-object"></a>ダイアログ オブジェクトをインスタンス化する

`dialogs` オブジェクトをインスタンス化します。 このダイアログ オブジェクトは、質問および回答プロセスを管理するために使用します。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
ボット クラスのメンバー変数を宣言し、それをボットのコンストラクターで初期化します。 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>ウォーターフォール ダイアログを定義する

質問するには、少なくとも、2 段階の**ウォーターフォール** ダイアログが必要になります。 この例では、最初の手順でユーザーに名前の入力を要求し、2 番目の手順でそのユーザーを名前で歓迎する、2 段階の**ウォーターフォール** ダイアログを構成します。 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

ボットのコンストラクターを変更してダイアログを追加します。
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

質問は、`Prompts` ライブラリに付属する `textPrompt` メソッドを使用して行われます。 `Prompts` ライブラリには、ユーザーにさまざまな種類の情報の入力を要求できる一連のプロンプトが用意されています。 他のプロンプトの種類の詳細については、「[ユーザーに入力を要求する](~/v4sdk/bot-builder-prompts.md)」を参照してください。

入力の要求を機能させるには、dialogId `textPrompt` の `dialogs` オブジェクトにプロンプトを追加し、それを `TextPrompt()` コンストラクターで作成する必要があります。

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
ユーザーが質問に回答したら、手順 2 の `args` パラメーターで応答を見つけることができます。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
ユーザーが質問に回答したら、手順 2 の `results` パラメーターで応答を見つけることができます。 この場合は、`results` がローカル変数 `userName` に割り当てられます。 後のチュートリアルでは、ユーザー入力を選択したストレージに保持する方法を示します。

---


これで、質問するための `dialogs` を定義したので、このダイアログを呼び出して入力要求プロセスを開始する必要があります。

## <a name="start-the-dialog"></a>ダイアログを開始する

# <a name="ctabcstab"></a>[C#](#tab/cstab)

ボット ロジックを次のように変更します。

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

ボット ロジックが `OnTurn()` メソッド内に入ります。 ユーザーが "Hi" と言うと、ボットは `greetings` ダイアログを開始します。 `greetings` ダイアログの最初の手順では、ユーザーに名前の入力を要求します。 ユーザーは、名前を含む応答をメッセージ アクティビティとして送信し、その結果が `dc.Continue()` メソッドを通してウォーターフォールの手順 2 に送信されます。 ウォーターフォールの 2 番目の手順では (定義されたとおりに) そのユーザーを名前で歓迎し、ダイアログを終了します。 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**基本**ボットの `processActivity()` メソッドを次のように変更します。

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

ボット ロジックが `processActivity()` メソッド内に入ります。 ユーザーが "Hi" と言うと、ボットは `greetings` ダイアログを開始します。 `greetings` ダイアログの最初の手順では、ユーザーに名前の入力を要求します。 ユーザーは、名前を含む応答をアクティビティの `text` メッセージとして送信します。 そのメッセージが想定された意図に一致せず、かつボットがユーザーに何の応答も送信していないため、その結果が `dc.continue()` メソッドを通してウォーターフォールの手順 2 に送信されます。 ウォーターフォールの 2 番目の手順では (定義されたとおりに) そのユーザーを名前で歓迎し、ダイアログを終了します。 たとえば、手順 2 でユーザーにようこそメッセージを送信しなかった場合、`processActivity` メソッドは、ユーザーに送信される*既定のメッセージ*で終了します。

---



## <a name="define-a-more-complex-waterfall-dialog"></a>より複雑なウォーターフォール ダイアログを定義する

ここまで、ウォーターフォール ダイアログの動作やその構築方法を説明してきたので、テーブルの予約を目的にしたより複雑なダイアログを試してみましょう。

テーブル予約の要求を管理するには、4 つの手順を使用して**ウォーターフォール** ダイアログを定義する必要があります。 この会話では、`TextPrompt` に加えて、`DatetimePrompt` と `NumberPrompt` も使用します。



# <a name="ctabcstab"></a>[C#](#tab/cstab)

エコー ボット テンプレートから始め、ボットの名前を CafeBot に変更します。 `DialogSet` といくつかの静的メンバー変数を追加します。

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

次に、`reserveTable` ダイアログを定義します。 このダイアログは、ボット クラス コンストラクター内で追加できます。
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

`reserveTable` ダイアログは次のようになります。

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

`reserveTable` ダイアログの会話フローでは、ウォーターフォールの最初の 3 つの手順を使用して、ユーザーに 3 つの質問をします。 手順 4 では、最後の質問への回答を処理し、そのユーザーに予約の確認を送信します。



# <a name="ctabcstab"></a>[C#](#tab/cstab)
`reserveTable` ダイアログの各ウォーターフォール手順では、プロンプトを使用してユーザーに情報の入力を要求します。 ダイアログ セットにプロンプトを追加するために、次のコードが使用されました。

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

このウォーターフォールを機能させるには、`dialogs` オブジェクトに次のプロンプトも追加する必要があります。

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

これで、このコードをボット ロジックに組み込む準備ができました。

## <a name="start-the-dialog"></a>ダイアログを開始する

# <a name="ctabcstab"></a>[C#](#tab/cstab)
ボットの `OnTurn` を次のコードが含まれるように変更します。
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


**Startup.cs** で、ConversationState ミドルウェアの初期化を、`EchoState` の代わりに `Dictionary<string,object>` から派生したクラスを使用するように変更します。

たとえば、`Configure()` では次のようにします。
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

この要求に対するユーザーの意図をキャプチャするには、`processActivity()` メソッドに数行のコードを追加します。 ボットの `processActivity()` メソッドを次のように変更します。

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

実行時に、ユーザーが文字列 `reserve table` を含むメッセージを送信するたびに、ボットによって `reserveTable` 会話が開始されます。

---



## <a name="next-steps"></a>次の手順

このチュートリアルでは、ボットはユーザーの入力をボット内の変数に保存します。 これらの情報を格納または保持する場合は、状態をミドルウェア レイヤーに追加する必要があります。 ユーザー状態データを保持する方法の詳細を次のチュートリアルで見てみましょう。 

> [!div class="nextstepaction"]
> [ユーザー データを保持する](bot-builder-tutorial-persist-user-inputs.md)
