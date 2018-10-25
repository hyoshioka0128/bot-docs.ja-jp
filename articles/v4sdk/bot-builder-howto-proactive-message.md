---
title: プロアクティブ メッセージングの使用方法 | Microsoft Docs
description: ボットを使用してプロアクティブ メッセージを送信する方法について説明します。
keywords: プロアクティブ メッセージ
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 032d7027db3ce83c54bbacf913c2021a22c3f356
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997589"
---
# <a name="how-to-use-proactive-messaging"></a>プロアクティブ メッセージングの使用方法

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

通常、ボットがユーザーに送信する各メッセージは、ユーザーの前の入力に直接関連しています。
場合によっては、ボットは、会話の現在のトピックまたはユーザーによって最後に送信されたメッセージに直接関連していないメッセージをユーザーに送信する必要があります。 この種のメッセージは、_プロアクティブ メッセージ_と呼ばれます。

## <a name="uses"></a>用途

プロアクティブ メッセージは、さまざまなシナリオで役立ちます。 ボットがタイマーやリマインダーを設定している場合は、その時刻になったときにユーザーに通知する必要があります。 または、ボットが外部システムから通知を受信した場合は、その情報をユーザーにすぐに伝達する必要があります。 たとえば、ユーザーが製品の価格を監視するようにボットに依頼していた場合、ボットは、製品の価格が 20% 下がった場合にユーザーにアラートを発行できます。 ユーザーの質問に対する回答をまとめるのに時間が必要な場合、ボットは、時間がかかることをユーザーに通知して、会話を続けられるようにします。 質問の回答がまとまったら、ボットは、その情報をユーザーと共有します。

ボットでプロアクティブ メッセージを実装するときは:

- 短い時間で複数のプロアクティブ メッセージを送信しない。 一部のチャネルでは、ボットがユーザーにメッセージを送信できる頻度に制限が設定され、これらの制限に違反した場合は、ボットが無効になります。
- これまでボットと対話したことがないか、メールや SMS などの別の手段を通じてボットと接触することを求められたことのないユーザーに対して、プロアクティブ メッセージを送信しない。

**アドホック型のプロアクティブ メッセージ**は、最も単純な種類のプロアクティブ メッセージです。
ボットは会話が開始されると単純に会話にメッセージを差し挟みます。つまり、ユーザーが現在、ボットと別の話題で会話中であり、話題を変えるつもりがない場合でも考慮しません。

通知をより円滑に処理するには、会話フローに通知を統合するための他の方法を検討してください (会話の状態にフラグを設定する方法や、通知をキューに追加する方法など)。

## <a name="prerequisites"></a>前提条件

プロアクティブ メッセージを送信するには、ボットが有効なアプリ ID とパスワードを持っている必要があります。 ただし、エミュレーターでのローカル テストでは、プレースホルダー アプリ ID を使用できます。

