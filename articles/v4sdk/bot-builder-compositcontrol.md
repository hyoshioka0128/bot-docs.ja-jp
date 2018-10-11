---
title: 統合された一連のダイアログを作成する | Microsoft Docs
description: Bot Builder SDK for Node.js および C# でダイアログ コンテナーを使用して、ボット ロジックをモジュール化する方法について説明します。
keywords: 複合コントロール、モジュラー ボット ロジック
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b0ce3e6d11d20985e23c1bb74bd5b9dce8190d0c
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707158"
---
# <a name="create-an-integrated-set-of-dialogs"></a>統合された一連のダイアログを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ユーザーに挨拶する、夕食のテーブルを予約する、食べ物を注文する、アラームを設定する、現在の天気を表示するなど、複数のタスクを処理するホテル ボットを作成しているとします。 これらの各タスクは、1 つのダイアログ オブジェクトを使用してボット内で処理できますが、これによりボットのメイン ファイルのダイアログ コードが大きくなりすぎて見づらくなります。

この問題は、モジュール化することによって対処することができます。 モジュール化により、コードが合理化され、デバッグが容易になります。 さらに、セクションに分割することもできるので、複数のチームが同時にボットを操作できるようになります。 ダイアログ コンテナーを使用すると、会話フローをコンポーネントに分割して複数の会話フローを管理するボットを作成できます。 いくつかの基本的な会話フローを作成し、それらをダイアログ コンテナーを使用して 1 つにまとめる方法を紹介します。

この例では、チェックイン、ウェイクアップ、および予約テーブルの各モジュールを組み合わせたホテル ボットを作成していきます。

## <a name="managing-state"></a>状態の管理

複合ダイアログを使用するボットの状態管理を設定するには、多くの方法があります。 このボットでは、これを実行する方法の 1 つを説明します。

状態管理の詳細については、「[Save state using conversation and user properties (会話およびユーザー プロパティを使用した状態の保存)](bot-builder-howto-v4-state.md)」を参照してください。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

各ダイアログ ボックスは、いくつかの情報を収集して、この情報をユーザー状態に保存します。 各ダイアログのクラスを定義し、そのクラスをユーザー状態クラスのプロパティとして使用します。

```csharp
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

/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}
```

