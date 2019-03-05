---
title: ダイアログ プロンプトを使用してユーザー入力を収集する | Microsoft Docs
description: Bot Framework SDK で、ダイアログ ライブラリを使用してユーザーに入力を求める方法について説明します。
keywords: プロンプト, プロンプト, ユーザー入力, ダイアログ, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 再プロンプト, 検証
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591123"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>ダイアログ プロンプトを使用してユーザー入力を収集する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

質問を投稿して情報を収集することは、ボットがユーザーとやり取りする主な手段の 1 つです。 "*ダイアログ*" ライブラリを使うことで、質問が行いやすくなるだけでなく、応答が検証され、確実に特定のデータ型と一致するように、またはカスタム検証ルールを満たすようになります。 このトピックでは、ウォーターフォール ダイアログからプロンプトを作成して呼び出す方法について詳しく説明します。

## <a name="prerequisites"></a>前提条件

- この記事のコードは、**DialogPromptBot** サンプルをベースにしています。 [C# サンプル](https://aka.ms/dialog-prompt-cs)または [JS サンプル](https://aka.ms/dialog-prompt-js)のコピーが必要です。
- [ダイアログ ライブラリ](bot-builder-concept-dialog.md)と[会話の管理](bot-builder-dialog-manage-conversation-flow.md)方法に関する基礎知識が必要です。
- テスト用の [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)。

## <a name="using-prompts"></a>プロンプトの使用

ダイアログでは、ダイアログとプロンプトの両方が同じダイアログ セットに含まれている場合にのみ、プロンプトを使用できます。 ダイアログ内の複数のステップおよび同じダイアログ セットの複数のダイアログで同じプロンプトを使用できます。 ただし、初期化時にカスタム検証をプロンプトに関連付けます。 同じ種類のプロンプトに異なる検証を使用するには、そのプロンプトの種類のインスタンスを複数用意し、それぞれに独自の検証コードを使用する必要があります。

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>ダイアログの状態の状態プロパティ アクセサーを定義する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

この記事で使用されるダイアログ プロンプトのサンプルでは、ユーザーに対して予約情報を入力するよう求めます。 パーティの人数と日付を管理するには、DialogPromptBot.cs ファイルで予約情報の内部クラスを定義します。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

次に、予約データの状態プロパティ アクセサーを追加します。

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

Startup.cs で、アクセサーを設定するように `ConfigureServices` メソッドを更新します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript に必要な HTTP サービス コードは変更しません。index.js ファイルはそのままにしておきます。

bot.js で、ダイアログ プロンプト ボットに必要な `require` ステートメントを追加します。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

状態プロパティ アクセサー、ダイアログ、およびプロンプトの識別子を追加します。

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>ダイアログ セットとプロンプトを作成する

一般的には、ボットを初期化するときに、プロンプトとダイアログを作成し、ダイアログ セットに追加します。 ボットがユーザーからの入力を受け取ると、ダイアログ セットはプロンプトの ID を解決できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`DialogPromptBot` クラスで、ダイアログ、プロンプト、およびダイアログ内で追跡する値の識別子を定義します。

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

ボットのコンストラクターで、ダイアログ セットを作成し、プロンプトと予約ダイアログを追加します。 カスタム検証は、プロンプトを作成するときに追加します。検証機能は後で実装します。

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

コンストラクターで、状態アクセサー プロパティを作成します。
次に、ダイアログ セットを作成し、プロンプト (カスタム検証を含む) を追加します。
次に、ウォーターフォール ダイアログのステップを定義し、セットに追加します。

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>ダイアログ ステップを実装する

メインのボット ファイルで、ウォーターフォール ダイアログの各ステップを実装します。 プロンプトが追加されたら、ウォーターフォール ダイアログの 1 つのステップでプロンプトを呼び出し、次のダイアログ ステップでプロンプトの結果を取得します。 ウォーターフォール ステップ内からプロンプトを呼び出すには、"_ウォーターフォール ステップ コンテキスト_" オブジェクトの _prompt_ メソッドを呼び出します。 最初のパラメーターは、使用するプロンプトの ID です。2 番目のパラメーターには、プロンプトのオプション (ユーザーに入力を求める際に使用するテキストなど) が含まれます。

これらのメソッドは、以下を示します。

- _prompt options_ を渡す方法を含め、ウォーターフォール ステップからプロンプトを呼び出す方法。
- _validations_ プロパティを使用して、カスタム検証コントロールに追加のパラメーターを提供する方法。
- _choices_ プロパティを使用して、選択プロンプトに選択肢を提供する方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**DialogPromptBot.cs** ファイルで、ウォーターフォール ダイアログのステップを実装します。

ここでは、ウォーターフォールの最初の 2 つのステップである `PromptForPartySizeAsync` と `PromptForLocationAsync` を示します。

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** ファイルで、ウォーターフォール ダイアログのステップを実装します。

ここでは、ウォーターフォールの最初の 2 つのステップである `promptForPartySize` と `promptForLocation` を示します。

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

_prompt_ メソッドの 2 番目のパラメーターは、_prompt options_ オブジェクトを受け取ります。このオブジェクトには、次のプロパティがあります。

| プロパティ | 説明 |
| :--- | :--- |
| _prompt_ | 入力を求めるためにユーザーに送信する最初のアクティビティ。 |
| _retry prompt_ | 最初の入力が有効ではなかった場合にユーザーに送信するアクティビティ。 |
| _choices_ | ユーザーが選択できる選択肢のリスト。選択プロンプトで使用されます。 |
| _validations_ | カスタム検証コントロールで使用する追加のパラメーターです。 |

一般に、prompt プロパティと retry prompt プロパティはアクティビティですが、さまざまなプログラミング言語でこれを処理する方法にいくつかのバリエーションがあります。

ユーザーに送信する最初のプロンプト アクティビティは常に指定する必要があります。

再試行プロンプトを指定すると、ユーザーの入力が、プロンプトで解析できない形式であるか (数値プロンプトに対して「tomorrow」と入力された場合など)、検証基準を満たしていないために、検証に失敗する場合に役立ちます。 再試行プロンプトが指定されていない場合、プロンプトでは、最初のプロンプト アクティビティを使用してユーザーに再度入力を求めます。

選択プロンプトの場合、使用可能な選択肢のリストを常に提供する必要があります。

## <a name="custom-validation"></a>カスタム検証

**ウォーターフォール**の次のステップに値を返す前にプロンプトの応答を検証できます。 検証関数は、_prompt validator context_ パラメーターを持ち、入力が検証に合格したかどうかを示すブール値を返します。

prompt validator context には、次のプロパティが含まれます。

| プロパティ | 説明 |
| :--- | :--- |
| _コンテキスト_ | ボットの現在のターン コンテキスト。 |
| _Recognized_ | _プロンプト認識エンジンの結果_。認識エンジンによって処理された、ユーザー入力に関する情報が含まれます。 |
| _オプション_ | プロンプトを開始する呼び出しで提供された _prompt options_ が含まれます。 |

プロンプト認識エンジンの結果には、次のプロパティがあります。

| プロパティ | 説明 |
| :--- | :--- |
| _成功_ | 認識エンジンが入力を解析できたかどうかを示します。 |
| _値_ | 認識エンジンからの戻り値。 必要に応じて、検証コードでこの値を変更できます。 |

### <a name="implement-validation-code"></a>検証コードを実装する

ボットのコンストラクターで、初期化時にカスタム検証をプロンプトに関連付けます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**パーティの人数の検証コントロール**

予約できるパーティの人数を制限します。 有効な範囲は、パーティの人数プロンプトの呼び出しに使用した _validations_ プロパティによって定義されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**日時の検証**

予約日の検証コントロールで、予約時刻を現在の時刻から 1 時間以上後に制限します。 条件に一致する最初の解決を維持し、残りをクリアします。

この検証コードはすべてを網羅しているわけではなく、日付と時刻を解析する入力に適しています。 ここで示しているのは、日時プロンプトを検証する場合のオプションです。実装は、ユーザーから収集しようとしている情報によって異なります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

日時プロンプトは、ユーザー入力に一致する使用可能な "_日時解決_" のリストまたは配列を返します。 たとえば、9:00 は午前 9 時または午後 9 時を意味する可能性があり、Sunday もあいまいです。 さらに、日時解決では、日付、時刻、日時、または範囲を表すことができます。 日時プロンプトでは、[Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) を使用してユーザー入力を解析します。

## <a name="update-the-turn-handler"></a>ターン ハンドラーを更新する

ダイアログを開始し、完了時にダイアログからの戻り値を受け入れるように、ボットのターン ハンドラーを更新します。 ここでは、ユーザーがボットと対話中であり、ボットにアクティブなウォーターフォール ダイアログが存在し、ダイアログの次のステップでプロンプトを使用することを想定しています。

ユーザーがボットにメッセージを送信すると、次のことが行われます。

1. ボットは、状態情報を取得します。
1. ボットは、ダイアログ コンテキストを作成します。
    - アクティブなダイアログがなく、ユーザーがまだ予約をしていない場合、ボットはダイアログを開始します。
    - アクティブなダイアログがある場合、ボットはそのダイアログを続けます。 ダイアログが終了した場合は、予約の詳細が状態キャッシュに記録されます。
1. ボットは、状態の変更をすべて保存します。

ダイアログのステップがステップ コンテキストの _prompt_メソッドを呼び出すと、次のことが行われます。

1. プロンプトの新しいインスタンスが作成され、ダイアログ スタックに配置され、開始されます  (メイン ダイアログは、プロンプトの終了を待機してから続行されます)。
1. プロンプトによって、入力を求めるアクティビティがユーザーに送信されます。

入力がプロンプトに送信されると、次のことが行われます。

1. プロンプトは、プロンプトの種類 (数値プロンプトや選択プロンプトなど) に応じて入力の処理を試みます。
1. プロンプトにカスタム検証が含まれている場合は、カスタム検証コードが実行されます。
1. 入力がすべての検証に合格した場合は、プロンプトが終了し、処理された入力が返されます。それ以外の場合は、プロンプトが再度開始されます。

**プロンプトの結果の処理**

プロンプトの結果の処理は、ユーザーにその情報を要求した理由によって異なります。 次のオプションがあります。

- ユーザーが確認または選択プロンプトに応答する場合など、情報を使用してダイアログのフローを制御します。
- 情報をダイアログの状態にキャッシュし (ウォーターフォール ステップ コンテキストの _values_ プロパティの値の設定など)、ダイアログの終了時に収集した情報を返します。
- 情報をボットの状態に保存します。 この場合、ボットの状態プロパティ アクセサーにアクセスできるようにダイアログを設計する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

同様の手法を使用し、あらゆる種類のプロンプトの回答を検証できます。

## <a name="test-your-bot"></a>ボットをテストする

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、README ファイルで [C#](https://aka.ms/dialog-prompt-cs) または [JS](https://aka.ms/dialog-prompt-js) を参照してください。
2. エミュレーターを起動し、次に示すように、メッセージを送信してボットをテストします。

![ダイアログ プロンプト サンプルをテストする](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>その他のリソース

お使いのターン ハンドラーからプロンプトを直接呼び出すには、[C# ](https://aka.ms/cs-prompt-validation-sample) または [JS](https://aka.ms/js-prompt-validation-sample) の _prompt-validations_ サンプルを参照してください。

ダイアログ ライブラリには、ユーザーの代わりに別のアプリケーションにアクセスするときに使用する "_OAuth トークン_" を取得するための "_OAuth プロンプト_" も含まれています。 認証の詳細については、お使いのボットに[認証を追加する](bot-builder-authentication.md)方法に関する記事をご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ブランチとループを使用して高度な会話フローを作成する](bot-builder-dialog-manage-complex-conversation-flow.md)
