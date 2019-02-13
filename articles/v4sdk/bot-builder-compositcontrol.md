---
title: ダイアログの再利用 | Microsoft Docs
description: Bot Framework SDK for Node.js および C# でダイアログ コンテナーを使用して、ボット ロジックをモジュール化する方法について説明します。
keywords: 複合コントロール、モジュラー ボット ロジック
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b72ffa951e176a174dd8b00e69229b27bf28a360
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711996"
---
# <a name="reuse-dialogs"></a>ダイアログの再利用

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーに挨拶する、夕食のテーブルを予約する、食べ物を注文する、アラームを設定する、現在の天気を表示するなど、複数のタスクを処理するホテル ボットを作成しているとします。 これらの各タスクは、1 つのダイアログ オブジェクトを使用してボット内で処理できますが、これによりダイアログ コードが大きくなりすぎて乱雑になることがあります。

会話フローの一部をコンポーネント ダイアログに変換し、大きなダイアログ セットを、より管理しやすい単位に分割することができます。 そうすることでコードが整理され、デバッグの手間が軽減されるほか、ボットの作業に複数のチームで同時に取り組めるようになります。
また、コンポーネント ダイアログのライブラリを作成すれば、他のダイアログ セットやボットで再利用することもできます。

この例では、チェックイン、目覚まし、テーブル予約の各コンポーネント ダイアログを組み合わせたホテル ボットを作成します。

この記事は、[単純](bot-builder-dialog-manage-conversation-flow.md)な会話フローと[複雑](bot-builder-dialog-manage-complex-conversation-flow.md)な会話フローの管理についての情報に基づいています。

## <a name="create-your-project"></a>プロジェクトを作成する

エコー ボット テンプレートから開始して、プロジェクトにダイアログ ライブラリを追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot** テンプレートをベースにして開始します。 手順については、[.NET のクイック スタート](../dotnet/bot-builder-dotnet-sdk-quickstart.md)をご覧ください。

ダイアログを使用するには、プロジェクトまたはソリューション用の `Microsoft.Bot.Builder.Dialogs` NuGet パッケージをインストールします。
次に、必要に応じてコード ファイル内の using ステートメントでダイアログ ライブラリを参照します。

**EchoBotWithCounter.cs** ファイルの名前を **HotelBot.cs** に変更し、クラス名を **HotelBot** に変更します。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Echo** テンプレートをベースにして開始します。 手順については、[JavaScript のクイック スタート](../javascript/bot-builder-javascript-quickstart.md)を参照してください。

`botbuilder-dialogs` ライブラリは、npm からダウンロードできます。 `botbuilder-dialogs` ライブラリをインストールするには、次の npm コマンドを実行します。

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>状態の管理

複合ダイアログを使用するボットの状態管理を設定するには、多くの方法があります。 このボットでは、各コンポーネント ダイアログから、その結果としてのオブジェクトが返されます。 返された値は、呼び出し元のコンテキストで管理されます。 状態管理の詳細については、「[Save state using conversation and user properties (会話およびユーザー プロパティを使用した状態の保存)](bot-builder-howto-v4-state.md)」を参照してください。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