ボットで使用するアプリ ID とパスワードを取得するには、[Azure portal](https://portal.azure.com) にログインして、**Bot Channels Registration** リソースを作成します。 テストの目的で、ボット用のこのアプリ ID とパスワードを Azure にデプロイせずに、ローカルで使用することができます。

> [!TIP]
> サブスクリプションがない場合は、<a href="https://azure.microsoft.com/en-us/free/" target="_blank">無料アカウント</a>に登録できます。

### <a name="required-libraries"></a>必要なライブラリ

いずれかの BotBuilder テンプレートから開始する場合、必要なライブラリは自動的にインストールされます。 これらは、プロアクティブ メッセージングに必要な固有の BotBuilder ライブラリです。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Microsoft.Bot.Builder.Integration.AspNet.Core** NuGet パッケージ  (これをインストールすると、**Microsoft.Bot.Builder** パッケージもインストールされます)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Microsoft.Bot.Builder** npm パッケージ。

---

## <a name="notes-on-the-sample-code"></a>サンプル コードに関する注意事項

この記事のコードは、プロアクティブ メッセージのサンプル ([C#](https://aka.ms/proactive-sample-cs) | [JS](https://aka.ms/proactive-sample-js)) から取得したものです。

このサンプルでは、所要時間が不確定なユーザー タスクをモデル化します。 ボットはタスクに関する情報を格納し、タスクが完了したら戻ることをユーザーに通知して、会話を続行することができます。 タスクが完了すると、ボットは確認メッセージを元の会話にプロアクティブに送信します。

## <a name="define-job-data-and-state"></a>ジョブのデータと状態を定義する

このシナリオでは、さまざまな会話でさまざまなユーザーが作成できる任意のジョブを追跡しています。 会話の参照やジョブ識別子など、各ジョブに関する情報を格納する必要があります。

- プロアクティブ メッセージを適切な会話に送信できるようにするために、会話の参照が必要です。
- ジョブを識別するための手段が必要です。 この例では、単純なタイムスタンプを使用します。
- 会話の状態やユーザーの状態とは別に、ジョブの状態を格納する必要があります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ジョブ データとジョブの状態に関するクラスを定義する必要があります。 ボットを登録し、ジョブのログ用に状態プロパティのアクセサーを設定する必要もあります。

### <a name="define-a-class-for-job-data"></a>ジョブ データに関するクラスを定義する

**JobLog** クラスは、ジョブ番号 (タイムスタンプ) によってインデックス付けされたジョブ データを追跡します。 ジョブ データはディクショナリの内部クラスとして定義されます。

```csharp
/// <summary>Contains a dictionary of job data, indexed by job number.</summary>
/// <remarks>The JobLog class tracks all the outstanding jobs.  Each job is
/// identified by a unique key.</remarks>
public class JobLog : Dictionary<long, JobLog.JobData>
{
    /// <summary>Describes the state of a job.</summary>
    public class JobData
    {
        /// <summary>Gets or sets the time-stamp for the job.</summary>
        /// <value>
        /// The time-stamp for the job when the job needs to fire.
        /// </value>
        public long TimeStamp { get; set; } = 0;

        /// <summary>Gets or sets a value indicating whether indicates whether the job has completed.</summary>
        /// <value>
        /// A value indicating whether indicates whether the job has completed.
        /// </value>
        public bool Completed { get; set; } = false;

        /// <summary>
        /// Gets or sets the conversation reference to which to send status updates.
        /// </summary>
        /// <value>
        /// The conversation reference to which to send status updates.
        /// </value>
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-middleware-class"></a>状態ミドルウェア クラスを定義する

**JobState** クラスは、会話の状態やユーザーの状態とは別に、ジョブの状態を管理します。

```csharp
using Microsoft.Bot.Builder;

/// <summary>A <see cref="BotState"/> for managing bot state for "bot jobs".</summary>
/// <remarks>Independent from both <see cref="UserState"/> and <see cref="ConversationState"/> because
/// the process of running the jobs and notifying the user interacts with the
/// bot as a distinct user on a separate conversation.</remarks>
public class JobState : BotState
{
    /// <summary>The key used to cache the state information in the turn context.</summary>
    private const string StorageKey = "ProactiveBot.JobState";

    /// <summary>
    /// Initializes a new instance of the <see cref="JobState"/> class.</summary>
    /// <param name="storage">The storage provider to use.</param>
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    /// <summary>Gets the storage key for caching state information.</summary>
    /// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
    /// for processing this conversation turn.</param>
    /// <returns>The storage key.</returns>
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>ボットと必要なサービスを登録する

**Startup.cs** ファイルは、ボットおよび関連するサービスを登録します。

1. 一連の using ステートメントは、次の名前空間の参照まで拡張されます。

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. `ConfigureServices` メソッドは、エラー処理や状態管理も含め、ボットを登録します。 また、ボットのエンドポイント サービスやジョブの状態のアクセサーも登録します。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // ...

        // Create Job State object.
        // The Job State object is where we persist anything at the job-scope.
        // Note: It's independent of any user or conversation.
        var jobState = new JobState(dataStore);

        // Make it available to our bot
        services.AddSingleton(sp => jobState);

        // Register the proactive bot.
        services.AddBot<ProactiveBot>(options =>
        {
            var secretKey = Configuration.GetSection("botFileSecret")?.Value;
            var botFilePath = Configuration.GetSection("botFilePath")?.Value;

            // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
            var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
            services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

            // Retrieve current endpoint.
            var environment = _isProduction ? "production" : "development";
            var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
            if (!(service is EndpointService endpointService))
            {
                throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
            }

            options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

            // Creates a logger for the application to use.
            ILogger logger = _loggerFactory.CreateLogger<ProactiveBot>();

            // Catches any errors that occur during a conversation turn and logs them.
            options.OnTurnError = async (context, exception) =>
            {
                logger.LogError($"Exception caught : {exception}");
                await context.SendActivityAsync("Sorry, it looks like something went wrong.");
            };

        });

        services.AddSingleton(sp =>
        {
            var config = BotConfiguration.Load(@".\BotConfiguration.bot");
            var endpointService = (EndpointService)config.Services.First(s => s.Type == "endpoint")
                                    ?? throw new InvalidOperationException(".bot file 'endpoint' must be configured prior to running.");

            return endpointService;
        });
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="set-up-the-server-code"></a>サーバー コードを設定する

**index.js** ファイルは次の処理を実行します。

- 必要なパッケージとサービスを含める
- ボットのクラスと **.bot** ファイルを参照する
- HTTP サーバーを作成する
- ボット アダプターとストレージ オブジェクトを作成する
- ボットを作成してサーバーを起動し、ボットにアクティビティを渡す

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const restify = require('restify');
const path = require('path');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, BotState, MemoryStorage } = require('botbuilder');
const { BotConfiguration } = require('botframework-config');

const { ProactiveBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file.
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open proactive-messages.bot file in the Emulator.`);
});

// .bot file path
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));

// Read the bot's configuration from a .bot file identified by BOT_FILE.
// This includes information about the bot's endpoints and configuration.
let botConfig;
try {
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.\n\n`);
    process.exit();
}

const DEV_ENVIRONMENT = 'development';

// Define the name of the bot, as specified in .bot file.
// See https://aka.ms/about-bot-file to learn more about .bot files.
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Load the configuration profile specific to this bot identity.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);

// Create the adapter. See https://aka.ms/about-bot-adapter to learn more about using information from
// the .bot file when configuring your adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.MicrosoftAppId,
    appPassword: endpointConfig.appPassword || process.env.MicrosoftAppPassword
});

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create state manager with in-memory storage provider.
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
};
```

---

<!--TODO: (Post-Ignite) -- link to a second topic on how to write a job completion DirectLine client that will generate appropriate job completed event activities.-->

## <a name="define-the-bot"></a>ボットを定義する

ユーザーはボットにジョブの作成と実行を依頼できます。 ジョブの完了時に、個別のジョブ サービスでボットに通知できます。

ボットは次のように設計されています。

- ユーザーからの `run` または `run job` メッセージに応答して、ジョブを作成します。
- ユーザーからの `show` または `show jobs` メッセージに応答して、登録済みのすべてのジョブを表示します。
- 完了したジョブを識別する _job completed_ イベントに応答して、ジョブを完了します。
- `done <jobIdentifier>` メッセージに応答して、ジョブの完了イベントをシミュレートします。
- ジョブの完了時に、元の会話を使用してユーザーにプロアクティブ メッセージを送信します。

イベント アクティビティをボットに送信できるシステムの実装方法については取り上げません。
<!--TODO: DirectLine--Add back in once the DirectLine topic is added back to the TOC.
See [how to create a Direct Line bot and client](bot-builder-howto-direct-line.md) for information on how to do so.
-->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ボットにはいくつかの側面があります。

- 初期化コード
- ターン ハンドラー
- ジョブの作成と完了のためのメソッド

### <a name="declare-the-class"></a>クラスを宣言する

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Schema;

namespace Microsoft.BotBuilderSamples
{
    /// <summary>
    /// For each interaction from the user, an instance of this class is called.
    /// This is a Transient lifetime service.  Transient lifetime services are created
    /// each time they're requested. For each Activity received, a new instance of this
    /// class is created. Objects that are expensive to construct, or have a lifetime
    /// beyond the single Turn, should be carefully managed.
    /// </summary>
    public class ProactiveBot : IBot
    {
        /// <summary>The name of events that signal that a job has completed.</summary>
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>初期化コードを追加する

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    // Validate AppId.
    // Note: For local testing, .bot AppId is empty for the Bot Framework Emulator.
    AppId = string.IsNullOrWhiteSpace(endpointService.AppId) ? "1" : endpointService.AppId;
}

private string AppId { get; }
```

