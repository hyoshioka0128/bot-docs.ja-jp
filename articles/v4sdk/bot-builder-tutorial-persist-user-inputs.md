---
title: ユーザー データを保持する | Microsoft Docs
description: Bot Builder SDK で、ユーザー状態データをストレージに保存する方法について説明します。
keywords: ユーザー データを保持する、ストレージ、会話データ
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 61e86ce9536bc5d77dc7bd411054b2f65bce8dd9
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326559"
---
# <a name="persist-user-data"></a>ユーザー データを保持する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットがユーザーに入力を要求した場合、それは一部の情報を何らかの形式のストレージに保持する機会になります。 Bot Builder SDK を使用すると、*メモリ内ストレージ*またはデータベース ストレージ (*CosmosDB*など) を使用してユーザー入力を格納することができます。 ボットのテスト中やプロトタイプ中には、主にローカル ストレージ類が使用されます。 ただし、データベース ストレージなどの永続的なストレージ類は運用環境のボットに最適です。

このトピックでは、ストレージ オブジェクトを定義し、保持できるようにユーザー入力をストレージ オブジェクトに保存する方法を示します。 まだ行っていない場合は、ダイアログを使用してユーザーに名前を要求します。 使用することを選択したストレージの種類には関係なく、それを接続してデータを保存するためのプロセスは同じです。 このトピック内のコードは、データを保持するストレージとして `CosmosDB` を使用します。

## <a name="prerequisites"></a>前提条件

