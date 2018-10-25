---
title: ダイアログを使用して単純な会話フローを管理する | Microsoft Docs
description: Bot Builder SDK for Node.js でダイアログを使用して単純な会話フローを管理する方法について説明します。
keywords: 単純な会話フロー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07035c8f0dfc7473192d8c51667ed1f5cefbc555
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999399"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>ダイアログを使用して単純な会話フローを管理する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローを管理できます。 単純な会話フローでは、ユーザーは "*ウォーターフォール*" の最初のステップから始め、最後のステップまで進むと会話が終了します。 [複雑な会話フロー](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)には、分岐とループが含まれます。

<!-- TODO: This paragraph belongs in a conceptual topic. -->

ダイアログとは、ボットのプログラム内で関数のように機能する、ボット内の構造のことです。 ダイアログはボットが送信するメッセージをビルドし、要求されるコンピューティング タスクを実行します。 それらは、特定の操作を、特定の順序で実行するよう設計されています。 ユーザーに応答して、外部的な刺激に反応して、別のダイアログによってなど、さまざまな方法で呼び出されることがあります。

ダイアログを使用することで、ボット開発者は会話フローを導くことができます。 複数のダイアログを作成して、それらをリンクすることで、ボットで処理するあらゆる会話フローを作成できます。 Bot Builder SDK の**ダイアログ** ライブラリには、会話フローの管理に役立つ "_プロンプト_"、"_ウォーターフォール ダイアログ_"、"_コンポーネント ダイアログ_" などの組み込み機能があります。 プロンプトを使用すると、ユーザーにさまざまな種類の情報を要求できます。 ウォーターフォールを使用すると、複数のステップを 1 つのシーケンスに結合できます。 また、コンポーネント ダイアログを使用して、複数のサブダイアログが含まれるモジュラー ダイアログ システムを作成できます。

この記事では、"_ダイアログ セット_" を使用して、プロンプトとウォーターフォールの両方を含む会話フローを作成します。 **マルチターン プロンプト**の [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)] のサンプルからコードを作成します。

ダイアログの概要については、「[ダイアログ ライブラリ](bot-builder-concept-dialog.md)」と「[ダイアログの状態](bot-builder-dialog-state.md)」をご覧ください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログ全般を使用するには、プロジェクトまたはソリューション用の `Microsoft.Bot.Builder.Dialogs` NuGet パッケージをインストールする必要があります。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ダイアログ全般を使用するには、NPM からダウンロード可能な `botbuilder-dialogs` ライブラリが必要です。

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>ダイアログを使用してユーザーにステップを案内する

この例では、ダイアログ セットを使用して、ユーザーに情報の入力を求める複数ステップ ダイアログを作成します。

### <a name="create-a-dialog-with-waterfall-steps"></a>ウォーターフォール ステップを持つダイアログを作成する

**WaterfallDialog** はダイアログ特有の実装であり、一般的にユーザーから情報を収集したり、一連のタスクをユーザーに案内したりするときに使用されます。 会話の各ステップは関数として実装されます。 各ステップで、ボットは、[ユーザーに入力を要求](bot-builder-prompts.md)し、応答を待ち、結果を次のステップに渡します。 最初の関数の結果は次の関数の引数として渡され、その次の関数でも同様です。

たとえば、次のコード サンプルでは、**ウォーター フォール**のステップを表すデリゲートの配列を定義します。 各プロンプトの後、ボットはユーザーの入力を確認します。 ダイアログで収集する入力を保持する方法は多数存在します。 「[ユーザー データを保持する](bot-builder-tutorial-persist-user-inputs.md)」でそのいくつかのオプションを確認できます。

このサンプルは、ダイアログ で収集された情報をそのまま、ユーザーのプロファイルに書き込みます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

このサンプルでは、ウォーターフォール ダイアログがボット ファイル内で定義されています。

このファイルで使用されている名前空間を参照します。

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

ダイアログ セットのインスタンス プロパティを定義します。

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

ボットのコンストラクター内にダイアログ セットを作成し、プロンプトとウォーターフォール ダイアログをそのセットに追加します。

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

また、各ステップを別個のメソッドとして定義します。 ラムダ式を使用してインラインでステップを定義することもできます。

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

ダイアログはボットのオン ターン ハンドラーから実行されます。これにより、まずダイアログのコンテキストが作成され、適宜そのダイアログを続行または開始し、その後ターンの最後に会話とユーザーの状態を保存します。

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

このサンプルでは、ウォーターフォール ダイアログが **bot.js** ファイル内で定義されています。

コードに必要なオブジェクトをインポートします。

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

ボットのコンストラクター内にダイアログ セットを定義して作成し、プロンプトとウォーターフォール ダイアログをそのセットに追加します。

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

また、各ステップを別個のメソッドとして定義します。 ラムダ式を使用してインラインでステップを定義することもできます。

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

ダイアログはボットのオン ターン ハンドラーから実行されます。これにより、まずダイアログのコンテキストが作成され、適宜そのダイアログを続行または開始し、その後ターンの最後に会話とユーザーの状態を保存します。

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>ダイアログ コンテキスト オブジェクトとウォーターフォール ステップ コンテキスト オブジェクト

ダイアログ コンテキスト オブジェクトを使用して、ボットのターン ハンドラー内からダイアログ セットと対話します。
ウォーターフォール ステップ コンテキスト オブジェクトを使用して、ウォーターフォール ステップ内からダイアログ セットと対話します。

## <a name="to-start-a-dialog"></a>ダイアログを開始するには

ダイアログを開始するには、開始する *dialogId* を、ダイアログ コンテキストの _beginDialog_、_prompt_、または _replaceDialog_ メソッドに渡します。 _beginDialog_ メソッドは、スタックの一番上にダイアログをプッシュします。_replaceDialog_ メソッドは、現在のダイアログをスタックから取り出し、置き換えるダイアログをスタックにプッシュします。

ダイアログ コンテキストの _prompt_ メソッドは、引数を受け取り、プロンプトに適切なオプションをコンストラクトするヘルパー メソッドです。その後、プロンプト ダイアログを開始します。 プロンプトの詳細については、[ユーザーへの入力の要求](bot-builder-prompts.md)に関する記事をご覧ください。

## <a name="to-end-a-dialog"></a>ダイアログを終了するには

_end dialog_ メソッドは、スタックからダイアログを取り出し、親ダイアログに省略可能な結果を返してダイアログを終了します。

ベスト プラクティスは、_endDialog_ メソッドはダイアログの最後に明示的に呼び出すことです。

## <a name="to-clear-the-dialog-stack"></a>ダイアログ スタックをクリアするには

スタックからすべてのダイアログを取り出す場合は、ダイアログ コンテキストの _cancel all dialogs_ メソッドを呼び出すことで、ダイアログ スタックをクリアできます。

## <a name="to-repeat-a-dialog"></a>ダイアログを繰り返すには

ダイアログを繰り返すには、_replace dialog_ メソッドを使用して、現在のダイアログをスタックから取り出し、置き換えるダイアログをスタックの一番上にプッシュしてそのダイアログを開始します。 これは、[複雑な会話フロー](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)を処理する優れた方法であり、メニューを管理する優れた手法でもあります。

## <a name="next-steps"></a>次の手順

ここでは、単純な会話フローを管理する方法を説明しました。次に、_replace dialog_ メソッドを活用して複雑な会話フローを処理する方法を見てみましょう。

> [!div class="nextstepaction"]
> [複雑な会話フローを管理する](bot-builder-dialog-manage-complex-conversation-flow.md)
