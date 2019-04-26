---
title: 連続して行われる会話フローの実装 | Microsoft Docs
description: Bot Framework SDK for Node.js でダイアログを使用して単純な会話フローを管理する方法について説明します。
keywords: 単純な会話フロー, 連続して行われる会話フロー, ダイアログ, プロンプト, ウォーターフォール, ダイアログ セット
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5361b2e411e12b296b60a0f27b560dee5f1f769f
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904865"
---
# <a name="implement-sequential-conversation-flow"></a>連続して行われる会話フローの実装

[!INCLUDE[applies-to](../includes/applies-to.md)]

ダイアログ ライブラリを使用して、単純な会話フローと複雑な会話フローを管理できます。 単純なインタラクションでは、ボットは決まった一連のステップを順番に実行していき、最後に会話が終了します。 この記事では "_ウォーターフォール ダイアログ_"、いくつかの "_プロンプト_"、および "_ダイアログ セット_" を使用して、ユーザーに一連の質問を行う単純なインタラクションを作成します。

## <a name="prerequisites"></a>前提条件
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- この記事のコードは、**multi-turn-prompt** サンプルをベースにしています。 サンプルのコピー ([C#](https://aka.ms/cs-multi-prompts-sample) または [JS](https://aka.ms/js-multi-prompts-sample)) が必要になります。
- [ボットの基本](bot-builder-basics.md)、[ダイアログ ライブラリ](bot-builder-concept-dialog.md)、[ダイアログの状態](bot-builder-dialog-state.md)、および [.bot](bot-file-basics.md) ファイルに関する知識。


以降のセクションで示す手順は、ほとんどのボットで単純なダイアログを実装するときに使用できます。

## <a name="configure-your-bot"></a>ボットを構成する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** ファイルの構成コードで、ボットのダイアログ状態に対して状態プロパティ アクセサーを初期化します。

ボットの状態管理オブジェクトと状態プロパティ アクセサーを保持する `MultiTurnPromptsBotAccessors` クラスを定義します。
ここでは、コードの一部のみを呼び出します。

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

`Startup` クラスの `ConfigureServices` メソッドでアクセサー クラスを登録します。
再度、コードの一部のみを呼び出します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

依存関係の挿入によって、アクセサーをボットのコンストラクター コードで使用できるようになります。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Index.js** ファイルで、状態管理オブジェクトを定義します。
ここでは、コードの一部のみを呼び出します。

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

ボットのコンストラクターにより、ボットの状態プロパティ アクセサー `this.dialogState` および `this.userProfile` が作成されます。

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>ダイアログを呼び出すようにボットのターン ハンドラーを更新する

ダイアログを実行するには、ボットのターン ハンドラーで、ボットのダイアログが含まれるダイアログ セットに対して、ダイアログ コンテキストを作成する必要があります  ボットでは複数のダイアログ セットを定義できますが、一般的には、お使いのボットに対して 1 つだけ定義します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログは、ボットのターン ハンドラーから実行されます。 ハンドラーによって、まず `DialogContext` が作成され、アクティブ ダイアログが続行されるか、必要に応じて新しいダイアログが開始されます。 その後、ターンの終了時に、ハンドラーによって会話とユーザーの状態が保存されます。

`MultiTurnPromptsBot` クラスでは、ダイアログ セットが格納される `_dialogs` プロパティを定義しました。このプロパティから、ダイアログ コンテキストを生成します。 ここでも、ターン ハンドラー コードの一部のみを示しています。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット コードでは、ダイアログ ライブラリのクラスがいくつか使用されます。

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

ダイアログは、ボットのターン ハンドラーから実行されます。 ハンドラーによって、まず `DialogContext` (`dc`) が作成され、アクティブ ダイアログが続行されるか、必要に応じて新しいダイアログが開始されます。 その後、ターンの終了時に、ハンドラーによって会話とユーザーの状態が保存されます。

`MultiTurnBot` クラスは、**bot.js** ファイルで定義されています。 このクラスのコンストラクターによってダイアログ セットの `dialogs` プロパティが追加されます。このプロパティから、ダイアログ コンテキストを生成します。 このボットでは、`WHO_ARE_YOU` ダイアログを使って、ユーザー データが 1 度だけ収集されます。 ユーザー プロファイルが設定されると、ボットは `HELLO_USER` ダイアログを使って応答します。 ここでも、ターン ハンドラー コードの一部のみを示しています。

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

ボットのターン ハンドラーでは、ダイアログ セット用のダイアログ コンテキストを作成します。 ダイアログ コンテキストはボットの状態キャッシュにアクセスして、最後のターンで会話が中断された場所を効果的に記憶します。

アクティブ ダイアログがある場合、ダイアログ コンテキストの _continue dialog_ メソッドが、このターンをトリガーしたユーザー入力を使用して、そのダイアログを進行させます。それ以外の場合、ボットによってダイアログ コンテキストの _begin dialog_ メソッドが呼び出され、ダイアログが開始されます。

最後に、状態管理オブジェクトで _save changes_ メソッドを呼び出して、このターンで発生した変更を保持します。

### <a name="about-dialog-and-bot-state"></a>ダイアログとボットの状態について

このボットでは、2 つの状態プロパティ アクセサーを定義しました。

* 1 つはダイアログ状態プロパティ用の会話状態内で作成されます。 ダイアログ状態が追跡するのは、ダイアログ セットのダイアログ内におけるユーザーの場所です。この状態は、begin dialog メソッドや continue dialog メソッドを呼び出すときなど、ダイアログ コンテキストによって更新されます。
* もう 1 つはユーザー プロファイル プロパティ用のユーザー状態内で作成されます。 これは、ユーザーに関して持っている情報を追跡するためにボットによって使用されます。この状態は、ボット コードで明示的に管理します。

状態管理オブジェクトのキャッシュのプロパティ値は、状態プロパティ アクセサーの _get_ メソッドおよび _set_ メソッドによって取得および設定されます。 キャッシュはターンで状態プロパティの値が最初に要求されたときに設定されますが、明示的に保持する必要があります。 この両方の状態プロパティに対する変更を保持するには、対応する状態管理オブジェクトの _save changes_  メソッドを呼び出します。

## <a name="initialize-your-bot-and-define-your-dialog"></a>ボットを初期化してダイアログを定義する

簡単な会話は、ユーザーに対する一連の質問としてモデル化されます。 C# と JavaScript の各バージョンでは若干手順が異なります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 名前を尋ねます。
1. 年齢を指定するかどうかを尋ねます。
1. 指定する場合は年齢を尋ね、指定しない場合は、このステップをスキップします。
1. 収集された情報に間違いが無いかどうかを尋ねます。
1. 状態メッセージを送信し、終了します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`who_are_you` ダイアログの場合:

1. 名前を尋ねます。
1. 年齢を指定するかどうかを尋ねます。
1. 指定する場合は年齢を尋ね、指定しない場合は、このステップをスキップします。
1. 状態メッセージを送信し、終了します。

`hello_user` ダイアログの場合:

1. ボットが収集したユーザー情報を表示します。

---

ご自身のウォーターフォール ステップを定義するときに覚えておく必要があることをいくつか次に示します。

* 各ボット ターンにはユーザーからの入力が反映され、ボットからの応答が続きます。 したがって、ウォーターフォール ステップの最後にユーザーに入力を求め、その回答は次のウォーターフォール ステップで受け取ります。
* 各プロンプトは事実上、プロンプトを表す 2 つのステップから成るダイアログであり、"有効な" 入力を受け取るまでループします  

このサンプルでは、ダイアログはボット ファイル内で定義され、ボットのコンストラクターで初期化されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ダイアログ セットのインスタンス プロパティを定義します。

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

ボットのコンストラクター内にダイアログ セットを作成し、プロンプトとウォーターフォール ダイアログをそのセットに追加します。

```csharp
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

このサンプルでは、各ステップを個別のメソッドとして定義します。 ラムダ式を使用して、コンストラクターのインラインでステップを定義することもできます。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

このサンプルでは、ウォーターフォール ダイアログが **bot.js** ファイル内で定義されています。

状態プロパティ アクセサー、プロンプト、およびダイアログに使用する識別子を定義します。

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

ボットのコンストラクター内にダイアログ セットを定義して作成し、プロンプトとウォーターフォール ダイアログをそのセットに追加します。
`NumberPrompt` には、ユーザーが年齢として 0 より大きい値を入力することを確認するカスタム検証が含まれています。

```javascript
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

このダイアログのステップ メソッドはインスタンス プロパティを参照するので、`this` オブジェクトが各ステップ メソッド内で正しく解決されるように、`bind` メソッドを使用する必要があります。

このサンプルでは、各ステップを個別のメソッドとして定義します。 ラムダ式を使用して、コンストラクターのインラインでステップを定義することもできます。

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

---

このサンプルでは、ダイアログ内からユーザー プロファイルの状態が更新されます。 この方法は単純なボットには有効ですが、ボットの枠を越えてダイアログを再利用する場合は機能しません。

ダイアログ ステップとボットの状態を別々に維持するオプションには、さまざまな種類があります。 たとえば、お使いのダイアログで完全な情報を収集すると、以下を行うことができます。

* _end dialog_ メソッドを使用して、収集したデータを戻り値として親コンテキストに提供する。 これは、ボットのターン ハンドラーでもダイアログ スタック上の前のアクティブ ダイアログでもかまいません。 プロンプト クラスはこのように設計されています。
* 適切なサービスへの要求を生成する。 お使いのボットが大規模なサービスへのフロントエンドとして動作している場合にうまく機能することがあります。

## <a name="test-your-dialog"></a>ダイアログをテストする

ご自身のボットをローカルでビルドして実行し、エミュレーターを使って操作します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. ユーザーが会話に追加される、会話の更新アクティビティへの応答として、ボットから最初のあいさつメッセージが送信されます。ます。
1. `hi` などを入力します。 このターンにはまだアクティブ ダイアログがないため、ボットによって `details` ダイアログが開始されます。
   * ボットはダイアログの最初のプロンプトを送信し、入力が追加されるのを待ちます。
1. ボットからの質問に答えて、ダイアログを進めます。
1. ダイアログの最後のステップで、ご自身の入力に基づいて `Thanks` メッセージが送信されます。
   * ダイアログが終了すると、ダイアログ スタックから削除され、ボットにはアクティブ ダイアログがなくなります。
1. `hi` などと入力して、ダイアログを再開します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. ユーザーが会話に追加される、会話の更新アクティビティへの応答として、ボットから最初のあいさつメッセージが送信されます。ます。
1. `hi` などを入力します。 このターンにはまだアクティブ ダイアログもユーザー プロファイルもため、ボットによって `who_are_you` ダイアログが開始されます。
   * ボットはダイアログの最初のプロンプトを送信し、入力が追加されるのを待ちます。
1. ボットからの質問に答えて、ダイアログを進めます。
1. ダイアログの最後のステップで、簡単な確認メッセージが送信されます。
1. `hi` などを入力します。
   * ボットによって 1 ステップの `hello_user` ダイアログが開始され、収集されたデータの情報が表示された後、すぐに終了します。

---

## <a name="additional-resources"></a>その他のリソース
ここで示すように、プロンプトの種類ごとに組み込みの検証を使用することも、独自のカスタム検証をプロンプトに追加することもできます。 詳細については、[ダイアログ プロンプトを使用したユーザー入力の収集](bot-builder-prompts.md)に関するページをご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ブランチとループを使用して高度な会話フローを作成する](bot-builder-dialog-manage-complex-conversation-flow.md)
