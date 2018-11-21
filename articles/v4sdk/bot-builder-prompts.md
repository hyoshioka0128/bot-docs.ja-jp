---
title: ダイアログ プロンプトを使用してユーザー入力を収集する | Microsoft Docs
description: Bot Builder SDK で、ダイアログ ライブラリを使用してユーザーに入力を求める方法について説明します。
keywords: プロンプト, プロンプト, ユーザー入力, ダイアログ, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 再プロンプト, 検証
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ec0cc5e942ed66c8683f8b0cc92ba7df2e36db42
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645672"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>ダイアログ プロンプトを使用してユーザー入力を収集する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

質問を投稿することで情報を収集することは、ボットがユーザーとやり取りする主な手段の 1 つです。 これは、[turn context](~/v4sdk/bot-builder-basics.md#defining-a-turn) オブジェクトの _send activity_ メソッドを使用して、その後次の受信メッセージを応答として処理することで、直接行うことができます。 ただし、Bot Builder SDK には、質問しやすくし、応答が特定のデータ型と一致するか、カスタム検証ルールを満たすように設計されたメソッドを提供する、[ダイアログ ライブラリ](bot-builder-concept-dialog.md) が用意されています。 このトピックでは、ユーザーに入力を求めるプロンプト オブジェクトを使用してこれを実現する方法について詳しく説明します。

## <a name="prompt-types"></a>プロンプトの種類

バックグラウンドでは、プロンプトは 2 つのステップから成るダイアログです。 プロンプトでは、最初のステップで入力を要求し、2 番目のステップで有効な値を返すか、最初からやり直して再度入力を要求します。

ダイアログ ライブラリには、多数の基本的なプロンプトが用意されており、それぞれ異なる種類の応答の収集に使用されます。

| Prompt | 説明 | 戻り値 |
|:----|:----|:----|
| _添付ファイル プロンプト_ | ドキュメントや画像など、1 つ以上の添付ファイルを要求します。 | "_添付ファイル_" オブジェクトのコレクション。 |
| _選択プロンプト_ | 一連のオプションから選択するよう求めます。 | "_見つかった選択肢_" オブジェクト。 |
| _確認プロンプト_ | 確認を求めます。 | ブール値。 |
| _日時プロンプト_ | 日時の入力を求めます。 | "_日時解決_" オブジェクトのコレクション。 |
| _数値プロンプト_ | 数値の入力を求めます。 | 数値。 |
| _テキスト プロンプト_ | 一般的なテキスト入力を求めます。 | 文字列。 |

ライブラリには、ユーザーの代わりに別のアプリケーションにアクセスする際に使用する "_OAuth トークン_" を取得するための "_OAuth プロンプト_" も含まれています。 認証の詳細については、[ボットに認証を追加する](bot-builder-authentication.md)方法に関する記事をご覧ください。

基本的なプロンプトでは、数を表す "ten" や "a dozen"、日時を表す "tomorrow" や "Friday at 10am" など、自然言語の入力を解釈できます。

## <a name="using-prompts"></a>プロンプトの使用

ダイアログでは、ダイアログとプロンプトの両方が同じダイアログ セットに含まれている場合にのみ、プロンプトを使用できます。

1. ダイアログの状態の状態プロパティ アクセサーを定義します。
1. ダイアログ セットを作成します。
1. プロンプトを作成し、ダイアログ セットに追加します。
1. プロンプトを使用するダイアログを作成し、ダイアログ セットに追加します。
1. ダイアログ内で、プロンプトの呼び出しを追加し、プロンプトの結果を取得します。

この記事では、プロンプトを作成する方法と、ウォーターフォール ダイアログからプロンプトを呼び出す方法について説明します。
一般的なダイアログの詳細については、[ダイアログ ライブラリ](bot-builder-concept-dialog.md)に関する記事をご覧ください。
ダイアログとプロンプトを使用する完全なボットの詳細については、[ダイアログを使用して単純な会話フローを管理する](bot-builder-dialog-manage-conversation-flow.md)方法に関する記事をご覧ください。

ダイアログ内の複数のステップおよび同じダイアログ セットの複数のダイアログで同じプロンプトを使用できます。
ただし、初期化時にカスタム検証をプロンプトに関連付けます。
そのため、同じ種類のプロンプトに異なる検証が必要な場合は、そのプロンプトの種類の複数のインスタンスを用意し、それぞれ独自の検証コードを使用する必要があります。

### <a name="create-a-prompt"></a>プロンプトを作成する

ユーザーに入力を求めるには、組み込みクラスのいずれかを使用してプロンプト (_"テキスト プロンプト"_ など) を定義し、ダイアログ セットに追加します。

* プロンプトには固定 ID があります  (識別子はダイアログ セット内で一意である必要があります)。
* プロンプトにカスタム検証コントロールを含めることができます  (「[カスタム検証](#custom-validation)」をご覧ください)。
* 一部のプロンプトでは、"_既定のロケール_" を指定できます。

一般に、ボットを初期化するときに、プロンプトとダイアログを作成し、ダイアログ セットに追加します。 ボットがユーザーからの入力を受け取ると、ダイアログ セットはプロンプトの ID を解決できます。

たとえば、次のコードでは、2 つのテキスト プロンプトを作成し、既存のダイアログ セットに追加します。 2 番目のテキスト プロンプトは、ここでは示されていない検証メソッドを参照します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここでは、`_dialogs` に既存のダイアログ セットが含まれます。`NameValidator` は検証メソッドです。

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここでは、`this.dialogs` に既存のダイアログ セットが含まれます。`NameValidator` は検証関数です。

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

ロケールは、**選択**、**確認**、**日時**、**数値**の各プロンプトの言語固有の動作を決定するために使用されます。 ユーザーからの入力に対して、使用されるロケールは次のようになります。

* チャネルがユーザーのメッセージで _locale_ プロパティを提供した場合は、それが使用されます。
* それ以外の場合、プロンプトのコンストラクターの呼び出し時に指定するか、後で設定することによって、プロンプトの "_既定のロケール_" が設定されていれば、それが使用されます。
* それ以外の場合は、ロケールとして英語 ("en-us") が使用されます。

> [!NOTE]
> ロケールは、言語または言語ファミリを表す 2、3、または 4 文字の ISO 639 コードです。

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>ウォーターフォール ダイアログからプロンプトを呼び出す

プロンプトが追加されたら、ウォーターフォール ダイアログの 1 つのステップでプロンプトを呼び出し、次のダイアログ ステップでプロンプトの結果を取得します。
ウォーターフォール ステップ内からプロンプトを呼び出すには、"_ウォーターフォール ステップ コンテキスト_" オブジェクトの _prompt_ メソッドを呼び出します。 最初のパラメーターは、使用するプロンプトの ID です。2 番目のパラメーターには、プロンプトのオプション (ユーザーに入力を求める際に使用するテキストなど) が含まれます。

ユーザーがボットと対話中であり、ボットにアクティブなウォーターフォール ダイアログが存在し、ダイアログの次のステップでプロンプトを使用するとします。

1. ユーザーがボットにメッセージを送信すると、次のことが行われます。
   1. ボットのターン ハンドラーによって、ダイアログ コンテキストが作成され、_continue_ メソッドが呼び出されます。
   1. アクティブなダイアログ (この例ではウォーターフォール ダイアログ) の次のステップに制御が移ります。
   1. このステップで、ユーザーに入力を求める、ウォーターフォール ステップ コンテキストの _prompt_ メソッドを呼び出します。
   1. ウォーターフォール ステップ コンテキストで、プロンプトがスタックにプッシュされて開始されます。
   1. プロンプトによって、入力を求めるアクティビティがユーザーに送信されます。
1. ユーザーがボットに次のメッセージを送信すると、次のことが行われます。
   1. ボットのターン ハンドラーによって、ダイアログ コンテキストが作成され、_continue_ メソッドが呼び出されます。
   1. アクティブなダイアログの次のステップに制御が移ります。これがプロンプトの 2 番目のターンになります。
   1. プロンプトによってユーザーの入力が検証されます。
      * 入力が無効の場合は、プロンプトが再度開始されて再度入力を求め、次のターンでこの一連のステップが繰り返されます。
      * それ以外の場合は、プロンプトが終了し、"_ダイアログ ターン結果_" オブジェクトが親ダイアログに返されます。 ウォーターフォール ダイアログの次のステップに制御が移り、ウォーターフォール ステップ コンテキストの _result_ プロパティでプロンプトの結果が提供されます。

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

プロンプトから制御が戻ると、ウォーターフォール ステップ コンテキストの _result_ プロパティがプロンプトの戻り値に設定されます。

次の例は、2 つの連続するウォーターフォール ステップの一部を示しています。 最初のステップでは、プロンプトを使用してユーザーに名前の入力を求めます。 2 番目のステップでは、プロンプトの戻り値を取得します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ここでは、`name` はテキスト プロンプトの ID です。`NameStepAsync` と `GreetingStepAsync` は、ウォーターフォール ダイアログの 2 つの連続するステップのデリゲートです。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ここでは、`name` はテキスト プロンプトの ID です。`nameStep` と `greetingStep` は、ウォーターフォール ダイアログの 2 つの連続するステップの関数です。

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>ボットのターン ハンドラーからプロンプトを呼び出す

ダイアログ コンテキストの _prompt_ メソッドを使用して、ターン ハンドラーから直接プロンプトを呼び出すことができます。
次のターンでダイアログ コンテキストの _continue dialog_ メソッドを呼び出し、戻り値 ("_ダイアログ ターン結果_" オブジェクト) を確認する必要があります。 この方法の例については、プロンプト検証のサンプル ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample)) を参照してください。また、別の方法については、[独自のプロンプトを使用してユーザーに入力を求める](bot-builder-primitive-prompts.md)方法に関する記事をご覧ください。

## <a name="prompt-options"></a>プロンプト オプション

_prompt_ メソッドの 2 番目のパラメーターは、_prompt options_ オブジェクトを受け取ります。このオブジェクトには、次のプロパティがあります。

| プロパティ | 説明 |
| :--- | :--- |
| _prompt_ | 入力を求めるためにユーザーに送信する最初のアクティビティ。 |
| _retry prompt_ | 最初の入力が有効ではなかった場合にユーザーに送信するアクティビティ。 |
| _choices_ | ユーザーが選択できる選択肢のリスト。選択プロンプトで使用されます。 |

一般に、prompt プロパティと retry prompt プロパティはアクティビティですが、さまざまなプログラミング言語でこれを処理する方法にいくつかのバリエーションがあります。

ユーザーに送信する最初のプロンプト アクティビティは常に指定する必要があります。

再試行プロンプトを指定すると、ユーザーの入力が、プロンプトで解析できない形式であるか (数値プロンプトに対して 「tomorrow」と入力された場合など)、検証基準を満たしていないために、検証に失敗する可能性がある場合に役立ちます。 再試行プロンプトが指定されていない場合、プロンプトでは、最初のプロンプト アクティビティを使用してユーザーに再度入力を求めます。

選択プロンプトの場合、使用可能な選択肢のリストを常に提供する必要があります。

次の例は、3 つのプロパティをすべて指定して、選択プロンプトを使用する方法を示しています。 _favorite color_ メソッドは、ウォーターフォール ダイアログのステップとして使用されます。ダイアログ セットには、ウォーターフォール ダイアログと、`colorChoice` という ID の選択プロンプトが含まれます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript SDK では、`prompt` プロパティと `retryPrompt` プロパティの両方に文字列を指定できます。 これらは、プロンプトによってメッセージ アクティビティに変換されます。

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

## <a name="custom-validation"></a>カスタム検証

**ウォーターフォール**の次のステップに値を返す前にプロンプトの応答を検証できます。 検証関数は、_prompt validator context_ パラメーターを持ち、入力が検証に合格したかどうかを示すブール値を返します。

prompt validator context には、次のプロパティが含まれます。

| プロパティ | 説明 |
| :--- | :--- |
| _コンテキスト_ | ボットの現在のターン コンテキスト。 |
| _Recognized_ | _プロンプト認識エンジンの結果_。認識エンジンによって処理された、ユーザー入力に関する情報が含まれます。 |

プロンプト認識エンジンの結果には、次のプロパティがあります。

| プロパティ | 説明 |
| :--- | :--- |
| _成功_ | 認識エンジンが入力を解析できたかどうかを示します。 |
| _値_ | 認識エンジンからの戻り値。 必要に応じて、検証コードでこの値を変更できます。 |

### <a name="setup"></a>セットアップ

検証コードを追加する前に、少し設定を行う必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットの **.cs** ファイルで、予約情報の内部クラスを定義します。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

**BotAccessors.cs** で、予約データの状態プロパティ アクセサーを追加します。

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

**Startup.cs** で、アクセサーを設定するように `ConfigureServices` を更新します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript に必要な HTTP サービス コードは変更しません。**index.js** ファイルはそのままにしておきます。

**bot.js** で、require ステートメントを更新し、状態プロパティ アクセサーの識別子を追加します。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

ボット ファイルで、ダイアログとプロンプトの識別子を追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>プロンプトとダイアログを定義する

ボットのコンストラクター コードで、ダイアログ セットを作成し、プロンプトと予約ダイアログを追加します。
プロンプトを作成するときに、カスタム検証を含めます。 次に検証機能を実装します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>検証コードを実装する

パーティの人数の検証コントロールを実装します。 予約を 6 から 20 人のパーティに制限します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task<bool> PartySizeValidatorAsync(
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
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as date.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

日時プロンプトは、ユーザー入力に一致する使用可能な "_日時解決_" のリストまたは配列を返します。 たとえば、9:00 は午前 9 時または午後 9 時を意味する可能性があり、Sunday もあいまいです。 さらに、日時解決では、日付、時刻、日時、または範囲を表すことができます。 日時プロンプトでは、[Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) を使用してユーザー入力を解析します。

予約日検証コントロールを実装します。 予約を現在の時刻から 1 時間以上に制限します。 条件に一致する最初の解決を維持し、残りをクリアします。

この検証コードは網羅的ではありません。 日付と時刻を解析する入力に最適です。 これは、日時プロンプトを検証する際のオプションを示しています。実装は、ユーザーから収集しようとしている情報によって異なります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
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
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
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

### <a name="implement-the-dialog-steps"></a>ダイアログ ステップを実装する

ダイアログ セットに追加したプロンプトを使用します。 ボットのコンストラクターでプロンプトを作成したときに、プロンプトに検証を追加しました。 プロンプトでは、初めてユーザーに入力を求めたときに、提供されているオプションから "_プロンプト_" アクティビティを送信します。 検証に失敗すると、ユーザーに別の入力を求める "_再試行プロンプト_" アクティビティが送信されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

// Second step of the main dialog: record the party size and prompt for the
// reservation date.
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

// Third step of the main dialog: return the collected party size and reservation date.
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>ターン ハンドラーを更新する

ダイアログを開始し、完了時にダイアログからの戻り値を受け入れるように、ボットのターン ハンドラーを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

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
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

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
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
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
                        `We'll see you ${reservation.date}.`);
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
                        `confirmed for ${dialogTurnResult.result.date}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        default:
            break;
    }
}
```

---

同様の手法を使用し、あらゆる種類のプロンプトの回答を検証できます。

## <a name="handling-prompt-results"></a>プロンプトの結果の処理

プロンプトの結果の処理は、ユーザーにその情報を要求した理由によって異なります。 次のオプションがあります。

* ユーザーが確認または選択プロンプトに応答する場合など、情報を使用してダイアログのフローを制御します。
* 情報をダイアログの状態にキャッシュし (ウォーターフォール ステップ コンテキストの _values_ プロパティの値の設定など)、ダイアログの終了時に収集した情報を返します。
* 情報をボットの状態に保存します。 この場合、ボットの状態プロパティ アクセサーにアクセスできるようにダイアログを設計する必要があります。

## <a name="additional-resources"></a>その他のリソース
複数のプロンプトの詳細については、[[C#](https://aka.ms/cs-multi-prompts-sample) または [JS](https://aka.ms/js-multi-prompts-sample)] のサンプルを参照してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [連続して行われる基本的な会話フローの実装](bot-builder-dialog-manage-conversation-flow.md)