ボットのターン内で、ダイアログ セットの `CreateContext` メソッドはダイアログの状態を確立します。
このメソッドは、パラメーターとして[ターン コンテキスト](bot-builder-concept-activity-processing.md#turn-context)と状態オブジェクトを使用します。

ダイアログの場合、この状態オブジェクトは `IDictionary<string, object>` インターフェイスを実装する必要があります。 このボットはダイアログ状態を格納するためだけに会話状態を使用しているので、会話状態クラスは単純な辞書になります。

```csharp
using System.Collections.Generic;

/// <summary>
/// Conversation state information.
/// We are also using this directly for dialog state, which needs an <see cref="IDictionary{string, object}"/> object.
/// </summary>
public class ConversationInfo : Dictionary<string, object> { }
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

ユーザーの入力を管理するには、親ダイアログから `userInfo` オブジェクトをダイアログ パラメーターとして渡します。 各ダイアログ クラスでは、ダイアログがコンストラクターに組み込まれ、情報を `userInfo` に保存できます。 このダイアログでは、ユーザーが情報を入力するときに、`step.values` オブジェクトのプロパティとして定義されたローカル状態オブジェクトに書き込むことができます。 ダイアログの完了後、ローカル状態オブジェクトは破棄されます。 そのため、ローカル状態オブジェクトを親の `userInfo` に保存します。これにより、ユーザーと交わしたすべての会話を通じてユーザーに関する情報が保持されます。 

---

## <a name="define-a-modular-check-in-dialog"></a>モジュラー チェックイン ダイアログの定義

まず、簡単なチェックイン ダイアログから始めます。このダイアログでは、ユーザーに名前と滞在する部屋を尋ねます。 このタスクをモジュール化するには、`ComponentDialog` を拡張する `CheckIn` クラスを作成します。 このクラスにはルート ダイアログの名前を定義するコンストラクターがあり、このダイアログは 3 つのステップで*ウォーターフォール*として定義されています。 ダイアログ オブジェクトの署名と構築は、標準のウォーターフォールとまったく同じです。

**チェックイン ダイアログの手順**

1. ゲストの名前を尋ねます。
1. 滞在を希望する部屋を尋ねます。
1. 確認メッセージを送信し、ダイアログを完了します。

ダイアログとウォーターフォールの詳細については、[ダイアログを使用した単純な会話フローの管理](bot-builder-dialog-manage-conversation-flow.md)に関するページをご覧ください。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

`CheckIn` クラスには、チェックイン ダイアログの手順を定義して、単一のインスタンスを作成し、それを静的 `Instance` プロパティで公開するプライベート コンストラクターがあります。

このダイアログでは、手順コンテキスト `step.values` のプロパティからアクセス可能なローカル状態オブジェクトに書き込むことができます。 ダイアログが完了すると、ローカル状態オブジェクトは破棄されます。 そのため、ローカル状態オブジェクトをボットの `userState` に保存します。これにより、ユーザーと交わしたすべての会話を通じてユーザーに関する情報が保持されます。

状態管理の詳細については、「[Save state using conversation and user properties (会話およびユーザー プロパティを使用した状態の保存)](bot-builder-howto-v4-state.md)」を参照してください。 

**CheckIn.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;

namespace HotelBot
{
    public class CheckIn : DialogContainer
    {
        public const string Id = "checkIn";

        private const string GuestKey = nameof(CheckIn);

        public static CheckIn Instance { get; } = new CheckIn();

        // You can start this from the parent using the dialog's ID.
        private CheckIn() : base(Id)
        {
            // Define the conversation flow using a waterfall model.
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the guest information and prompt for the guest's name.
                    dc.ActiveDialog.State[GuestKey] = new GuestInfo();
                    await dc.Prompt("textPrompt", "What is your name?");
                },
                async (dc, args, next) =>
                {
                    // Save the name and prompt for the room number.
                    var name = args["Value"] as string;

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Name = name;

                    await dc.Prompt("numberPrompt",
                        $"Hi {name}. What room will you be staying in?");
                },
                async (dc, args, next) =>
                {
                    // Save the room number and "sign off".
                    var room = (string)args["Text"];

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Room = room;

                    await dc.Context.SendActivity("Great, enjoy your stay!");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Guest = (GuestInfo)guestInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("textPrompt", new TextPrompt());
            this.Dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**checkIn.js**
```JavaScript
const { ComponentDialog, DialogSet, TextPrompt, NumberPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckIn extends ComponentDialog {
    constructor(dialogId, userInfo) {
        // Dialog ID of 'checkIn' will start when class is called in the parent
        super(dialogId);

        // Defining the conversation flow using a waterfall model
        this.dialogs.add(new WaterfallDialog('checkIn', [
            async function (step) {
                // Create a new local guestInfo step.values object
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name 
                step.values.guestInfo.userName = step.result;
                return await step.prompt('numberPrompt', `Hi ${name}. What room will you be staying in?`);
            },
            async function (step) { 
                // Save the room number
                step.values.guestInfo.room = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay!`);

                // Save dialog's state object to the parent's state object
                const user = await userInfo.get(step.context);
                user.guestInfo = step.values.guestInfo;
                // Save changes
                await userInfo.set(step.context, user);
                return await step.endDialog();
            }
        ]));
        
        // Defining the prompt used in this conversation flow
        this.dialogs.add(new TextPrompt('textPrompt'));
        this.dialogs.add(new NumberPrompt('numberPrompt'));
    }
}
exports.CheckIn = CheckIn;
```

---

## <a name="define-modular-reserve-table-and-wake-up-dialogs"></a>モジュラーの予約テーブルとウェイクアップのダイアログの定義

コンポーネント ダイアログを使用する利点の 1 つは、ダイアログをまとめて作成できることです。 各 `DialogSet` は `dialogs` の排他的なセットを維持するので、`dialogs` の共有または相互参照は容易に実行できません。 ここでコンポーネント ダイアログの出番です。 コンポーネント ダイアログを使用して、ダイアログ全体のダイアログ フローの管理をより簡単にする複合ダイアログを作成できます。 これを説明するために、さらに 2 つのダイアログを作成しましょう。1 つはユーザーに夕食のために予約するテーブルを尋ねるダイアログ、もう 1 つはモーニング コールを作成するダイアログです。 ここでも、各ダイアログに個別のクラスを使用し、各ダイアログは `ComponentDialog` を拡張します。

**予約テーブル ダイアログの手順**

1. 予約するテーブルを尋ねます。
1. 確認メッセージを送信し、ダイアログを完了します。

**ウェイクアップ ダイアログの手順**

1. 設定するウェイクアップ時間を尋ねます。
1. 確認メッセージを送信し、ダイアログを完了します。

# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

**ReserveTable.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Linq;
using Choice = Microsoft.Bot.Builder.Prompts.Choices.Choice;
using FoundChoice = Microsoft.Bot.Builder.Prompts.Choices.FoundChoice;

namespace HotelBot
{
    public class ReserveTable : DialogContainer
    {
        public const string Id = "reserveTable";

        private const string TableKey = "Table";

        public static ReserveTable Instance { get; } = new ReserveTable();

        private ReserveTable() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the table information and prompt for the table number.
                    dc.ActiveDialog.State[TableKey] = new TableInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    var prompt = $"Welcome {guestInfo.Name}, " +
                        $"which table would you like to reserve?";
                    var choices = new string[] { "1", "2", "3", "4", "5", "6" };
                    await dc.Prompt("choicePrompt", prompt,
                        new ChoicePromptOptions
                        {
                            Choices = choices.Select(s => new Choice { Value = s }).ToList()
                        });
                },
                async (dc, args, next) =>
                {
                    // Save the table number and "sign off".
                    var table = (args["Value"] as FoundChoice).Value;

                    var tableInfo = dc.ActiveDialog.State[TableKey];
                    ((TableInfo)tableInfo).Number = table;

                    await dc.Context.SendActivity(
                        $"Sounds great; we will reserve table number {table} for you.");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Table = (TableInfo)tableInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("choicePrompt", new ChoicePrompt(Culture.English));
        }
    }
}
```

**WakeUp.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
using System.Linq;
using DateTimeResolution = Microsoft.Bot.Builder.Prompts.DateTimeResult.DateTimeResolution;

namespace HotelBot
{
    public class WakeUp : DialogContainer
    {
        public const string Id = "wakeUp";

        private const string WakeUpKey = "WakeUp";

        public static WakeUp Instance { get; } = new WakeUp();

        private WakeUp() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the wake up call information and prompt for the alarm time.
                   dc.ActiveDialog.State[WakeUpKey] = new WakeUpInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    await dc.Prompt("datePrompt", $"Hi {guestInfo.Name}, " +
                        $"what time would you like your alarm set for?");
                },
                async (dc, args, next) =>
                {
                    // Save the alarm time and "sign off".
                    var resolution = (List<DateTimeResolution>)args["Resolution"];

                    var wakeUp = dc.ActiveDialog.State[WakeUpKey];
                    ((WakeUpInfo)wakeUp).Time = resolution?.FirstOrDefault()?.Value;

                    var userState = UserState<UserInfo>.Get(dc.Context);
                    await dc.Context.SendActivity(
                        $"Your alarm is set to {((WakeUpInfo)wakeUp).Time}" +
                        $" for room {userState.Guest.Room}.");

                    // Save dialog state to user state and end the dialog.
                    userState.WakeUp = (WakeUpInfo)wakeUp;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("datePrompt", new DateTimePrompt(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

**reserveTable.js**
```JavaScript
const { ComponentDialog, DialogSet, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTable extends ComponentDialog {
    constructor(dialogId, userInfo) {
        super(dialogId); 

        // Defining the conversation flow using a waterfall model
        this.dialogs.add(new WaterfallDialog('reserve_table', [
            async function (step) {
                // Get the user state from context
                const user = await userInfo.get(step.context);

                // Create a new local reserveTable state object
                step.values.reserveTable = {};

                const prompt = `Welcome ${ user.guestInfo.userName }, which table would you like to reserve?`;
                const choices = ['1', '2', '3', '4', '5', '6'];
                return await step.prompt('choicePrompt', prompt, choices);
            },
            async function(step) {
                const choice = step.result;
                
                // Save the table number
                step.values.reserveTable.tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve table number ${ choice.value } for you.`);
                
                // Get the user state from context
                const user = await userInfo.get(dc.context);
                // Persist dialog's state object to the parent's state object
                user.reserveTable = step.values.reserveTable;
               
                // Save changes
                await userInfo.set(step.context, user);

                // End the dialog
                return await step.endDialog();
            }
        ]));

        // Defining the prompt used in this conversation flow
        this.dialogs.add(new ChoicePrompt('choicePrompt'));
    }
}
exports.ReserveTable = ReserveTable;
```

**wakeUp.js**
```JavaScript
const { ComponentDialog, DialogSet, DatetimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class WakeUp extends ComponentDialog {
    constructor(userInfo) {
        // Dialog ID of 'wakeup' will start when class is called in the parent
        super('wakeUp');

        this.dialogs.add(new WaterfallDialog('wakeUp', [
            async function (step) {
                // Get the user state from context
                const user = await userInfo.get(step.context); 

                // Create a new local reserveTable state object
                step.values.wakeUp = {};  
                             
                return await step.prompt('datePrompt', `Hello, ${ user.guestInfo.userName }. What time would you like your alarm to be set?`);
            },
            async function (step) {
                const time = step.result;
            
                // Get the user state from context
                const user = await userInfo.get(step.context);

                // Save the time
                step.values.wakeUp.time = time[0].value

                await step.context.sendActivity(`Your alarm is set to ${ time[0].value } for room ${ user.guestInfo.room }`);
                
                // Save dialog's state object to the parent's state object
                user.wakeUp = step.values.wakeUp;

                // Save changes
                await userInfo.step(step.context, user);

                // End the dialog
                return await step.endDialog();
            }]));

        // Defining the prompt used in this conversation flow
        this.dialogs.add(new DatetimePrompt('datePrompt'));
    }
}
exports.WakeUp = WakeUp;
```

---

## <a name="add-modular-dialogs-to-a-bot"></a>モジュラー ダイアログのボットへの追加

メイン ボット ファイルは、これらの 3 つのモジュール化されたダイアログを 1 つのボットに結びつけます。

- 3 つのダイアログすべてがボットのメイン ダイアログ セットに追加されます。
- 予約テーブルとウェイクアップの各ダイアログは、メイン ダイアログの会話フローに組み込まれます。
- 新しい会話の開始時には、アクティブなダイアログは表示されず、ボットのオン ターン ロジックが引き継ぎます。

**ボットのオンターン ハンドラー**

ボット ターンがアクティブなダイアログ外にある場合は常にユーザの状態をチェックします。
1. 既にゲストの情報がある場合は、メイン ダイアログが起動します。
1. それ以外の場合は、メイン ダイアログのチェックインの子ダイアログを開始します。

**メイン ダイアログの手順**

1. テーブルの予約、またはモーニング コールの設定など、ゲストに希望する用件を尋ねます。
1. 対応する子ダイアログを開始するか、または _"入力が認識されません"_ というメッセージを送信し、次の手順に進みます。
1. スキップして、このダイアログの先頭に戻ります。


# <a name="ctabcsharp"></a>[C# を選択した場合](#tab/csharp)

ダイアログ フローは、ダイアログ コンテキストの `Continue` メソッドを使用して更新されます。 このメソッドは、ダイアログ スタックでウォーターフォールの次の手順を実行します。

**HotelBot.cs**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

namespace HotelBot
{
    public class HotelBot : IBot
    {
        private const string MainMenuId = "mainMenu";

        private DialogSet _dialogs { get; } = ComposeMainDialog();

        /// <summary>
        /// Every Conversation turn for our bot calls this method. 
        /// </summary>
        /// <param name="context">The current turn context.</param>        
        public async Task OnTurn(ITurnContext context)
        {
            if (context.Activity.Type is ActivityTypes.Message)
            {
                // Get the user and conversation state from the turn context.
                var userInfo = UserState<UserInfo>.Get(context);
                var conversationInfo = ConversationState<ConversationInfo>.Get(context);

                // Establish dialog state from the conversation state.
                var dc = _dialogs.CreateContext(context, conversationInfo);

                // Continue any current dialog.
                await dc.Continue();

                // Every turn sends a response, so if no response was sent,
                // then there no dialog is currently active.
                if (!context.Responded)
                {
                    // If we don't yet have the guest's info, start the check-in dialog.
                    if (string.IsNullOrEmpty(userInfo?.Guest?.Name))
                    {
                        await dc.Begin(CheckIn.Id);
                    }
                    // Otherwise, start our bot's main dialog.
                    else
                    {
                        await dc.Begin(MainMenuId);
                    }
                }
            }
        }

        /// <summary>
        /// Composes a main dialog for our bot.
        /// </summary>
        /// <returns>A new main dialog.</returns>
        private static DialogSet ComposeMainDialog()
        {
            var dialogs = new DialogSet();

            dialogs.Add(MainMenuId, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    var menu = new List<string> { "Reserve Table", "Wake Up" };
                    await dc.Context.SendActivity(MessageFactory.SuggestedActions(menu, "How can I help you?"));
                },
                async (dc, args, next) =>
                {
                    // Decide which dialog to start.
                    var result = (args["Activity"] as Activity)?.Text?.Trim().ToLowerInvariant();
                    switch (result)
                    {
                        case "reserve table":
                            await dc.Begin(ReserveTable.Id);
                            break;
                        case "wake up":
                            await dc.Begin(WakeUp.Id);
                            break;
                        default:
                            await dc.Context.SendActivity("Sorry, I don't understand that command. Please choose an option from the list below.");
                            await next();
                            break;
                    }
                },
                async (dc, args, next) =>
                {
                    // Show the main menu again.
                    await dc.Replace(MainMenuId);
                }
            });

            // Add our child dialogs.
            dialogs.Add(CheckIn.Id, CheckIn.Instance);
            dialogs.Add(ReserveTable.Id, ReserveTable.Instance);
            dialogs.Add(WakeUp.Id, WakeUp.Instance);

            return dialogs;
        }
    }
}
```

最後に、`StartUp` クラスの `ConfigureServices` メソッドを更新し、ボットを接続して状態ミドルウェアを追加する必要があります。

**Startup.cs**
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(HotelBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // Add state management middleware for conversation and user state.
        var path = Path.Combine(Path.GetTempPath(), nameof(HotelBot));
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        IStorage storage = new FileStorage(path);

        options.Middleware.Add(new ConversationState<ConversationInfo>(storage));
        options.Middleware.Add(new UserState<UserInfo>(storage));
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript を選択した場合](#tab/javascript)

ダイアログ フローは、ダイアログ コンテキストの `continue` メソッドを使用して更新されます。 このメソッドは、ダイアログ スタックでウォーターフォールの次の手順を実行します。

**app.js**
```JavaScript
const { BotFrameworkAdapter, ConversationState, UserState, MemoryStorage, MessageFactory } = require("botbuilder");
const { DialogSet } = require("botbuilder-dialogs");
const restify = require("restify");
var azure = require('botbuilder-azure'); 

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Memory Storage
const storage = new MemoryStorage();
// ConversationState lasts for the entirety of a conversation then gets disposed of
const conversationState = new ConversationState(storage);

// UserState persists information about the user across all of the conversations you have with that user
const userState  = new UserState(storage);

// Create a place in the conversation state to store dialog state.
const dialogState = conversationState.createProperty('dialogState');

// Create a place in the user storage to store a user info.
const userInfo = userState.createProperty('userInfo');

// Create a dialog set and pass in our dialogState property.
const dialogs = new DialogSet(dialogState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        const dc = dialogs.createContext(context);
 
        // Continue the current dialog if one is currently active
        await dc.continueDialog();

        // Default action
        if (!context.responded && isMessage) {

            // Getting the user info from the state
            const user = await userInfo.get(dc.context); 
            // If guest info is undefined prompt the user to check in
            if (!user.guestInfo) {
                await dc.beginDialog('checkInPrompt');
            } else {
                await dc.beginDialog('mainMenu'); 
            }           
        }
        
        // End by saving any changes to the state that have occured during this turn.
        await conversationState.saveChanges(dc.context);
        await userState.saveChanges(dc.context);
    });
});

dialogs.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        const menu = ["Reserve Table", "Wake Up"];
        await step.context.sendActivity(MessageFactory.suggestedActions(menu));
        return await step.next();
    },
    async function (step) {
        // Decide which module to start
        switch (step.result) {
            case "Reserve Table":
                return await step.beginDialog('reservePrompt');
                break;
            case "Wake Up":
                return await step.beginDialog('wakeUpPrompt');
                break;
            default:
                await step.context.sendActivity("Sorry, i don't understand that command. Please choose an option from the list below.");
                return await step.next();
                break;            
        }
    },
    async function (step){
        return await step.replaceDialog('mainMenu'); // Show the menu again
    }

]));

// Importing the dialogs 
const checkIn = require("./checkIn");
dialogs.add(new checkIn.CheckIn('checkInPrompt', userState));

const reserve_table = require("./reserveTable");
dialogs.add(new reserve_table.ReserveTable('reservePrompt', userState));

const wake_up = require("./wake_up");
dialogs.add(new wake_up.WakeUp('wakeUpPrompt', userState));
```

---
<!-- TODO: These dialogs are not fully modularized, as there are cross dependencies:
    - Importantly, the dialogs need to know details about the bot's user state.
-->

ご覧のように、[プロンプト](bot-builder-prompts.md)をダイアログに追加する方法と同様の方法で、モジュラー ダイアログがボットのメイン ダイアログに追加されます。 必要な数の子ダイアログをメイン ダイアログに追加することができます。 各モジュールは、ボットがユーザーに提供できるその他の機能とサービスを追加します。

## <a name="next-steps"></a>次の手順

これで、ダイアログを使用してボット ロジックをモジュール化する方法を学んだので、ダイアログを開始するタイミングをボットが決定できるように Language Understanding (LUIS) を使用する方法を見てみましょう。

> [!div class="nextstepaction"]
> [LUIS を使用した言語の理解](./bot-builder-howto-v4-luis.md)
