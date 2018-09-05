---
title: ダイアログを使用して単純な会話フローを管理する | Microsoft Docs
description: Bot Builder SDK for Node.js でダイアログを使用して単純な会話フローを管理する方法について説明します。
keywords: 単純な会話フロー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756378"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>ダイアログを使用して単純な会話フローを管理する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローの両方を管理できます。 単純な会話フローでは、ユーザーは "*ウォーターフォール*" の最初のステップから始め、最後のステップまで進むと会話のやり取りが終了します。 ダイアログでは、ダイアログの一部を分岐およびループできる[複雑な会話フロー](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)を処理することもできます。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. -->ダイアログは、プログラムの関数に似ています。 通常、特定の操作を特定の順序で実行するように設計されており、必要に応じて何度でも呼び出すことができます。 ダイアログを使用することで、ボット開発者は会話フローを導くことができます。 複数のダイアログを連鎖させて、ボットで処理するほぼすべての会話フローを処理できます。 Bot Builder SDK の**ダイアログ** ライブラリには、会話フローの管理に役立つ "_プロンプト_" や "_ウォーターフォール ダイアログ_" などの組み込み機能があります。 プロンプトを使用すると、ユーザーにさまざまな種類の情報を要求できます。 ウォーターフォールを使用すると、複数のステップを 1 つのシーケンスに結合できます。

この記事では、"_ダイアログ セット_" を使用して、プロンプトとウォーターフォール ステップの両方を含む会話フローを作成します。 2 つのサンプル ダイアログがあります。 1 つ目は、ユーザー入力を必要としない操作を実行するワンステップ ダイアログです。 2 つ目は、ユーザーに情報の入力を求める複数ステップ ダイアログです。

## <a name="install-the-dialogs-library"></a>ダイアログ ライブラリをインストールする

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
npm install --save botbuilder-dialogs@preview
```

ボットで**ダイアログ**を使用するには、**app.js** ファイルに次のコードを追加します。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>ダイアログ スタックを作成する

この最初の例では、2 つの数値を加算し、結果を表示できるワンステップ ダイアログを作成します。

ダイアログを使用するには、まず "*ダイアログ セット*" を作成する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` ライブラリには、`DialogSet` クラスがあります。
**AdditionDialog** クラスを作成し、必要な using ステートメントを追加します。
名前付きダイアログと一連のダイアログをダイアログ セットに追加し、後で名前でアクセスできます。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

**DialogSet** からクラスを派生させ、このダイアログ セットのダイアログと入力情報の識別に使用する ID とキーを定義します。

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`botbuilder-dialogs` ライブラリには、`DialogSet` クラスがあります。
この **DialogSet** クラスは、**ダイアログ スタック**を定義し、そのスタックを管理するための単純なインターフェイスを表示します。
Bot Builder SDK は、スタックを配列として実装します。

**DialogSet** を作成するには、以下を実行します。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

