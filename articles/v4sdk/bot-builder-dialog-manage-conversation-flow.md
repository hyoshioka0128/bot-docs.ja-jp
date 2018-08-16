---
title: ダイアログを使用して会話フローを管理する | Microsoft Docs
description: Bot Builder SDK for Node.js でダイアログを使用して会話フローを管理する方法について説明します。
keywords: 会話フロー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302204"
---
# <a name="manage-conversation-flow-with-dialogs"></a>ダイアログを使用して会話フローを管理する
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


会話フローの管理は、ボットを構築する上で最も重要なタスクです。 Bot Builder SDK では、**ダイアログ**を使用して会話フローを管理できます。

ダイアログは、プログラムの関数に似ています。 それは、一般的に特定の操作を実行するように設計されており、必要に応じて何度でも呼び出すことができます。 複数のダイアログを連鎖させて、ボットで処理するほぼすべての会話フローを処理できます。 Bot Builder SDK の**ダイアログ** ライブラリには、**プロンプト**や**ウォーターフォール**などの組み込み機能があり、ダイアログを通して会話の流れを管理するのに役立ちます。 プロンプト ライブラリは、さまざまな種類の情報をユーザーに確認するために使用できる多様なプロンプトを提供します。 ウォーターフォールは、複数のステップを 1 つのシーケンスに結合する方法を提供します。

この記事では、ダイアログ オブジェクトを作成し、プロンプトとウォーターフォール ステップをダイアログ セットに追加して、単純な会話フローと複雑な会話フローの両方を管理する方法を示します。 

## <a name="install-the-dialogs-library"></a>ダイアログ ライブラリをインストールする

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
ダイアログを使用するには、プロジェクトまたはソリューション用の `Microsoft.Bot.Builder.Dialogs` NuGet パッケージをインストールします。
次に、コード ファイル内でステートメントを使用して、そのダイアログ ライブラリを参照します。 例: 

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
`botbuilder-dialogs` ライブラリは、NPM からダウンロードできます。 `botbuilder-dialogs` ライブラリをインストールするには、次の NPM コマンドを実行します。

```cmd
npm install --save botbuilder-dialogs
```

**ダイアログ**をボットで使用するには、それをボット コードに含めます。 例: 

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>ダイアログ スタックを作成する

ダイアログを使用するには、まず "*ダイアログ セット*" を作成する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` ライブラリには、`DialogSet` クラスがあります。
名前付きのダイアログと一連のダイアログ をダイアログ セットに追加し、後で名前でアクセスできます。

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`botbuilder-dialogs` ライブラリには、`DialogSet` クラスがあります。
この **DialogSet** クラスは、**ダイアログ スタック**を定義し、そのスタックを管理するための単純なインターフェイスを表示します。
Bot Builder SDK は、スタックを配列として実装します。

**DialogSet** を作成するには、以下を実行します。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

上の呼び出しは、`dialogStack` という名前の既定の**ダイアログ スタック**を持つ **DialogSet** を作成します。
スタックに名前を付ける場合は、パラメーターとして **DialogSet()** に渡すことができます。 次に例を示します。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```
---

ダイアログの作成は、このセットにダイアログの定義を追加するだけのことです。 ダイアログは、_begin_ または _replace_ メソッドを呼び出してそれをダイアログ スタックにプッシュするまでは実行されません。 

ダイアログの名前 (たとえば `addTwoNumbers`) は、それぞれのダイアログ セット内で一意である必要があります。 各セットに、必要な数のダイアログを定義できます。

ダイアログ ライブラリは、次のダイアログを定義します。
-   **プロンプト** ダイアログ。このダイアログは、少なくとも 2 つの関数を使用します。1 つはユーザーに入力を求め、1 つはその入力を処理します。
    **ウォーターフォール** モデルを使用して、これらを一続きにすることができます。
-   **ウォーターフォール** ダイアログ。次々に実行される "_ウォーターフォール ステップ_" シーケンスを定義します。
    ウォーターフォール ダイアログに 1 つのステップのみを設定できます。この場合、ダイアログは、単純なワンステップ ダイアログとみなすことができます。

## <a name="create-a-single-step-dialog"></a>シングル ステップ ダイアログを作成する

シングル ステップ ダイアログは、1 往復の会話フローをキャプチャするために役立ちます。 次の例は、ユーザーが "1 + 2" のように発話したかどうかを検出し、`addTwoNumbers` ダイアログを開始して "1 + 2 = 3" で応答できるボットを作成します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

値がダイアログに渡され、ダイアログから `IDictionary<string,object>` プロパティ バッグとして返されます。

ダイアログ セット内に単純なダイアログを作成するには、`Add` メソッドを使用します。 以下では、`addTwoNumbers` という名前のワンステップ ウォーターフォールを追加しています。

