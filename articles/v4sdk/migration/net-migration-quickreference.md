---
title: .NET v3 から v4 への移行のクイック リファレンス | Microsoft Docs
description: .NET Bot Framework SDK の v3 と v4 の主な相違点を概説します。
keywords: .Net, ボットの移行, ダイアログ, v3 ボット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ff072cd3a16c3a58099cf91c1de962837994075b
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2019
ms.locfileid: "73441495"
---
# <a name="net-migration-quick-reference"></a>.NET 移行クイック リファレンス

BotBuilder .NET SDK v4 では、ボットの作成方法に影響する基本的な変更がいくつか行われています。 このガイドでは、v3 と v4 の各 SDK で作業を行う場合の一般的な違いを強調するためにクイック リファレンスを提供することを目的としています。

- ボットとチャネルの間で情報を渡す方法が変更されました。 v3 では、_Conversation_ オブジェクトと _SendAsync_ メソッドを使用してメッセージを処理し、Autofac を広範囲にわたって使用してさまざまな依存関係の読み込んでいました。 v4 では、_Adapter_ オブジェクトと _TurnContext_ オブジェクトを使用してメッセージを処理し、任意の依存関係の挿入ライブラリを使用できます。

- また、ダイアログとボット インスタンスがさらに分離されました。 v3 では、ダイアログがコア SDK に組み込まれ、スタックが内部的に処理され、子ダイアログは _Call_ メソッドと _Forward_ メソッドを使用して読み込まれていました。 v4 では、ダイアログを引数としてボット インスタンスに渡すようになったため、構成における柔軟性が向上しています。子ダイアログは、_BeginDialogAsync_ メソッドと _ReplaceDialogAsync_ メソッドを使用して読み込みます。

- さらに、v4 では、`ActivityHandler` クラスを提供しています。これを利用して、_メッセージ_ アクティビティ、_会話の更新_アクティビティ、_イベント_ アクティビティなど、さまざまな種類のアクティビティの処理を自動化することができます。

これらの機能強化の結果、特に、ボット オブジェクトの作成、ダイアログの定義、イベント処理ロジックのコーディング関連で、.NET でボットを開発するための構文が変更されています。

このトピックの残りの部分では、.NET Bot Framework SDK v3 の構成要素と、それに対応する v4 の構成要素を比較します。

## <a name="to-process-incoming-messages"></a>受信メッセージを処理するには

### <a name="v3"></a>v3

```cs
[BotAuthentication]
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
        if (activity.GetActivityType() == ActivityTypes.Message)
        {
            await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }
}
```

### <a name="v4"></a>v4

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

## <a name="to-send-a-message-to-a-user"></a>ユーザーにメッセージを送信するには

### <a name="v3"></a>v3

```cs
await context.PostAsync("Hello and welcome to the help desk bot.");
```

### <a name="v4"></a>v4

```cs
await turnContext.SendActivityAsync("Hello and welcome to the help desk bot.");
```

## <a name="to-load-a-root-dialog"></a>ルート ダイアログを読み込むには

### <a name="v3"></a>v3

```cs
await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
```

### <a name="v4"></a>v4

```cs
// Create a DialogExtensions class with a Run method.
public static class DialogExtensions
{
    public static async Task Run(
        this Dialog dialog,
        ITurnContext turnContext,
        IStatePropertyAccessor<DialogState> accessor,
        CancellationToken cancellationToken)
    {
        var dialogSet = new DialogSet(accessor);
        dialogSet.Add(dialog);

        var dialogContext = await dialogSet.CreateContextAsync(turnContext, cancellationToken);

        var results = await dialogContext.ContinueDialogAsync(cancellationToken);
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync(dialog.Id, null, cancellationToken);
        }
    }
}

// Call it from the ActivityHandler's OnMessageActivityAsync override
protected override async Task OnMessageActivityAsync(
    ITurnContext<IMessageActivity> turnContext,
    CancellationToken cancellationToken)
{
    // Run the Dialog with the new message Activity.
    await Dialog.Run(
        turnContext,
        ConversationState.CreateProperty<DialogState>("DialogState"),
        cancellationToken);
}
```

## <a name="to-start-a-child-dialog"></a>子ダイアログを開始するには

### <a name="v3"></a>v3

```cs
context.Call(new NextDialog(), this.ResumeAfterNextDialog);
```

or

```cs
await context.Forward(new NextDialog(), this.ResumeAfterNextDialog, message);
```

### <a name="v4"></a>v4

```cs
dialogContext.BeginDialogAsync("<child-dialog-id>", options);
```

or