上の呼び出しは、`dialogStack` という名前の既定の**ダイアログ スタック**を持つ **DialogSet** を作成します。
スタックに名前を付ける場合は、パラメーターとして **DialogSet()** に渡すことができます。 例: 

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```

---

ダイアログの作成は、このセットにダイアログの定義を追加するだけのことです。 ダイアログは、_begin_ または _replace_ メソッドを呼び出してそれをダイアログ スタックにプッシュするまでは実行されません。

ダイアログの名前 (たとえば `addTwoNumbers`) は、それぞれのダイアログ セット内で一意である必要があります。 各セットに、必要な数のダイアログを定義できます。 複数のダイアログ セットを作成し、それらをシームレスに連携させる場合は、[モジュラー ボット ロジックの作成](bot-builder-compositcontrol.md)に関する記事をご覧ください。

ダイアログ ライブラリは、次のダイアログを定義します。

* **プロンプト** ダイアログ。このダイアログは、少なくとも 2 つの関数を使用します。1 つはユーザーに入力を求め、1 つはその入力を処理します。 **ウォーターフォール** モデルを使用して、これらを一続きにすることができます。
* **ウォーターフォール** ダイアログ。次々に実行される "_ウォーターフォール ステップ_" シーケンスを定義します。 ウォーターフォール ダイアログに 1 つのステップのみを設定できます。この場合、ダイアログは、単純なワンステップ ダイアログとみなすことができます。

## <a name="create-a-single-step-dialog"></a>シングル ステップ ダイアログを作成する

シングル ステップ ダイアログは、1 往復の会話フローをキャプチャするために役立ちます。 次の例は、ユーザーが "1 + 2" のように発話したかどうかを検出し、`addTwoNumbers` ダイアログを開始して "1 + 2 = 3" で応答できるボットを作成します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

値がダイアログに渡され、ダイアログから `IDictionary<string,object>` プロパティ バッグとして返されます。

ダイアログ セット内に単純なダイアログを作成するには、`Add` メソッドを使用します。 以下では、`addTwoNumbers` という名前のワンステップ ウォーターフォールを追加しています。

このステップは、渡されたダイアログの引数に、加算される数値を表す `first` プロパティと `second` プロパティ が含まれているとみなします。

**AdditionDialog** クラスに次のコンストラクターを追加します。

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>ダイアログに引数を渡す

ボット コードで using ステートメントを更新します。

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

加算ダイアログのクラスに静的プロパティを追加します。

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

ボットの `OnTurn` メソッド内からダイアログを呼び出すには、`OnTurn` を、以下を含むように変更します。

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

ボット クラスに **TryParseAddingTwoNumbers** ヘルパー関数を追加します。 このヘルパー関数は、単純な正規表現を使用して、ユーザーのメッセージが 2 つの数値を加算する要求であるかどうかを検出します。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

EchoBot テンプレートを使用している場合は、**EchoState** クラスの名前を **ConversationData** に変更し、以下を含むように変更します。

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

「[Create a bot with the Bot Builder SDK v4](../javascript/bot-builder-javascript-quickstart.md)」(Bot Builder SDK v4 を使用してボットを作成する) で説明されている JS テンプレートから始めます。 **app.js** に、`botbuilder-dialogs` を要求するステートメントを追加します。

```js
const {DialogSet} = require('botbuilder-dialogs');
```

**app.js**に、`dialogs` セットに属する `addTwoNumbers` という名前の単純なダイアログを定義する次のコードを追加します。

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

**app.js** の受信アクティビティを処理するコードを、以下に置き換えます ボットは、受信メッセージが 2 つの数値を加算する要求と思われるかどうかをチェックするヘルパー関数を呼び出します。 該当する場合は、それらの数値を、`DialogContext.begin` の引数に渡します。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

**app.js** にヘルパー関数を追加します。 このヘルパー関数は、単純な正規表現を使用して、ユーザーのメッセージが 2 つの数値を加算する要求であるかどうかを検出します。 正規表現が一致する場合は、加算する数値を格納する配列を返します。

```javascript
async function TryParseAddingTwoNumbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>ボットを実行する

Bot Framework Emulator でボットを実行し、試しに "what's 1+1?" などと言って みます。

![ボットを実行する](./media/how-to-dialogs/bot-output-add-numbers.png)

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>ダイアログを使用してユーザーにステップを案内する

この次の例では、ユーザーに情報の入力を求める複数ステップ ダイアログを作成します。

### <a name="create-a-dialog-with-waterfall-steps"></a>ウォーターフォール ステップを持つダイアログを作成する

**ウォーターフォール**はダイアログ特有の実装であり、ユーザーから情報を収集したり、一連のタスクをユーザーに案内したりするときに最も一般的に使用されます。 タスクは関数の配列として実装され、最初の関数の結果が次の関数の入力として渡され、以下同様に処理されます。 各関数は、通常は、プロセス全体の 1 つのステップを表します。 各ステップで、ボットは、[ユーザーに入力を要求](bot-builder-prompts.md)し、応答を待ち、結果を次のステップに渡します。

たとえば、次のコード サンプルでは、**ウォーター フォール**の 3 つのステップを表す配列内に 3 つの関数を定義しています。 各プロンプトの後、ボットはユーザーの入力を確認しますが、入力は保存しませんでした。 ユーザー入力を保持する場合、詳細については、「[ユーザー データを保持する](bot-builder-tutorial-persist-user-inputs.md)」をご覧ください｡

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここでは、あいさつダイアログのコンストラクターを示しています。**GreetingDialog** は **DialogSet** から派生しています。**Inputs.Text** には **TextPrompt** オブジェクトに使用する ID が含まれ、**Main** にはあいさつダイアログ自体の ID が含まれます。

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

