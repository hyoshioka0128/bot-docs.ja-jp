---
title: ダイアログ ライブラリを使用してユーザーに入力を求める | Microsoft Docs
description: Bot Builder SDK for Node.js で、ダイアログ ライブラリを使用してユーザーに入力を求める方法について説明します。
keywords: プロンプト, ダイアログ, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 再プロンプト, 検証
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27066f76db29a82b4ab9dd75bf5eee01dcce3116
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389711"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>ダイアログ ライブラリを使用してユーザーに入力を求める

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

質問を投稿することで情報を収集することは、ボットがユーザーとやり取りする主な手段の 1 つです。 これは、[turn context](bot-builder-concept-activity-processing.md#turn-context) オブジェクトの _send activity_ メソッドを使用して、その後次の受信メッセージを応答として処理することで、直接行うことができます。 ただし、Bot Builder SDK には、簡単に質問をして、その応答が特定のデータ型と一致するかカスタム検証ルールを満たすように設計されたメソッドを提供する、**ダイアログ** ライブラリが用意されています。 このトピックでは、**プロンプト**を使用してこれを達成し、ユーザーに入力を求める方法について説明します。

この記事では、ダイアログ内でプロンプトを使用する方法について説明します。 一般的なダイアログの使用に関する情報については、[ダイアログを使用した単純な会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関するページをご覧ください。

## <a name="prompt-types"></a>プロンプトの種類

ダイアログ ライブラリには、さまざまな種類のプロンプトがあり、それぞれが異なる種類の応答の収集に使用されます。

| Prompt | 説明 |
|:----|:----|
| **AttachmentPrompt** | 文書や画像など、添付ファイルをユーザーに求めます。 |
| **ChoicePrompt** | 一連の選択肢から選択するようにユーザーに求めます。 |
| **ConfirmPrompt** | 自分のアクションを確定するようにユーザーに求めます。 |
| **DatetimePrompt** | 日時をユーザーに求めます。 "明日の午後 8 時" や "金曜日の午前 10 時" など、自然言語を利用してユーザーは回答できます。 Bot Framework SDK では、LUIS `builtin.datetimeV2` 事前作成済みエンティティが使用されます。 詳細については、「[builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)」を参照してください。 |
| **NumberPrompt** | 数字をユーザーに求めます。 ユーザーは "10" か "ten" で回答できます。 回答がたとえば "ten" の場合、プロンプトにより応答が数字に変換され、結果として `10` が返されます。 |
| **TextPrompt** | テキストの文字列をユーザーに求めます。 |

## <a name="add-references-to-prompt-library"></a>プロンプト ライブラリに参照を追加する

**botbuilder-dialogs** パッケージをボットに追加することで**ダイアログ** ライブラリを取得できます。 ダイアログについては、[ダイアログを使用した単純な会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関するページで取り上げますが、Microsoft ではプロンプトにダイアログを使用します。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

NuGet から **Microsoft.Bot.Builder.Dialogs** パッケージをインストールします。

次に、ボット コードからライブラリの参照を追加します。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

アクセサーを介して会話ダイアログの状態を設定する必要があります。 ここではこのコードについてあまり説明しませんが、その詳細については[状態](bot-builder-howto-v4-state.md)に関する記事で確認できます。

**Startup.cs** のボット オプション内で、まず状態のオブジェクトを定義し、次にシングルトンを追加してそのボット コンストラクターにアクセサー クラスを指定します。 `BotAccessor` のクラスは、単に会話とユーザーの状態を、それらの各項目のアクセサーと共に格納します。 クラスの全定義は、この記事の最後に設定されたリンク先のサンプルに用意されています。 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

その後、ボット コード内で、ダイアログ セットに対して次のオブジェクトを定義します。

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

NPM からダイアログ パッケージをインストールします。

```cmd
npm install --save botbuilder-dialogs
```

ボットで**ダイアログ**を使用するには、それをボット コードに含めます。

app.js ファイルで、次を追加します。

```javascript
// Import components from the dialogs library.
const { DialogSet } = require("botbuilder-dialogs");
// Import components from the main Bot Builder library.
const { ConversationState, MemoryStorage } = require('botbuilder');

// Set up a memory storage system to store information.
const storage = new MemoryStorage();
// We'll use ConversationState to track the state of the dialogs.
const conversationState = new ConversationState(storage);
// Create a property used to track state.
const dialogState = conversationState.createProperty('dialogState');

// Create a dialog set to control our prompts, store the state in dialogState
const dialogs = new DialogSet(dialogState);
```

---

## <a name="prompt-the-user"></a>ユーザーに入力を求める

ユーザーに入力を求めるには、**TextPrompt** などの組み込みのクラスのいずれか 1 つを使用してプロンプトを定義し、その後それをダイアログ セットに追加して、ダイアログ ID に割り当てます。

追加されたプロンプトは、2 ステップのウォーターフォール ダイアログで使用します。"*ウォーターフォール*" ダイアログは、ステップのシーケンスを定義する方法の 1 つです。 複数のプロンプトを結合して、複数ステップの会話を作成できます。 詳細については、[ダイアログを使用した単純なな会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関する記事の[ダイアログの使用](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)に関するセクションをご覧ください。

たとえば、次のダイアログでは、ユーザーに名前の入力を求め、その応答を使用してユーザーにあいさつします。 最初のターンでは、ユーザーに名前の入力を要求します。 ユーザーの応答は 2 番目のステップの関数にパラメーターとして渡され、その関数が入力を処理してパーソナライズされたあいさつを送信します。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

ダイアログで使用する各プロンプトにも名前が表示されます。その名前はプロンプトにアクセスする目的でダイアログまたはボットにより使用されます。 以上のすべてのサンプルで、プロンプト ID を定数として公開しています。

ボットのコンストラクター内で、2 ステップのウォーターフォールの定義と使用するダイアログのプロンプトを追加します。 ここではそれらを独立した関数として追加しますが、必要な場合はインラインのラムダとして定義できます。

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

次に、ボット内に 2 ステップのウォーター フォールを定義します。 テキスト プロンプトには、前に定義した `TextPrompt` の *name* ID を指定します。 メソッド名は前述の `WaterfallStep[]` それらと一致することに注意してください。 ここにある将来のサンプルにそのコードは含まれませんが、追加のステップでその `WaterfallStep[]` に正しい順序でメソッド名を追加する必要があります。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

TextPrompt クラスをアプリにインポートします。

```javascript
const { TextPrompt } = require("botbuilder-dialogs");
```

新しいプロンプトを作成し、それをダイアログ セットに追加します。

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add(new TextPrompt('textPrompt'));
dialogs.add('greetings', [
    async function (step){
        // the results of this prompt will be passed to the next step
        return await step.prompt('textPrompt', 'What is your name?');
    },
    async function(step) {
        // step.result is the result of the prompt defined above
        const userName = step.result;
        await step.context.sendActivity(`Hi ${userName}!`);
        return await step.endDialog();
    }
]);
```

---

> [!NOTE]
> ダイアログを開始するには、ダイアログ コンテキストを取得し、その _begin_ メソッドを使用します。 詳細については、[ダイアログを使用した単純な会話フローの管理](./bot-builder-dialog-manage-conversation-flow.md)に関するページをご覧ください。

## <a name="reusable-prompts"></a>再利用可能プロンプト

プロンプトは、同じ種類の回答であれば別の質問をするのに再利用できます。 たとえば、前出のサンプル コードではテキスト プロンプトが定義され、それを利用してユーザーに名前を求めました。 その同じプロンプトを利用し、「職場はどこですか」など、ユーザーに別のテキスト文字列を求めることもできます。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

この例では、テキスト プロンプト *name* の ID はコードの明瞭化の役には立ちません。 ただし、プロンプト ID を任意の ID にすることが可能であることを示すよい例です。

これで、このメソッドにはユーザーの職場がどこであるかを質問する 3 番目のステップが追加されます。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add(new TextPrompt('textPrompt'));
dialogs.add('greetings',[
    async function (step){
        // Use the textPrompt to ask for a name.
        return await step.prompt('textPrompt', 'What is your name?');
    },
    async function (step){
        const userName = step.result;
        await step.context.sendActivity(`Hi ${ userName }!`);

        // Now, reuse the same prompt to ask them where they work.
        return await step.prompt('textPrompt', 'Where do you work?');
    },
    async function(step) {
        const workPlace = step.result;
        await step.context.sendActivity(`${ workPlace } is a cool place!`);

        return await step.endDialog();
    }
]);
```

---

複数の異なるプロンプトを使用する必要がある場合は、それぞれのプロンプトに一意の *dialogId* を指定します。 ダイアログ セットに追加される各ダイアログまたはプロンプトには一意の ID が必要です。 同じ種類の**プロンプト** ダイアログを複数作成することもできます。 たとえば、上記の例には 2 つの **TextPrompt** ダイアログを作成できます。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```javascript
dialogs.add(new TextPrompt('namePrompt'));
dialogs.add(new TextPrompt('workPlacePrompt'));
```

---

コードの再利用性のために、`TextPrompt` を 1 つ定義すると、応答はすべてテキストが求められるため、これらすべてのプロンプトでうまく動作します。 ダイアログに名前を付けることは、プロンプトの入力に異なる検証ルールを適用する必要があるときに便利です。 `NumberPrompt` を利用してプロンプトの応答を検証する方法を見てみましょう。

## <a name="specify-prompt-options"></a>プロンプト オプションを指定する

ダイアログ ステップ内でプロンプトを使用するとき、再プロンプト文字列など、プロンプト オプションを提供することもできます。

再プロンプト文字列の指定は、数値の求めに対して "明日" が回答されるなどの解析できない形式であるか、入力で検証基準を満たせないときなどの、ユーザー入力がプロンプトを満たせない場合に便利です。 数値を求めるプロンプトでは、"twelve" や "4 分の 1"、"12" や "0.25" など、さまざまな入力を解釈できます。

ローカルは、特定のプロンプト (**NumberPrompt** など) におけるオプションのパラメーターです。 これは、プロンプトの入力をより正確に解析するのに役立つことがありますが、必須ではありません。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

次のコードでは、既存のダイアログ セット **_dialogs** に数値を求めるプロンプトが追加されます。

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

ダイアログ ステップ内で、次のコードにより、ユーザーに入力が求められ、その入力が数値として解釈できない場合に使用する再プロンプト文字列が用意されます。

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```javascript
// Import the NumberPrompt class from the dialog library.
const { NumberPrompt } = require("botbuilder-dialogs");

// Add a NumberPrompt to our dialog set and give it the ID numberPrompt.
dialogs.add(new NumberPrompt('numberPrompt'));

// Call the numberPrompt dialog with the (optional) retryPrompt parameter.
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

---

選択プロンプトには追加の必須のパラメーターがあります。ユーザーが利用できる選択肢の一覧です。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

**ChoicePrompt** を使用して一連の選択肢から選択するようにユーザーに求めるとき、その一連の選択肢を含むプロンプトを提供する必要があります。これは **PromptOptions** オブジェクト内で提供します。 ここでは、**ChoiceFactory** を使用し、選択肢の一覧を適切な形式に変換しています。

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```javascript
// Import the ChoicePrompt class into your app from the dialogs library.
const { ChoicePrompt } = require("botbuilder-dialogs");
```

```javascript
// Add a ChoicePrompt to the dialog set and give it an ID of choicePrompt.
dialogs.add(new ChoicePrompt('choicePrompt'));
```

```javascript
// Call the choicePrompt into action, passing in an array of options.
const list = ['green', 'blue', 'red', 'yellow'];
await dc.prompt('choicePrompt', 'Please make a choice', list, { retryPrompt: 'Please choose a color.' });
```

---

## <a name="validate-a-prompt-response"></a>プロンプトの応答を検証する

**ウォーターフォール**の次のステップに値を返す前にプロンプトの応答を検証できます。 たとえば、**6** から **20** までの数値範囲内で **NumberPrompt** を検証する目的で、次のような検証関数を追加できます。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

ダイアログ セットにプロンプトを追加して検証関数を追加したときの変更点

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

その後検証がその独自のメソッドとして定義され、検証に合格したかどうかによって true または false が示されます。 false が返される場合は、ユーザーに再度プロンプトを出します。

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add(new NumberPrompt('partySizePrompt', async (promptContext) => {
    // Check to make sure a value was recognized.
    if (promptContext.recognized.succeeded) {
        const value = promptContext.recognized.value;
        try {
            if (value < 6 ) {
                throw new Error('Party size too small.');
            } else if (value > 20) {
                throw new Error('Party size too big.')
            } else {
                return true; // Indicate that this is a valid value.
            }
        } catch (err) {
            await promptContext.context.sendActivity(`${ err.message } <br/>Please provide a valid number between 6 and 20.`);
            return false; // Indicate that this is invalid.
        }
    } else {
        return false;
    }
}));
```

---

同様に、将来の日付と時刻に対する **DatetimePrompt** 応答を検証する場合、次のような検証ロジックを使用できます。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

その他の例は Microsoft の[サンプル リポジトリ](https://aka.ms/bot-samples-readme)にあります。

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add(new atetimePrompt('dateTimePrompt', async (promptContext) => {
    if (promptContext.recognized.succeeded) {
        const values = promptContext.recognized.value;
        try {
            if (values.length < 0) { throw new Error('missing time') }
            if (values[0].type !== 'date') { throw new Error('unsupported type') }
            const value = new Date(values[0].value);
            if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = value;
            return true; // indicate valid 
        } catch (err) {
            await promptContext.context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
            return false; // indicate invalid
        }
    } else {
        return false;
    }
}));
```

その他の例は Microsoft の[サンプル リポジトリ](https://aka.ms/bot-samples-readme)にあります。

---

> [!TIP]
> 日時プロンプトでは、ユーザーがあいまいな回答を入力した場合、いくつかの異なる日付に解決できます。 その使用目的に基づき、最初の解決だけでなく、プロンプト結果で与えられた解決をすべて確認することをお勧めします。

同様の手法を使用し、あらゆる種類のプロンプトの回答を検証できます。

## <a name="save-user-data"></a>ユーザー データを保存する

ユーザー入力をプロンプトで求めるとき、その入力の処理方法にはいくつかの選択肢があります。 たとえば、入力を使用後に破棄したり、グローバル変数に保存したり、揮発性のある、あるいはメモリ内ストレージ コンテナーに保存したり、ファイルに保存したり、外部データベースに保存したりできます。 ユーザー データの保存方法については、[ユーザー データの管理](bot-builder-howto-v4-state.md)に関するページを参照してください。

## <a name="additional-resources"></a>その他のリソース

これらのプロンプトをいくつか使用している完全なサンプルについて詳しくは、[C#](https://aka.ms/cs-multi-prompts-sample) または [JavaScript](https://aka.ms/js-multi-prompts-sample) 形式のマルチ ターン プロンプト ボットをご覧ください。

## <a name="next-steps"></a>次の手順

これでプロンプトでユーザーに入力を求める方法がわかりました。次に、ダイアログを利用してさまざまな会話フローを管理することでボット コードとユーザー エクスペリエンスを強化しましょう。