### <a name="add-a-turn-handler"></a>ターン ハンドラーを追加する

すべてのボットにターン ハンドラーを実装する必要があります。 アダプターはアクティビティをこのメソッドに転送します。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":

                // Display information for all jobs in the log.
                if (jobLog.Count > 0)
                {
                    await turnContext.SendActivityAsync(
                        "| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>" +
                        "| :--- | :---: | :---: |<br>" +
                        string.Join("<br>", jobLog.Values.Select(j =>
                            $"| {j.TimeStamp} &nbsp; | {j.Conversation.Conversation.Id.Split('|')[0]} &nbsp; | {j.Completed} |")));
                }
                else
                {
                    await turnContext.SendActivityAsync("The job log is empty.");
                }

                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}

// Handles non-message activities.
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    // On a job completed event, mark the job as complete and notify the user.
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>ジョブの作成と完了のメソッドを追加する

ジョブを開始するために、ボットはジョブを作成して、そのジョブに関する情報と現在の会話をジョブ ログに記録します。 ボットは任意の会話でジョブ完了イベントを受け取ると、そのジョブ ID を検証したうえで、ジョブを完了するコードを呼び出します。

ジョブを完了するコードは、状態からジョブ ログを取得してから、ジョブを "完了" としてマークし、プロアクティブ メッセージを送信します。これにはアダプターの _continue conversation_ メソッドが使用されます。