```cs
dialogContext.ReplaceDialogAsync("<child-dialog-id>", options);
```

## <a name="to-end-a-dialog"></a>ダイアログを終了するには

### <a name="v3"></a>v3

```cs
context.Done(ReturnValue);
```

### <a name="v4"></a>v4

```cs
await context.EndDialogAsync(ReturnValue);
```

## <a name="to-prompt-a-user-for-input"></a>ユーザーに入力を要求するには

### <a name="v3"></a>v3

```cs
PromptDialog.Choice(
    context,
    this.OnOptionSelected,
    Options, PromptMessage,
    ErrorMessage,
    3,
    PromptStyle.PerLine);
```

### <a name="v4"></a>v4

```cs
// In the dialog's constructor, register the prompt, and waterfall steps.
AddDialog(new TextPrompt(nameof(TextPrompt)));
AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
{
    FirstStepAsync,
    SecondStepAsync,
}));

// The initial child Dialog to run.
InitialDialogId = nameof(WaterfallDialog);

// ...

// In the first step, invoke the prompt.
private async Task<DialogTurnResult> FirstStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    return await stepContext.PromptAsync(
        nameof(TextPrompt),
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your destination.") },
        cancellationToken);
}

// In the second step, retrieve the Result from the stepContext.
private async Task<DialogTurnResult> SecondStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    var destination = (string)stepContext.Result;
}
```

## <a name="to-save-information-to-dialog-state"></a>ダイアログの状態の情報を保存するには

### <a name="v3"></a>v3

v3 では、すべてのダイアログとそれらのフィールドが自動シリアル化されていました。

### <a name="v4"></a>v4

```cs
// StepContext values are auto-serialized in V4, and scoped to the dialog.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>状態の変更を永続化レイヤーに書き込むには

### <a name="v3"></a>v3

状態データは、ターンの終了時に既定で自動保存されます。

### <a name="v4"></a>v4

```cs
// You now must explicitly save state changes before the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>状態ストレージを作成して登録するには

### <a name="v3"></a>v3

```cs
// Autofac was used internally by the sdk, and state was automatic.
Conversation.UpdateContainer(
builder =>
{
    builder.RegisterModule(new AzureModule(Assembly.GetExecutingAssembly()));
    var store = new InMemoryDataStore();
    builder.Register(c => store)
        .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
        .AsSelf()
        .SingleInstance();
});
```

### <a name="v4"></a>v4

```cs
// Create the storage we'll be using for User and Conversation state.
// In-memory storage is great for testing purposes.
services.AddSingleton<IStorage, MemoryStorage>();

// Create the user state (used in this bot's Dialog implementation).
services.AddSingleton<UserState>();

// Create the conversation state (used by the Dialog system itself).
services.AddSingleton<ConversationState>();

// The dialog that will be run by the bot.
services.AddSingleton<MainDialog>();

// Create the bot as a transient. In this case the ASP.NET controller is expecting an IBot.
services.AddTransient<IBot, DialogBot>();

// In the bot's ActivityHandler implementation, call SaveChangesAsync after the OnTurnAsync completes.
public class DialogBot : ActivityHandler
{
    protected readonly Dialog Dialog;
    protected readonly BotState ConversationState;
    protected readonly BotState UserState;

    public DialogBot(ConversationState conversationState, UserState userState, Dialog dialog)
    {
        ConversationState = conversationState;
        UserState = userState;
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        // Save any state changes that might have ocurred during the turn.
        await ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        // Run the dialog, passing in the message activity for this turn.
        await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>("DialogState"), cancellationToken);
    }
}
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>ダイアログからスローされたエラーをキャッチするには

### <a name="v3"></a>v3

```cs
// Create a custom IPostToBot implementation to catch exceptions.
public sealed class CustomPostUnhandledExceptionToUser : IPostToBot
{
    private readonly IPostToBot inner;
    private readonly IBotToUser botToUser;
    private readonly ResourceManager resources;
    private readonly System.Diagnostics.TraceListener trace;

    public CustomPostUnhandledExceptionToUser(IPostToBot inner, IBotToUser botToUser, ResourceManager resources, System.Diagnostics.TraceListener trace)
    {
        SetField.NotNull(out this.inner, nameof(inner), inner);
        SetField.NotNull(out this.botToUser, nameof(botToUser), botToUser);
        SetField.NotNull(out this.resources, nameof(resources), resources);
        SetField.NotNull(out this.trace, nameof(trace), trace);
    }

