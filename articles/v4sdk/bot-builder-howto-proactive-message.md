---
title: プロアクティブ メッセージを送信する | Microsoft Docs
description: ボットでメッセージをプロアクティブに送信する方法について説明します。
keywords: プロアクティブ メッセージ
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c22ce6a35d4d49506360a78a439f15137c429d9d
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905136"
---
# <a name="send-proactive-messages"></a>プロアクティブ メッセージを送信する 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


多くの場合、ボットは_リアクティブ メッセージ_を送信しますが、[プロアクティブ メッセージ](bot-builder-proactive-messages.md)を送信できる必要が生じることもあります。 

プロアクティブ メッセージングは、一般的に、不特定の時間のかかるタスクをボットが実行する場合に使用します。 この場合、タスクに関する情報を格納し、タスクの完了時にボット画面に戻ることをユーザーに通知して、会話を続行することができます。 タスクが完了したら、ボットは確認メッセージをプロアクティブに送信することで、会話を再開できます。

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>このサンプルに関するメモ

基本的な EchoBot サンプルを変更します。
- 名前空間として `Microsoft.Samples.Proactive` を使用します。
- 状態ファイルを `JobData.cs` ファイルに置き換えます。
- ボット ファイルを `ProactiveBot.cs` ファイルに置き換えます。

> [!NOTE]
> プロアクティブ メッセージングでは、現在のことろ、ボットに有効な ApplicationID とパスワードが必要です。


## <a name="define-task-data"></a>タスク データを定義する

このシナリオでは、さまざまな会話でさまざまなユーザーが作成できる任意のタスクを追跡しています。 そのため、ユーザーまたは会話状態ミドルウェアではなく、一般的なボット状態ミドルウェアを使用しています。

以下のクラスは、個々のジョブに使用するデータ構造を定義します。


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


また、スタートアップ コードに状態ミドルウェアを追加する必要があります。


`StartUp.cs` ファイルで、`ConfigureServices` メソッドを更新して、ボット状態にジョブのディクショナリを追加します。 以下のコードでは、`options.Middleware.Add` への最後の呼び出しです。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>ジョブを作成して実行するようにボットを更新する

ターンごとに、ユーザーが `run` または `run job` と入力するとジョブを作成できるようにします。

応答で、ボットがそのターン内で以下の手順に従います。
- ジョブを作成します。
- プロアクティブ メッセージを後で送信するために、現在の会話に関する情報を記録します。
- ジョブを開始すること、およびジョブが完了したら後で知らせることをユーザーに通知します。
- 非同期ジョブを開始します。
- ターンを終了させます。

開始しているジョブは単純な 5 秒間のタイマーであり、その後、プロアクティブ メッセージを送信することで完了します。
- アダプターの会話続行メソッドへの呼び出しが、ボットによって開始される新しいターンを作成します。
- このターンには独自の[ターン コンテキスト](bot-builder-concept-activity-processing.md#turn-context)があり、そこから状態に関する情報を取得します。
- このコンテキストを使用して、ユーザーにプロアクティブ メッセージを送信します。



> [!NOTE]
> `GetAppId` メソッドは、.NET SDK でプロアクティブ メッセージングを有効にするための回避策です。

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

ユーザーにプロアクティブ メッセージを送信するには、まず、ユーザーがボットに少なくとも 1 つのリアクティブ スタイルのメッセージを送信する必要があります。 

アクティビティ オブジェクトへの参照を取得し、将来使用するために保存する必要があるため、ボットに 1 つのメッセージを送信する必要があります。 アクティビティ オブジェクトは、メッセージの受信時のチャネルに関する情報、そのユーザー ID、会話 ID のほか、今後のすべてのメッセージを受信するサーバーも含まれるため、ユーザー アドレスと見なすことができます。 このオブジェクトは単純な JSON であり、改ざんせずに全体を保存する必要があります。

ユーザーがサブスクライブに言及するたびに会話参照を保存する方法を示す、短いコード スニペットから始めましょう。
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
上記のスニペットは `saveReference()` 関数を呼び出し、これにより `MemoryStorage` でユーザーの参照を保存し、`userId` を返します。 参照が正常に保存されたら `subscribeUser()` を呼び出し、これによりサブスクライブされていることをユーザーに通知します。 

`subscribeUser()` 関数は、実際のサブスクリプションを設定します。 2 秒間のタイマーを開始し、タイマーが経過したらユーザーにメッセージをプロアクティブに送信する、単純な実装を見てみましょう。

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

`subscribeUser()` 関数は、ストレージから読み込むことで参照オブジェクトを検索するタイマーを設定します。 参照オブジェクトが見つかった場合は、ユーザーとの会話を続行できます。 `continueConversation` メソッドを使用すると、ボットは、既に通信した会話またはユーザーにメッセージをプロアクティブに送信できます。

---

## <a name="test-your-bot"></a>ボットをテストする

ボットをテストするには、登録専用のボットとして Azure にデプロイし、Web チャットでテストするか、またはエミュレーターを使用してローカルでテストします。
