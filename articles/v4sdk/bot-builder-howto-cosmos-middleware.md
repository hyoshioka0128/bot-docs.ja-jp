---
title: Cosmos DB を使用してミドルウェアを作成する | Microsoft Docs
description: Cosmos DB へログを記録するカスタム ミドルウェアの作成方法について説明します。
keywords: ミドルウェア、cosmos db、データベース、azure、onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f36b00a91644859445676676374f7d321087800
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906103"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>Cosmos DB でログを記録するミドルウェアを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

便利なミドルウェアが SDK で提供されている一方で、状況によっては、一定の目標を達成するために独自のミドルウェアを実装する必要があります。

このサンプルでは、Cosmos DB に接続するミドルウェア レイヤーを作成して、受信したメッセージと、返された返答をすべてログに記録します。 この記事に示したコードは、[サンプル](../dotnet/bot-builder-dotnet-samples.md)のページで提供されており、完全なコードとして入手できます。

## <a name="set-up"></a>セットアップ

コードに取り組む前に、いくつかの設定を行う必要があります。

### <a name="create-your-database"></a>データベースを作成する

最初に、データベースを格納する場所が必要です。  これは、Azure アカウント経由で行うか、またはテストを目的とする場合は [Cosmos DB Emulator](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) を使用できます。 どちらの方法を選択した場合も、データベースのエンドポイント URI とキーが必要になります。

サンプルおよびこの記事に示したコードでは、エミュレーターを使用しています。 エンドポイントとキーは、Cosmos DB テスト アカウントに対して提供されているものです。これは、エミュレーターのドキュメントに提供されている共通の値であり、実稼働環境には使用できません。

実際の Azure Cosmos DB を使用する場合は、[Cosmos DB の使用](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started)に関するトピックの手順 1 に従ってください。 手順を完了すると、データベース設定の **[キー]** タブで、エンドポイント URI とキーの両方が利用可能になります。 これらの値は、後述する構成ファイルで必要になります。

### <a name="build-a-basic-bot"></a>基本のボットを作成する

このトピックの以降の説明は、概要のページで作成した HelloBot などの基本のボットから作成しています。 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) を使用して、ボットへの接続、対話、およびテストが可能です。

Bot Framework を使用して標準のボットに必要な参照だけでなく、次の NuGet パッケージへの参照を追加する必要があります。

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

これらのライブラリはそれぞれ、データベースとの通信に使用され、構成ファイルの使用を可能にします。

### <a name="add-configuration-file"></a>Add configuration file

いくつかの理由から、構成ファイルの使用が推奨されているため、ここでは 1 つの構成ファイルを使用します。 使用する構成データは短くて単純ですが、ボットがより複雑になる場合は、ここに他の構成設定を追加できます。

# <a name="ctabcs"></a>[C#](#tab/cs)
1. `app.config` という名前のプロジェクトにファイルを追加します。
2. ファイルに次の情報を保存します。 エンドポイントとキーは、ローカル エミュレーターで定義されますが、Cosmos DB インスタンスで更新できます。
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. `config.js` という名前のプロジェクトにファイルを追加します。
2. ファイルに次の情報を保存します。 エンドポイントとキーは、ローカル エミュレーターで定義されますが、Cosmos DB インスタンスで更新できます。
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

実際の Cosmos DB を使用している場合は、上記の 2 つのキー値を Cosmos DB 設定で保持している値に置換できます。 または、テスト用と実際の Cosmos DB 用の間で切り替えられるように、構成に次の 2 つのキーをさらに追加できます。

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 2 つのキーをさらに追加して実際のデータベースを使用する場合、必ず下のコンストラクターに、`EmulatorDbUrl` および `EmulatorDbKey` の代わりに正しいキーを指定してください。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

2 つのキーをさらに追加して実際のデータベースを使用する場合、必ず下のコンストラクターに、`EmulatorServiceEndpoint` および `EmulatorAuthKey` の代わりに正しいキーを指定してください。

---



## <a name="creating-your-middleware"></a>ミドルウェアを作成する

# <a name="ctabcs"></a>[C#](#tab/cs)
実際のミドルウェア ロジックの記述を開始する前に、ミドルウェアのボット プロジェクトに新しいクラスを作成します。 クラスを作成したら、ボットの残りの部分で使用する `Microsoft.Bot.Samples` に名前空間を変更します。 今回は、次に示す数個の新しいライブラリへの参照を追加したことがわかります。

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

各種の `Microsoft.Azure.Documents.*` は、データベースとの通信の別の部分に対応しており、`Newtonsoft.Json` はデータベースに入出力される JSON の解析に役立ちます。

最後に、ミドルウェアが適切に動作するように、ミドルウェアの各部分に適切なハンドラーの定義を求める `IMiddleware` を実装します。

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
実際のミドルウェア ロジックの記述を開始する前に、`onTurn()` で始まるカスタム ミドルウェアを作成する必要があります。 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>ローカル変数の定義