使用する開発環境によって、特定のリソースが必要です。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [Visual Studio 2015 以降をインストールする](https://www.visualstudio.com/downloads/)。
* [BotBuilder V4 テンプレートをインストールする](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [Visual Studio Code をインストールする](https://www.visualstudio.com/downloads/)。
* [Node.js v8.5 以上をインストールする](https://nodejs.org/en/)。
* [Yeoman をインストールする](http://yeoman.io/)。
* Node.js v4 Botbuilder テンプレート ジェネレーターをインストールする。

    ```shell
    npm install generator-botbuilder
    ```

---

このチュートリアルは、次のパッケージを使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

基本的な EchoBot テンプレートから始めます。 手順については、[.NET のクイック スタート](~/dotnet/bot-builder-dotnet-quickstart.md)をご覧ください。

NuGet パケット マネージャーから次の追加パッケージをインストールします。

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

基本的な EchoBot テンプレートから始めます。 手順については、[JavaScript のクイック スタート](~/javascript/bot-builder-javascript-quickstart.md)を参照してください。

次の追加 npm パッケージをインストールします。

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

このチュートリアルで作成するボットをテストするには、[Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) をインストールする必要があります。

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>CosmosDB サービスを作成してアプリケーションの設定を更新する

CosmosDB サービスとデータベースを設定するには、[CosmosDB の使用](bot-builder-howto-v4-storage.md#using-cosmos-db)に関する手順に従います。 手順は次のとおりです。

1. 新しいブラウザー ウィンドウで、<a href="http://portal.azure.com/" target="_blank">Azure Portal</a> にサインインします。
1. **[リソースの作成]、[データベース]、[Azure Cosmos DB]** の順にクリックします。
1. **[新規アカウント] ページ**の **[ID]** フィールドに一意の名前を指定します。 **[API]** として **[SQL]** を選択し、**[サブスクリプション]**、**[場所]**、および **[リソース グループ]** に情報を指定します。
1. **Create** をクリックしてください。

その後、そのサービスにこのボットで使用するコレクションを追加します。

コレクションへの追加に使用したデータベース ID とコレクション ID、およびそのコレクションのキー設定から URI とプライマリ キーを記録します。これらはボットをサービスに接続するのに必要です。

### <a name="update-your-application-settings"></a>アプリケーションの設定を更新する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**appsettings.json** を更新して CosmosDB の接続情報を追加します。

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

プロジェクト フォルダー内 で **.env** ファイルを探し、Cosmos 固有のデータが指定された次のエントリーを追加します。

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

その後、メインのボットの**index.js** ファイルで、`MemoryStorage` の代わりに `CosmosDbStorage` を使用するよう `storage` を置き換えます。 実行時に、環境変数が取り込まれ、これらのフィールドに入力されます。

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>ストレージ、状態マネージャー、状態プロパティ アクセサー オブジェクトを作成する

ボットは状態管理およびストレージ オブジェクトを使用して状態を管理維持します。 このマネージャーは、基になるストレージの種類には関係なく、状態プロパティ アクセサーを使用して状態プロパティにアクセスできる抽象化レイヤーを提供します。 状態マネージャーを使用してストレージにデータを書き込みます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>ユーザー データ用のクラスを定義する

ファイル **CounterState.cs** の名前を **UserData.cs** に変更し、**CounterState** クラスの名前を **UserData** に変更します。

このクラスを更新して、収集したデータが保持されるようにします。

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>状態オブジェクトと状態プロパティ アクセサー オブジェクトのクラスを定義する

ファイル **EchoBotAccessors.cs** の名前を **BotAccessors.cs** に変更し、**EchoBotAccessors** クラスの名前を **BotAccessors** に変更します。

このクラスを更新して、状態オブジェクトとボットで必要とされる状態プロパティ アクセサーを格納します。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>ボットの起動コードを更新する

**Startup.cs** ファイルで、使用するステートメントを更新します。

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

`ConfigureServices` メソッドで、バックアップするストレージ オブジェクトを作成した場所からボットの追加の呼び出しを更新し、その後ボットのアクセサー オブジェクトを登録します。

ダイアログの状態を追跡するには、`DialogState` オブジェクトの会話状態が必要です。 ダイアログの状態プロパティ アクセサーのシングルトンと、ボットが使用するダイアログ セットを登録しています。 ボットがそのユーザー状態に対して独自の状態プロパティ アクセサーを作成します。

`BotAccessors` アクセサーは、対象のボットの複数の状態オブジェクトのストレージを管理する効率的な方法です。

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>サーバー コードを更新する

プロジェクトの **index.js** ファイルで、次の require ステートメントを更新します。

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

このチュートリアルでは、`UserState` を使用してデータを格納します。 新しい `userState` オブジェクトを作成し、このコード行を更新して 2 番目のパラメーターを `MainDialog` クラスに渡す必要があります。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

一般的なエラーが発生した場合は、会話状態とユーザー状態の両方をクリアします。

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

さらに、ボット オブジェクトを呼び出すように HTTP サーバー ループを更新します。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>ボットのロジックを更新する

`MyBot` クラスで、ボットが動作するための必須のライブラリを要求します。 このチュートリアルでは、**ダイアログ** ライブラリを使用します。

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

`MyBot` クラスのコンストラクターを更新して 2 番目のパラメーター `userState` を受け入れます。 また、コンストラクターを更新して、このチュートリアルに必要な状態、ダイアログ、プロンプトを定義します。 このケースでは、2 ステップのウォーターフォールを定義します。"_ステップ 1_" ではユーザーの名前を要求し、"_ステップ 2_" ではユーザーの応答を返します。 その情報を保持するかどうかはボットのメイン ロジックによって変わります。

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

ユーザー データを保存する場合は、いくつかの選択肢があります。 SDK には異なるスコープを持つ状態オブジェクトがいくつか用意されており、そこから選択できます。 ここでは、会話状態を使用してダイアログ状態オブジェクトを管理し、ユーザー状態を使用してユーザー データを管理します。

ストレージや状態全般について詳しくは、[ストレージ](bot-builder-howto-v4-storage.md)および[会話状態とユーザー状態を管理する方法](bot-builder-howto-v4-state.md)に関する記事をご覧ください。

## <a name="create-a-greeting-dialog"></a>あいさつのダイアログを作成する

ダイアログを使用してユーザーの名前を収集します。 このシナリオを簡潔にするために、ダイアログがユーザーの名前を返し、ボットがユーザー データ オブジェクトと状態を管理します。

**GreetingsDialog** クラスを作成し、次のコードを追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`MainDialog` のコンストラクター内にダイアログが作成される、前述のセクションをご覧ください。

---

ダイアログについて詳しくは、[入力を求める方法](bot-builder-prompts.md)および[単純な会話フローを管理する方法](bot-builder-dialog-manage-conversation-flow.md)に関する記事をご覧ください。

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>ダイアログとユーザー状態を使用してボットを更新する

ボットの構築方法とユーザーの入力の管理方法については別途説明します。

### <a name="add-the-dialog-and-a-user-state-accessor"></a>ダイアログとユーザー状態アクセサーを追加する

ユーザー データのダイアログ インスタンスと状態プロパティ アクセサーをトラックする必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットを初期化するコードを追加します。

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`MainDialog` のコンストラクター内でこれらの状態アクセサーが定義される、前述のセクションをご覧ください。

---

### <a name="update-the-turn-handler"></a>ターン ハンドラーを更新する

ターン ハンドラーは初めて会話に参加するときにユーザーにあいさつし、ボットにメッセージを送信するときはいつでも応答します。 いずれかの時点でボットがまだユーザーの名前を知らない場合、あいさつダイアログを開始して名前を要求します。 ダイアログが完了すると、状態プロパティ アクセサーにより生成された状態オブジェクトを使用して名前をユーザー状態に保存します。 ターンが終了すると、アクセサーと状態マネージャーがオブジェクトに対する変更をストレージに書き出します。

また、_ユーザー データの削除_アクティビティに対するサポートを追加します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットの `OnTurnAsync` メソッドを更新します。

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットの `onTurn` ハンドラーを更新します。

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio でのボットの開始

アプリケーションをビルドして実行します。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。
2. Visual Studio ソリューションを作成したディレクトリにある .bot ファイルを選択します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットはメッセージで応答します。
![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [会話とユーザー状態の管理](bot-builder-howto-v4-state.md)