各ダイアログで何らかの情報が収集され、ボットのターン ハンドラーまたはメイン メニューによって、この情報がユーザー状態に保存されます。 チェックイン、テーブルの予約、アラームの設定に使用するコンポーネント ダイアログを定義します。 それらはいずれも適切なクラスのオブジェクトを返します。 以下の各クラスを新しい C# クラス モジュールとしてプロジェクトに追加してください。

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}
```

**EchoBotAccessors.cs** の名前を **BotAccessors.cs** に変更し、クラス名を **BotAccessors** に更新します。

その後、次のコードを使用してファイルを更新します。 収集したユーザーの情報とダイアログの状態の両方について、アクセサーが必要です。

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

**Startup.cs** ファイルで、状態を設定する `ConfigureServices` メソッドとボットの状態プロパティ アクセサーのコードを更新します。

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

各ダイアログで何らかの情報が収集され、ボットのターン ハンドラーまたはメイン メニューによって、この情報がユーザー状態に保存されます。 チェックイン、テーブルの予約、アラームの設定に使用するコンポーネント ダイアログを定義します。 それらはいずれも適切なプロパティを備えたオブジェクトを返します。 これらのプロパティは、ボットのユーザー状態に集約することになります。

**bot.js** ファイル内のボットのコンストラクターを更新して、ユーザーの状態とダイアログの状態を追跡する状態プロパティ アクセサーを作成します。

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

**index.js** ファイルで、`botbuilder` ライブラリからインポートされたクラスと、状態オブジェクトおよびボットを作成するコードを更新します。

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="about-component-dialogs"></a>コンポーネント ダイアログについて

コンポーネント ダイアログを使用すると、大規模なダイアログ セットをより管理しやすい要素に分割して、特定のシナリオを処理する独立したダイアログを作成できます。 各要素には独自のダイアログ セットがあり、その要素を含むダイアログ セットとの名前の競合を回避しています。

コンポーネント ダイアログにダイアログとプロンプトを追加するには、_add dialog_ メソッドを使用します。
このメソッドを使用して追加した最初の項目が初期ダイアログとして設定されますが、コンポーネント ダイアログのコンストラクターで `InitialDialogId` プロパティを明示的に設定することで、これを変更できます。
コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。

## <a name="define-the-check-in-component-dialog"></a>チェックイン コンポーネント ダイアログを定義する

まず、簡単なチェックイン ダイアログから始めます。このダイアログでは、ユーザーに名前と滞在する部屋を尋ねます。 `ComponentDialog` を拡張する `CheckInDialog` クラスを作成します。 このクラスにはルート ダイアログの名前を定義するコンストラクターがあり、このダイアログは 3 つのステップを含んだウォーターフォール ダイアログとして定義します。

チェックイン ダイアログには、次のステップがあります。

1. ゲストの名前を尋ねます。
1. 滞在を希望する部屋を尋ねます。
1. 確認メッセージを送信し、ダイアログを完了します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

プロジェクトに `CheckInDialog` クラスを追加します。以下のコードを使用してください。

このダイアログでは、ステップ コンテキストの `Values` プロパティからアクセス可能なローカル状態オブジェクトに書き込むことができます。 ダイアログが完了すると、ローカル状態オブジェクトは破棄されます。 そのため、ダイアログからは、ゲスト情報を含んだ値を返します。

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the ID assigned in the parent.
    public CheckInDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkInDialog.js** ファイルを作成し、`ComponentDialog` を拡張する `CheckInDialog` クラスを追加します。

このダイアログでは、ステップ コンテキストの `values` プロパティからアクセス可能なローカル状態オブジェクトに書き込むことができます。 ダイアログが完了すると、ローカル状態オブジェクトは破棄されます。 そのため、ダイアログからは、ゲスト情報を含んだ値を返します。

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class CheckInDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>テーブル予約と目覚ましのコンポーネント ダイアログを定義する

コンポーネント ダイアログを使用する利点の 1 つは、異なるダイアログをまとめて使用できることです。 各 `DialogSet` は `dialogs` の排他的なセットを維持するので、`dialogs` の共有または相互参照は容易に実行できません。 ここでコンポーネント ダイアログの出番です。 会話フローの複雑な部分または入り組んだ部分は、コンポーネント ダイアログにカプセル化することで、ダイアログの管理とメンテナンスが容易になります。 他の 2 つのコンポーネント ダイアログを作成します。1 つは夕食のために予約するテーブルをユーザーにたずねるダイアログ、もう 1 つは目覚ましコールを作成するダイアログです。 ここでも、各ダイアログに個別のクラスを使用し、各ダイアログはメインの `ComponentDialog` を拡張します。

起動時にダイアログ オプションでゲストの情報を受け付けるように、これらを設計します。

テーブル予約ダイアログには、次のステップがあります。

1. 予約するテーブルを尋ねます。
1. 確認メッセージを送信し、ダイアログを完了して、テーブル番号を返します。

目覚ましダイアログには、次のステップがあります。

1. 設定するウェイクアップ時間を尋ねます。
1. 確認メッセージを送信し、ダイアログを完了して、アラーム時刻を返します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

プロジェクトに `ReserveTableDialog` クラスを追加します。以下のコードを使用してください。

ダイアログの開始時に渡された options オブジェクトからゲストの名前を取得します。 ここでは次に、テーブル番号が含まれた値をダイアログから返します。

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

プロジェクトに `SetAlarmDialog` クラスを追加します。以下のコードを使用してください。

ダイアログの開始時に渡された options オブジェクトからゲストの部屋番号を取得します。 次に、目覚ましコールの指定時刻が含まれた値をダイアログから返します。

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTableDialog.js** ファイルを作成し、`ComponentDialog` を拡張する `ReserveTableDialog` クラスを追加します。

ダイアログの開始時に渡された options オブジェクトからゲストの名前を取得します。 ここでは次に、テーブル番号が含まれた値をダイアログから返します。

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class ReserveTableDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

**setAlarmDialog.js** ファイルを作成し、`ComponentDialog` を拡張する `SetAlarmDialog` クラスを追加します。

ダイアログの開始時に渡された options オブジェクトからゲストの部屋番号を取得します。 次に、目覚ましコールの指定時刻が含まれた値をダイアログから返します。

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class SetAlarmDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new DateTimePrompt('datePrompt'));

        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>ボットにコンポーネント ダイアログを追加する

3 つのコンポーネント ダイアログを定義したところで、ボットでそれらを使用することができます。

- 3 つのダイアログすべてがボットのメイン ダイアログ セットに追加されます。
- 新しい会話の開始時には、アクティブなダイアログは表示されず、ボットのオン ターン ロジックが引き継ぎます。
- ユーザーのゲスト情報がない場合、チェックイン ダイアログを開始します。
- ゲスト情報がある場合はメイン ダイアログが優位となり、ユーザーは何度でもテーブル予約ダイアログまたは目覚ましダイアログを開始することができます。

ボットのオンターン ハンドラーのロジックを更新します。

1. ユーザーの状態を取得します。
1. アクティブなダイアログがあれば続行します。
1. このターンを完了したダイアログは、チェックイン ダイアログであると考えられます。
   1. ゲスト情報を格納します。
   1. メイン ダイアログを開始します。
1. ボットからユーザーにまだメッセージが送信されていない場合、アクティブなダイアログは実行されていません。
    1. ゲストの情報がない場合は、チェックイン ダイアログを開始します。
    1. それ以外の場合は、メイン ダイアログを開始します。
1. 状態の変化があれば保存します。

メイン ダイアログには、次のステップがあります。

1. テーブルの予約、またはモーニング コールの設定など、ゲストに希望する用件を尋ねます。
1. 対応する子ダイアログを開始するか、または "_入力が認識されません_" というメッセージを送信して、最初のステップからやり直します。
1. 子ダイアログからの戻り値を処理して、メイン ダイアログを再開します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelBot.cs** ファイルで、using ステートメントを更新します。

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

初期化コードを更新して、ボットのダイアログ セットとメイン ダイアログを定義します。

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

また、そのダイアログ セットを使用するようにボットのターン ハンドラーを更新します。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボット ファイル (**bot.js**) には、使用する SDK のクラスだけでなく、ここでコンポーネント ダイアログ用に定義したクラスもインポートする必要があります。

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

さらに、ダイアログ セットを作成して、使用することになるダイアログをすべて追加する必要があります。

メイン ダイアログのウォーターフォール ステップは、インラインで定義するのではなく、クラス内の関数として定義しています。 関数内から `this` で正しく解決できるよう、これらの関数の `bind()` を使用しています。

更新されたボットのコンストラクターは次のとおりです。

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

ボットのコンストラクターの下に、メイン ダイアログのステップを実装する次のコードを追加します。

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

また、ボットのターン ハンドラーを更新します。

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

ご覧のように、[プロンプト](bot-builder-prompts.md)をダイアログに追加する方法と同様のやり方で、コンポーネント ダイアログがボットのメイン ダイアログに追加されます。 必要な数の子ダイアログをメイン ダイアログに追加することができます。 各モジュールは、ボットがユーザーに提供できるその他の機能とサービスを追加します。

## <a name="next-steps"></a>次の手順

これで、コンポーネント ダイアログを使用する方法を学んだので、ダイアログを開始するタイミングをボットが決定できるように Language Understanding (LUIS) を使用する方法を見てみましょう。

> [!div class="nextstepaction"]
> [LUIS を使用した言語の理解](./bot-builder-howto-v4-luis.md)