# <a name="ctabcs"></a>[C#](#tab/cs)
次に、使用するデータベースと、ログに記録したい情報を格納するクラスを操作するための、ローカル変数が必要になります。 この `Log` と呼ばれる情報クラスでは、各メンバーに関連付けられている JSON プロパティに付与される名前を定義します。 これらの名前は、後で使用します。

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
次に、ログに記録する情報を格納するためのローカル変数が必要になります。 ミドルウェアでは、ストレージ プロバイダーとして Cosmos DB を保持する conversationState にアクセスする必要があります。 変数 `info` は、読み取りおよび書き込みの対象となる状態のオブジェクトです。 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>データベース接続を初期化する

# <a name="ctabcs"></a>[C#](#tab/cs)
このミドルウェアのコンストラクターは、先ほど定義した構成ファイルから必要な情報をプルして、`DocumentClient` を作成する処理を担います。これにより、データベースに対する読み取りおよび書き込みが可能になります。 また、データベースとコレクションが設定されていることを確認します。

新しい `DocumentClient` の正しい使用方法として、破棄することが求められるため、デストラクターがこれを行います。

データベースの作成は、最初にデータベース、次にそのデータベース内のコレクションの基本的な読み取りの試行に依存しています。 `NotFound` 例外が発生した場合、スタックにスローするのではなく、この例外をキャッチしてデータベースまたはコレクションを作成します。 ほとんどの場合、データベースやコレクションは事前に作成されているでしょうが、この例外時の動作のおかげで、このサンプルでは、最初の実行前にデータベースの作成を考慮しなくてもかまいません。 サンプル コードでは、わかりやすくするためにデータベースとコレクションは 2 つの関数に分割されていますが、以下のスニペットでは統合されています。

CosmosDB の詳細については、該当の [Web サイト](https://docs.microsoft.com/en-us/azure/cosmos-db/)を参照してください。 このトピックの以降の部分では、Cosmos DB 自体の詳細について簡単に確認し、Cosmos DB を使用するためにボットで必要なものに着目します。

`Key`、`Endpoint`、および `docClient` の定義は上記のスニペットに含まれています。わかりやすくするために、以下にそのコピーのみを示します。

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
データベースの作成は、最初にデータベース、次にそのデータベース内のコレクションの基本的な読み取りの試行に依存しています。 `undefined` が発生した場合、これをキャッチして、空の配列に設定される `log` というオブジェクトを作成します。 ほとんどの場合、`log` は事前に作成されているでしょうが、この例外時の動作のおかげで、このサンプルでは、最初の実行前にこの作成に関して考慮しなくてもかまいません。 サンプル コードでは、`info.log` が未定義かどうかを確認します。 未定義の場合は、空の配列にこれを設定します。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>データベース ロジックから読み取りを行う

最後のヘルパー関数では、データベースから読み取りを行い、直近の指定レコード数を返します。 特にデータ ストアが非常に大規模な場合は、ここで使用している方法よりも、データを取得するためのより優れた最善策があることに注目してください。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

以下のコードはまだ `onTurn()` 関数内にあり、ユーザーが文字列 "history" を入力した場合に呼び出されます。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>OnTurn() を定義する

標準的なミドルウェア メソッド `OnTurn()` では、残りの作業を処理します。 現在のアクティビティがメッセージであり、`next()` の呼び出し前後の両方にチェックを行う場合にのみ、ログに記録したいと考えています。

最初のチェック事項は、ユーザーが直近の履歴を必要とている特別なケースに該当するメッセージかどうかです。 該当する場合、データベースから read を呼び出して、直近のレコードを取得し、その内容をconversation に送信します。 これで現在のアクティビティが完全に処理されるので、実行に渡らずにパイプラインが回避されます。

ボットが送信する応答を取得するには、メッセージを受信するたびにハンドラーを作成して、これらの応答を捕捉します。応答の内容は、`OnSendActivity()` に渡すラムダで確認できます。 ラムダでは、このコンテキスト オブジェクトに対して `SendActivity()` 経由で送信されたすべてのメッセージを収集するための文字列を作成します。

実行によって `next()` からパイプラインが返されたら、ログ データを組み立てて、データベースに書き出します。 

数個のメッセージがこのボットに送信された後に、Cosmos DB データ エクスプローラーを見て、個々のレコードのデータを確認する必要があります。 これらのレコードの 1 つを確認するときは、最初の 3 つの項目がログ データの 3 つの値であり、各 `JsonProperty` に対して指定した文字列の名前になっていることを確認する必要があります。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`onTurn` 関数の上記の説明を基に、コードを作成しています。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>サンプル出力

完全なサンプルでは、ボットはプロンプト ライブラリを使用して、ユーザーの名前と好きな番号を要求します。 プロンプト ライブラリの詳細は、[ユーザーのプロンプト](~/v4sdk/bot-builder-prompts.md)に関するトピックで確認できます。

以下は、上記で説明したコードを実行しているサンプル ボットの会話例を示しています。

![Cosmos ミドルウェア例の会話](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>その他のリソース

* [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Cosmos DB Emulator](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)