    async Task IPostToBot.PostAsync(IActivity activity, CancellationToken token)
    {
        try
        {
            await this.inner.PostAsync(activity, token);
        }
        catch (Exception ex)
        {
            try
            {
                // Log exception and send custom error message here.
                await this.botToUser.PostAsync("custom error message");
            }
            catch (Exception inner)
            {
                this.trace.WriteLine(inner);
            }

            throw;
        }
    }
}

// Register this using AutoFac, replacing the default PostUnhandledExceptionToUser.
builder
  .RegisterType<CustomPostUnhandledExceptionToUser>()
  .Keyed<IPostToBot>(typeof(PostUnhandledExceptionToUser));
```

### <a name="v4"></a>v4

```cs
// Provide an error handler in your implementation of the BotFrameworkHttpAdapter.
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(
        ICredentialProvider credentialProvider,
        ILogger<BotFrameworkHttpAdapter> logger,
        ConversationState conversationState = null)
        : base(credentialProvider)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError($"Exception caught : {exception.Message}");

            // Send a catch-all apology to the user.
            await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

            if (conversationState != null)
            {
                try
                {
                    // Delete the conversation state for the current conversation, to prevent the
                    // bot from getting stuck in a error-loop caused by being in a bad state.
                    // Conversation state is similar to "cookie-state" in a web page.
                    await conversationState.DeleteAsync(turnContext);
                }
                catch (Exception e)
                {
                    logger.LogError(
                        $"Exception caught on attempting to Delete ConversationState : {e.Message}");
                }
            }
        };
    }
}
```

## <a name="to-process-different-activity-types"></a>さまざまなアクティビティの種類を処理するには

### <a name="v3"></a>v3

```cs
// Within your MessageController, check the message type.
string messageType = activity.GetActivityType();
if (messageType == ActivityTypes.Message)
{
    await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
}
else if (messageType == ActivityTypes.DeleteUserData)
{
}
else if (messageType == ActivityTypes.ConversationUpdate)
{
}
else if (messageType == ActivityTypes.ContactRelationUpdate)
{
}
else if (messageType == ActivityTypes.Typing)
{
}
```

### <a name="v4"></a>v4

```cs
// In the bot's ActivityHandler implementation, override relevant methods.

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle message activities here.
}

protected override Task OnConversationUpdateActivityAsync(ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle conversation update activities in general here.
}

protected override Task OnEventActivityAsync(ITurnContext<IEventActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle event activities in general here.
}
```

## <a name="to-log-all-activities"></a>すべてのアクティビティをログに記録するには

### <a name="v3"></a>v3

[IActivityLogger](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.history.iactivitylogger) が使用されました。

```csharp
builder.RegisterType<ActivityLoggerImplementation>().AsImplementedInterfaces().InstancePerDependency(); 