- continue conversation の呼び出しにより、チャネルはユーザーに依存しないターンを開始するよう求められます。
- アダプターは、ターン ハンドラーでボットの通常のコールバックの代わりに、関連するコールバックを実行します。 このターンにはそれ自体のターン コンテキストがあり、そこから状態に関する情報を取得して、ユーザーにプロアクティブ メッセージを送信します。

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}

// Sends a proactive message to the user.
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}

// Creates the turn logic to use for the proactive message.
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

ボットは **bot.js** で定義され、いくつかの側面があります。

- 初期化コード
- ターン ハンドラー
- ジョブの作成と完了のためのメソッド

### <a name="declare-the-class-and-add-initialization-code"></a>クラスを宣言して初期化コードを追加する

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>ターン ハンドラー

`onTurn` メソッドと `showJobs` メソッドは、`ProactiveBot` クラス内で定義されます。 `onTurn` はユーザーからの入力を処理します。 また、仮想ジョブ フルフィルメント システムからのイベント アクティビティも受け取ります。 `showJobs` はジョブ ログを書式設定して送信します。

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>ジョブを開始するためのロジック

`createJob` メソッドは、`ProactiveBot` クラス内で定義されます。 これは、ユーザーに対して新しいジョブを作成してログ記録します。 理論上は、この情報をジョブ フルフィルメント システムにも転送します。

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>ジョブを完了するためのロジック

`completeJob` メソッドは、`ProactiveBot` クラス内で定義されます。 これは、何らかのブックキーピングを実行し、ユーザーに対して、ジョブが完了したというプロアクティブ メッセージを (そのユーザーの元の会話内で) 送信します。

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>ボットをテストする

ボットをローカルで構築して実行し、 2 つのエミュレーター ウィンドウを開きます。

1. この 2 つのウィンドウで会話 ID が異なることに注意してください。
1. 1 番目のウィンドウで「`run`」と何度か入力し、いくつかのジョブを開始します。
1. 2 番目のウィンドウで「`show`」と入力し、ログ内のジョブの一覧を表示します。
1. 2 番目のウィンドウで「`done <jobNumber>`」と入力します。`<jobNumber>` はログ内のジョブの番号のいずれかで、山かっこはつけません  (ボット コードはこれを jobComplete イベントのように解釈するよう設計されています)。
1. 1 番目のウィンドウでボットがユーザーにプロアクティブ メッセージを送信していることに注意してください。

<!--TODO: Recreate the screen shots once we're happy with both the C# and JS versions of the code.-->

ユーザーから見ると、会話は次のようになります。

![ユーザーのエミュレーター セッション](~/v4sdk/media/how-to-proactive/user.png)

シミュレートされたジョブ システムから見ると、次のようになります。

![ジョブ システムのエミュレーター セッション](~/v4sdk/media/how-to-proactive/job-system.png)

<!-- Add a next steps section. -->