このステップは、渡されたダイアログの引数に、加算される数値を表す `first` プロパティと `second` プロパティ が含まれているとみなします。

EchoBot テンプレートから始めます。 コードに bot クラスを追加して、コンストラクターにダイアログを追加します。
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>ダイアログに引数を渡す

ボットの `OnTurn` メソッド内からダイアログを呼び出すには、`OnTurn` を、以下を含むように変更します。
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

bot クラスにヘルパー関数を追加します。 このヘルパー関数は、単純な正規表現を使用して、ユーザーのメッセージが 2 つの数値を加算する要求であるかどうかを検出します。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

EchoBot テンプレートを使用している場合は、**EchoState.cs** 内の `EchoState` クラス を次のように変更します。

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

**app.js** にヘルパー関数を追加します。 このヘルパー関数は、単純な正規表現を使用して、ユーザーのメッセージが 2 つの数値を加算する要求であるかどうかを検出します。 正規表現が一致する場合は、加算する数値を格納する配列を返します。

```javascript
async function MatchesAdd2Numbers(message) {
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>複合ダイアログを作成する

次のスニペットは、botbuilder-dotnet リポジトリの[Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) コード サンプルの一部です。

Startup.cs で:
1.  ボットの名前を `DialogContainerBot` に変更します。
1.  ボットの会話状態用のプロパティ バッグとして、単純なディクショナリを使用します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

`EchoBot` の名前を `DialogContainerBot` に変更します。

`DialogContainerBot.cs` で、プロファイル ダイアログ用のクラスを定義します。

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

次に、ボット定義内で、ボットのメイン ダイアログ用のフィールドを宣言し、ボットのコンストラクター内で初期化します。
ボットのメイン ダイアログに、プロファイル ダイアログが含まれます。

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

ボットの `OnTurn` メソッドで:
-   会話を始めるときに、ユーザーにあいさつします。
-   ユーザーからメッセージを取得するたびに、メイン ダイアログを初期化して _continue_ します。

    ダイアログが応答を生成しない場合は、ダイアログが既に完了しているか、まだ開始されていないとみなし、セット内のダイアログの名前を指定して、ダイアログを _begin_ します。

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>ウォーターフォール ステップを持つダイアログを作成する

会話は、ユーザーとボットの間で交わされる一連のメッセージで構成されます。 ボットの目的が一連のステップを通してユーザーを誘導することにある場合は、**ウォーターフォール** モデルを使用して会話のステップを定義できます。

**ウォーターフォール**はダイアログ特有の実装であり、ユーザーから情報を収集したり、一連のタスクをユーザーに案内したりするときに最も一般的に使用されます。 タスクは関数の配列として実装され、最初の関数の結果が次の関数の入力として渡され、以下同様に処理されます。 各関数は、通常は、プロセス全体の 1 つのステップを表します。 各ステップで、ボットは、[ユーザーに入力を要求](bot-builder-prompts.md)し、応答を待ち、結果を次のステップに渡します。

たとえば、次のコード サンプルは、**ウォーター フォール**の 3 つのステップを表す 3 つの関数を配列内に定義します。

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

**ウォーター フォール** ステップのシグネチャは以下のとおりです。

| パラメーター | 説明 |
| ---- | ----- |
| `context` | ダイアログのコンテキスト。 |
| `args` | 省略可能。ステップに渡される引数が含まれます。 |
| `next` | 省略可能。ウォーターフォールの次のステップに進むことを許可するメソッド。 このメソッドを呼び出すときに、*args* 引数を指定して、ウォーターフォール内の次のステップに引数を渡すことができます。 |

各ステップは、戻る前に次のメソッドのいずれかを呼び出す必要があります。*next()*、*dialogs.prompt()*、*dialogs.end()*、*dialogs.begin()*、または *Promise.resolve()*。これ以外の場合、ボットはそのステップでスタックします。 つまり、関数がこれらのメソッドのいずれかを返さない場合、ユーザーがボットにメッセージを送信するたびに、すべてのユーザー入力でこのステップが再実行されます。

ウォーター フォールの終わりに達したときのベスト プラクティスは、ダイアログをスタックから取り出すことができるように、`end()` メソッドで戻ることです。 詳細については、「[ダイアログを開始する](#end-a-dialog)」セクションを参照してください。 同様に、あるステップから次のステップに進むには、ウォーターフォール ステップをプロンプトで終了するか、`next()` 関数を明示的に呼び出して、ウォーターフォールを進める必要があります。 

### <a name="start-a-dialog"></a>ダイアログを開始する

ダイアログを開始するには、開始する *dialogId* を、`begin()` メソッド、`prompt()` メソッド、または `replace()` メソッドに渡します。 **begin** メソッドは、スタックの一番上にダイアログをプッシュし、**replace** メソッドは、現在のダイアログをスタックから取り出し、置き換えるダイアログをスタックにプッシュします。

引数を指定しないでダイアログを開始するには:

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

引数を指定してダイアログを開始するには:

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

**プロンプト** ダイアログ開始するには:

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

開始するプロンプトの種類によっては、プロンプトの引数のシグネチャが異なる可能性があります。 **DialogSet.prompt** メソッドはヘルパー メソッドです。 このメソッドは引数を受け取り、プロンプト用の適切なオプションを作成した後、**begin** メソッドを呼び出してプロンプト ダイアログを開始します。

スタック上のダイアログを置き換えるには:

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

**replace()** メソッドの使い方の詳細については、「[ダイアログを繰り返す](#repeat-a-dialog)」セクションと「[ダイアログをループさせる](#dialog-loops)」セクションを参照してください。

## <a name="end-a-dialog"></a>ダイアログを終了する

ダイアログは、スタックから取り出し、親ダイアログに省略可能な結果を返すことで終了します。 親ダイアログには、返された結果を使用して呼び出される **Dialog.resume()** メソッドがあります。

ベスト プラクティスは、ダイアログの終わりに `end()` メソッドを明示的に呼び出すことです。ただし、ウォーターフォールの終わりに達すると、ダイアログは自動的にスタックから取り出されるため、これは必須ではありません。

ダイアログを終了するには:

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

親ダイアログに渡される省略可能な引数を使用してダイアログを終了するには:

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

または、resolved promise を返すことによって、ダイアログを終了することもできます。

```javascript
await Promise.resolve();
```

`Promise.resolve()` を呼び出すと、ダイアログが終了し、スタックから取り出されます。 ただし、このメソッドでは、親ダイアログを呼び出して実行が再開されることはありません。 `Promise.resolve()` への呼び出し後、実行は停止され、ボットは、ユーザーがボットのメッセージを送信したときに離れた親ダイアログを再開します。 これは、ダイアログを終了するための理想的なユーザー エクスペリエンスにならない場合があります。 ボットがユーザーとの対話を続行できるように、`end()` または `replace()` のどちらかを使用してダイアログを終了することを検討してください。

### <a name="clear-the-dialog-stack"></a>ダイアログ スタックをクリアする

スタックからすべてのダイアログを取り出す場合は、`dc.endAll()` メソッドを呼び出すことで、ダイアログ スタックをクリアできます。

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>ダイアログを繰り返す

ダイアログを繰り返すには、`dialogs.replace()` メソッドを使用します。

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

メイン メニューを既定で表示する場合は、次の手順で `mainMenu` ダイアログを作成できます。

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
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
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

このダイアログは、`ChoicePrompt` を使用してメニューを表示し、ユーザーがオプションを選択するまで待機します。 ユーザーが `Order Dinner` または `Reserve a table` のいずれかを選択すると、選択に合ったサイアログが開始され、タスクが完了したら、最後のステップでダイアログを終了する代わりに、ダイアログそのものを繰り返します。

### <a name="dialog-loops"></a>ダイアログをループさせる

`replace()` メソッドの別の使用方法は、ループをエミュレートすることです。 たとえば、次のシナリオがあるとします。 ユーザーが複数のメニュー アイテムをカートに追加できるようにする場合は、ユーザーが注文を完了するまで、メニューの選択をループさせることができます。

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
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

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
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

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
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

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

上のサンプル コードは、メインの `orderDinner` ダイアログが、`orderPrompt` という名前のヘルパー ダイアログを使用して、ユーザーの選択を処理することを示しています。 `orderPrompt` ダイアログは、メニューを表示し、ユーザーにアイテムを選択するように依頼し、アイテムをカートに追加し、再度選択するように促します。 これにより、ユーザーは、複数のアイテムを注文に追加できます。 ダイアログのループは、ユーザーが `Process order` または `Cancel` を選択するまで続きます。 その時点で、実行が親ダイアログ (例: `orderDinner`) に戻ります。 `orderDinner` ダイアログは、ユーザーが注文を進める場合は、最後のハウス キーピング処理を実行します。そうでない場合は、実行を終了し、親ダイアログ (例: `mainMenu`) に実行を戻します。 `mainMenu` ダイアログは、最後のステップの実行を続行します。それは、メイン メニューの選択肢を再表示することです。

---

## <a name="next-steps"></a>次の手順

**ダイアログ**、**プロンプト**、および**ウォーターフォール** を使用して会話フローを管理する方法を学習したので、ボットのメイン ロジックである `dialogs` オブジェクトにすべてをまとめる代わりに、ダイアログをモジュール化されたタスクに分割する方法を確認できます。

> [!div class="nextstepaction"]
> [複合コントロールを使用してモジュラー化されたボットのロジックを作成する](bot-builder-compositcontrol.md)