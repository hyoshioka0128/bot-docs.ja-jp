---
title: ダイアログ ライブラリを使用してユーザーに入力を求める | Microsoft Docs
description: Bot Builder SDK for Node.js で、ダイアログ ライブラリを使用してユーザーに入力を求める方法について説明します。
keywords: プロンプト, ダイアログ, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 再プロンプト, 検証
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905366"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>ダイアログ ライブラリを使用してユーザーに入力を求める

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットでは多くの場合、ユーザーに提示した質問を介してその情報を収集します。 単純に[ターン コンテキスト](bot-builder-concept-activity-processing.md#turn-context) オブジェクトの _send activity_ メソッドを使用して標準のメッセージをユーザーに送信し、文字列入力を要求できますが、Bot Builder SDK には**ダイアログ** ライブラリがあり、そのライブラリを使用し、さまざまな種類の情報を要求できます。 このトピックでは、**プロンプト**を使用してユーザーに入力を求める方法について説明します。

この記事では、ダイアログ内でプロンプトを使用する方法について説明します。 一般的なダイアログの使用に関する情報については、[ダイアログを使用した単純な会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関するページをご覧ください。

## <a name="prompt-types"></a>プロンプトの種類

ダイアログ ライブラリには、さまざまな種類のプロンプトがあり、それぞれで異なる種類の応答が要求されます。

| Prompt | 説明 |
|:----|:----|
| **AttachmentPrompt** | 文書や画像など、添付ファイルをユーザーに求めます。 |
| **ChoicePrompt** | 一連の選択肢から選択するようにユーザーに求めます。 |
| **ConfirmPrompt** | 自分のアクションを確定するようにユーザーに求めます。 |
| **DatetimePrompt** | 日時をユーザーに求めます。 "明日の午後 8 時" や "金曜日の午前 10 時" など、自然言語を利用してユーザーは回答できます。 Bot Framework SDK では、LUIS `builtin.datetimeV2` 事前作成済みエンティティが使用されます。 詳細については、「[builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)」を参照してください。 |
| **NumberPrompt** | 数字をユーザーに求めます。 ユーザーは "10" か "ten" で回答できます。 回答がたとえば "ten" の場合、プロンプトにより応答が数字に変換され、結果として `10` が返されます。 |
| **TextPrompt** | テキストの文字列をユーザーに求めます。 |

## <a name="add-references-to-prompt-library"></a>プロンプト ライブラリに参照を追加する

**ダイアログ** パッケージをボットに追加することで**ダイアログ** ライブラリを取得できます。 ダイアログについては、[ダイアログを使用した単純な会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関するページで取り上げますが、Microsoft ではプロンプトにダイアログを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

NuGet から **Microsoft.Bot.Builder.Dialogs** パッケージをインストールします。

次に、ボット コードからライブラリの参照を追加します。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

ボット コード ファイルでダイアログをクラスとして定義するか、プロパティとしてインラインで定義できます。

この記事にあるコードは、ダイアログをクラスとして定義して記述されています。
次の例では、ダイアログのコンストラクターにコードを追加します。

ダイアログのメイン フローは手順の収集であり、ID を与える必要があります。 ボットではこの ID を使用してダイアログを取得します。そのため、これを定数として公開することをお勧めします。

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

NPM からダイアログ パッケージをインストールします。

```cmd
npm install --save botbuilder-dialogs@preview
```

ボットで**ダイアログ**を使用するには、それをボット コードに含めます。

app.js ファイルで、次を追加します。

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>ユーザーに入力を求める

ユーザーに入力を求める目的で、プロンプトをダイアログに追加できます。 たとえば、**TextPrompt** タイプのプロンプトを定義し、それに **textPrompt** というダイアログ ID を与えることができます。

プロンプト ダイアログが追加されたら、単純な 2 ステップ ウォーターフォール ダイアログでそれを使用するか、多ステップ ウォーターフォールで複数のプロンプトをまとめて使用できます。 *ウォーターフォール* ダイアログは要するに一連のステップを定義する手段です。 詳細については、[ダイアログを使用した単純なな会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関する記事の[ダイアログの使用](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)に関するセクションをご覧ください。

最初のターンで、ダイアログからユーザーに名前の入力が求められます。2 回目のターンでユーザー入力がプロンプトへの回答として処理されます。

たとえば、次のダイアログでは、ユーザーに名前の入力が求められ、名前であいさつされます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログで使用する各プロンプトにも名前が表示されます。その名前はプロンプトにアクセスする目的でダイアログまたはボットにより使用されます。 以上のすべてのサンプルで、プロンプト ID を定数として公開しています。

ダイアログ コンテキストの **Prompt** または **End** メソッドを呼び出すと、ダイアログにステップの終わりが伝えられます。 これらのステートメントなしでは、ダイアログは適切に実行されません。

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> ダイアログを開始するには、ダイアログ コンテキストを取得し、その _begin_ メソッドを使用します。 詳細については、[ダイアログを使用した単純な会話フローの管理](./bot-builder-dialog-manage-conversation-flow.md)に関するページをご覧ください。

## <a name="reusable-prompts"></a>再利用可能プロンプト

プロンプトを再利用し、同じ種類のプロンプトでさまざまな情報を求めることができます。 たとえば、前出のサンプル コードではテキスト プロンプトが定義され、それを利用してユーザーに名前を求めました。 その同じプロンプトを利用し、「職場はどこですか」など、別のテキスト文字列を求めることもできます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

ただし、プロンプトとそのプロンプトで求める値のペアを作る場合、各プロンプトに一意の *dialogId* を与えることができます。 一意の ID でダイアログが追加されます。 さまざまな ID を使用し、同じ種類の**プロンプト** ダイアログを複数作成することもできます。 たとえば、上記の例には 2 つの **TextPrompt** ダイアログを作成できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

コードの再利用性のために、`textPrompt` を 1 つ定義すると、応答としてテキスト文字列を求めるため、これらすべてのプロンプトでうまく動作します。 ただし、ダイアログに名前を付けることは、プロンプトの入力を検証する必要があるときに便利です。 その場合、プロンプトでは **TextPrompt** が使用されますが、それぞれで別の値セットが求められます。 `NumberPrompt` を利用してプロンプトの応答を検証する方法を見てみましょう。

## <a name="specify-prompt-options"></a>プロンプト オプションを指定する

ダイアログ ステップ内でプロンプトを使用するとき、再プロンプト文字列など、プロンプト オプションを提供することもできます。

再プロンプト文字列の指定は、数値の求めに対して "明日" が回答されるなどの解析できない形式であるか、入力で検証基準を満たせないときなどの、ユーザー入力がプロンプトを満たせない場合に便利です。

> [!TIP]
> 数値を求めるプロンプトを作成するときは、そのプロンプトで使用される入力カルチャを指定する必要があります。 数値を求めるプロンプトでは、"twelve" や "4 分の 1"、"12" や "0.25" など、さまざまな入力を解釈できます。 入力カルチャにより、プロンプトではユーザーの入力がより正確に解釈されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

入力カルチャは追加ライブラリで定義されます。

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

次のコードでは、既存のダイアログ セット **dialogs** に数値を求めるプロンプトが追加されます。

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

ダイアログ ステップ内で、次のコードにより、ユーザーに入力が求められ、その入力が数値として解釈できない場合に使用する再プロンプト文字列が用意されます。

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

特に、選択肢プロンプトでは、追加情報がいくつか必要になります。ユーザーが選択できる選択肢の一覧です。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

この例では、次の名前空間からの型が使用されます。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


**ChoicePrompt** を使用して一連の選択肢から選択するようにユーザーに求めるとき、その一連の選択肢を含むプロンプトを提供する必要があります。これは **ChoicePromptOptions** オブジェクト内で提供します。 ここでは、**ChoiceFactory** を使用し、選択肢の一覧を適切な形式に変換しています。

また、ユーザーに入力選択肢を再び提供する方法として、**SuggestedActions** アクティビティを再プロンプトとして使用しています。


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>プロンプトの応答を検証する

**ウォーターフォール**の次のステップに有効な値を返す前にプロンプトの応答を検証できます。 たとえば、**6** から **20** までの数値範囲内で **NumberPrompt** を検証する目的で、次のような検証ロジックを使用できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

検証は独自のプライベート メソッド内でカプセル化し、追加することもできます。

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

ダイアログ内で、入力の検証に使用するメソッドを指定します。

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

同様に、将来の日付と時刻に対する **DatetimePrompt** 応答を検証する場合、次のような検証ロジックを使用できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

その他の例は Microsoft の[サンプル リポジトリ](https://github.com/Microsoft/botbuilder-dotnet)にあります。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

その他の例は Microsoft の[サンプル リポジトリ](https://github.com/Microsoft/botbuilder-js)にあります。

---

> [!TIP]
> 日時プロンプトでは、ユーザーがあいまいな回答を入力した場合、いくつかの異なる日付に解決できます。 その使用目的に基づき、最初の解決だけでなく、プロンプト結果で与えられた解決をすべて確認することをお勧めします。

同様の手法を使用し、あらゆる種類のプロンプトの回答を検証できます。

## <a name="save-user-data"></a>ユーザー データを保存する

ユーザー入力をプロンプトで求めるとき、その入力の処理方法にはいくつかの選択肢があります。 たとえば、入力を使用後に破棄したり、グローバル変数に保存したり、揮発性のある、あるいはメモリ内ストレージ コンテナーに保存したり、ファイルに保存したり、外部データベースに保存したりできます。 ユーザー データの保存方法については、[ユーザー データの管理](bot-builder-howto-v4-state.md)に関するページを参照してください。

## <a name="next-steps"></a>次の手順

これでプロンプトでユーザーに入力を求める方法がわかりました。次に、ダイアログを利用してさまざまな会話フローを管理することでボット コードとユーザー エクスペリエンスを強化しましょう。