public class ActivityLoggerImplementation : IActivityLogger
{
    async Task IActivityLogger.LogAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

### <a name="v4"></a>v4

[ITranscriptLogger](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.itranscriptlogger) を使用します。

```csharp
var transcriptMiddleware = new TranscriptLoggerMiddleware(new TranscriptLoggerImplementation(Configuration.GetSection("StorageConnectionString").Value));
adapter.Use(transcriptMiddleware);

public class TranscriptLoggerImplementation : ITranscriptLogger
{
    async Task ITranscriptLogger.LogActivityAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

## <a name="to-add-bot-state-storage"></a>ボット状態ストレージを追加するには

"_ユーザー データ_"、"_会話データ_"、および "_個人的な会話データ_" を格納するためのインターフェイスが変更されました。

### <a name="v3"></a>v3

状態は、`IBotDataStore` 実装を使用して保持され、Autofac を使用して SDK のダイアログ状態システムにそれを挿入します。  Microsoft は [Microsoft.Bot.Builder.Azure](https://github.com/Microsoft/BotBuilder-Azure/) で `MemoryStorage`、`DocumentDbBotDataStore`、`TableBotDataStore`、および `SqlBotDataStore` クラスを提供しています。

データを保持するために [IBotDataStore<BotData>](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdatastore-1?view=botbuilder-dotnet-3.0) が使用されました。

```csharp
Task<bool> FlushAsync(IAddress key, CancellationToken cancellationToken);
Task<T> LoadAsync(IAddress key, BotStoreType botStoreType, CancellationToken cancellationToken);
Task SaveAsync(IAddress key, BotStoreType botStoreType, T data, CancellationToken cancellationToken);
```

```csharp
var dbPath = ConfigurationManager.AppSettings["DocDbPath"];
var dbKey = ConfigurationManager.AppSettings["DocDbKey"];
var docDbUri = new Uri(dbPath);
var storage = new DocumentDbBotDataStore(docDbUri, dbKey);
builder.Register(c => storage)
                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                .AsSelf()
                .SingleInstance();
```

### <a name="v4"></a>v4

ストレージ層では `IStorage` インターフェイスが使用されます。お使いのボットの各状態管理オブジェクトを作成するときに、ストレージ層オブジェクトを指定します (`UserState`、`ConversationState`、`PrivateConversationState` など)。 状態管理オブジェクトは、基になるストレージ層にキーを提供し、またプロパティ マネージャーとしても機能します。 たとえば、`IPropertyManager.CreateProperty<T>(string name)` を使用して状態プロパティ アクセサーを作成します。  これらのプロパティ アクセサーは、ボットの基になるストレージに対して値を取得または格納するために使用されます。

[IStorage](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.istorage?view=botbuilder-dotnet-stable) を使用してデータを保持します。

```csharp
Task DeleteAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task<IDictionary<string, object>> ReadAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task WriteAsync(IDictionary<string, object> changes, CancellationToken cancellationToken = default(CancellationToken));
```

```csharp
var storageOptions = new CosmosDbPartitionedStorageOptions()
{
    AuthKey = configuration["cosmosKey"],
    ContainerId = configuration["cosmosContainer"],
    CosmosDbEndpoint = configuration["cosmosPath"],
    DatabaseId = configuration["cosmosDatabase"]
};

IStorage dataStore = new CosmosDbPartitionedStorage(storageOptions);
var conversationState = new ConversationState(dataStore);
services.AddSingleton(conversationState);

```

> [!NOTE]
> `CosmosDbPartitionedStorage` を使用する場合は、データベースを作成し、上記のように Cosmos DB エンドポイント、承認キー、およびデータベース ID を提供する必要があります。 単純にコンテナーの ID を特定してください。ボットが作成し、ボットの状態を格納するために正しく構成されていることを確認します。 コンテナーを自分で作成する場合は、パーティション キーが **/id** に設定されていることを確認し、`CosmosDbPartitionedStorageOptions.ContainerId` プロパティを設定します。

## <a name="to-use-form-flow"></a>フォーム フローを使用するには

### <a name="v3"></a>v3

[Microsoft.Bot.Builder.FormFlow](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.formflow?view=botbuilder-dotnet-3.0) がコア Bot Builder SDK に含まれました。

### <a name="v4"></a>v4

[Bot.Builder.Community.Dialogs.FormFlow](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.FormFlow/) は Bot Builder Community ライブラリになりました。  ソースはコミュニティの[リポジトリ](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.FormFlow)で入手できます。

## <a name="to-use-luisdialog"></a>LuisDialog を使用するには

### <a name="v3"></a>v3

[Microsoft.Bot.Builder.Dialogs.LuisDialog](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1?view=botbuilder-dotnet-3.0) がコア Bot Builder SDK に含まれました。

### <a name="v4"></a>v4

[Bot.Builder.Community.Dialogs.Luis](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.Luis/) は Bot Builder Community ライブラリになりました。  ソースはコミュニティの[リポジトリ](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.Luis)で入手できます。

## <a name="to-use-qna-maker"></a>QnA Maker を使用するには

### <a name="v3"></a>v3

```csharp
[Serializable]
[QnAMaker("QnAEndpointKey", "QnAKnowledgebaseId", <ScoreThreshold>, <TotalResults>, "QnAEndpointHostName")]
public class SimpleQnADialog : QnAMakerDialog
{
}
```

### <a name="v4"></a>v4

```csharp
public class QnABot : ActivityHandler
{
  private readonly IConfiguration _configuration;
  private readonly ILogger<QnABot> _logger;
  private readonly IHttpClientFactory _httpClientFactory;

  public QnABot(IConfiguration configuration, ILogger<QnABot> logger, IHttpClientFactory httpClientFactory)
  {
    _configuration = configuration;
    _logger = logger;
    _httpClientFactory = httpClientFactory;
  }

  protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
  {
    var httpClient = _httpClientFactory.CreateClient();

    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
    {
      KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
      EndpointKey = _configuration["QnAEndpointKey"],
      Host = _configuration["QnAEndpointHostName"]
    },
    null,
    httpClient);

    _logger.LogInformation("Calling QnA Maker");

    // The actual call to the QnA Maker service.
    var response = await qnaMaker.GetAnswersAsync(turnContext);
    if (response != null && response.Length > 0)
    {
      await turnContext.SendActivityAsync(MessageFactory.Text(response[0].Answer), cancellationToken);
    }
    else
    {
      await turnContext.SendActivityAsync(MessageFactory.Text("No QnA Maker answers were found."), cancellationToken);
    }
  }
}
```