---

**ウォーター フォール** ステップのシグネチャは以下のとおりです。

| パラメーター | 説明 |
| :---- | :----- |
| `dc` | ダイアログのコンテキスト。 |
| `args` | 省略可能。ステップに渡される引数が含まれます。 |
| `next` | 省略可能。プロンプトなしでウォーターフォールの次のステップに進むことを可能にするメソッド。 このメソッドを呼び出すときに、*args* 引数を指定できます。これにより、ウォーターフォールの次のステップに引数を渡すことができます。 |

各ステップでは、制御を戻す前に、*next()* デリゲートまたはダイアログ コンテキスト メソッドのいずれか (*begin*、*end*、*prompt*、または *replace*) を呼び出す必要があります。そうしないと、ボットはそのステップで停止します。 つまり、関数がこれらのメソッドのいずれかで終了しない場合、ユーザーがボットにメッセージを送信するたびに、すべてのユーザー入力によってこのステップが再実行されます。

ウォーターフォールの終わりに達したら、ダイアログをスタックから取り出すことができるように、_end_ メソッドで制御を戻すのがベスト プラクティスです。 詳細については、後述の「[ダイアログを終了する](#end-a-dialog)」ご覧ください。 同様に、あるステップから次のステップに進むには、ウォーターフォール ステップをプロンプトで終了するか、_next_ デリゲートを明示的に呼び出してウォーターフォールを進める必要があります。

## <a name="start-a-dialog"></a>ダイアログを開始する

ダイアログを開始するには、開始する *dialogId* を、ダイアログ コンテキストの _begin_、_prompt_、または _replace_ メソッドに渡します。 _begin_ メソッドは、スタックの一番上にダイアログをプッシュします。_replace_ メソッドは、現在のダイアログをスタックから取り出し、置き換えるダイアログをスタックにプッシュします。

引数を指定しないでダイアログを開始するには:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

引数を指定してダイアログを開始するには:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

**プロンプト** ダイアログ開始するには:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここでは、**Inputs.Text** には、同じダイアログ セット内の **TextPrompt** の ID が含まれます。

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

開始するプロンプトの種類によっては、プロンプトの引数のシグネチャが異なる可能性があります。 **DialogSet.prompt** メソッドはヘルパー メソッドです。 このメソッドは引数を受け取り、プロンプト用の適切なオプションを作成した後、**begin** メソッドを呼び出してプロンプト ダイアログを開始します。 プロンプトの詳細については、[ユーザーへの入力の要求](bot-builder-prompts.md)に関する記事をご覧ください。

## <a name="end-a-dialog"></a>ダイアログを終了する

_end_ メソッドは、スタックからダイアログを取り出し、親ダイアログに省略可能な結果を返してダイアログを終了します。

ダイアログの終わりに _end_ メソッドを明示的に呼び出すのがベスト プラクティスです。ただし、ウォーターフォールの終わりに達すると、ダイアログは自動的にスタックから取り出されるため、これは必須ではありません。

ダイアログを終了するには:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

ダイアログを終了し、親ダイアログに情報を返すには、プロパティ バッグ引数を含めます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>ダイアログ スタックをクリアする

スタックからすべてのダイアログを取り出す場合は、_end all_ メソッドを呼び出すことで、ダイアログ スタックをクリアできます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>ダイアログを繰り返す

ダイアログを繰り返すには、_replace_ メソッドを使用します。 ダイアログ コンテキストの *replace* メソッドは、現在のダイアログをスタックから取り出し、置き換えるダイアログをスタックの一番上にプッシュしてそのダイアログを開始します。 これは、[複雑な会話フロー](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)を処理する優れた方法であり、メニューを管理する優れた手法でもあります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>次の手順

ここでは、単純な会話フローを管理する方法を説明しました。次に、_replace_ メソッドを活用して複雑な会話フローを処理する方法を見てみましょう。

> [!div class="nextstepaction"]
> [複雑な会話フローを管理する](bot-builder-dialog-manage-complex-conversation-flow.md)
