---
title: ストレージに直接書き込む | Microsoft Docs
description: Bot Framework SDK for .NET でストレージに直接書き込む方法について説明します。
keywords: ストレージ, 読み取りと書き込み, メモリ ストレージ, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 268ac62c68d2157a50d3b20cddf7e8f54cccaefe
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591203"
---
# <a name="write-directly-to-storage"></a>ストレージに直接書き込む

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ミドルウェアまたはコンテキスト オブジェクトを使用せずに、ストレージ オブジェクトに対して直接読み取りや書き込みを行うことができます。 ボットの会話フローの外にあるソースからのデータをボットが使用する場合は、この方法が適切な可能性があります。 たとえば、ユーザーが天気予報を尋ねるボットで、外部データベースからの読み取りによって指定日の天気予報を取得するとします。 天気データベースの内容はユーザーの情報または会話のコンテキストに依存しないため、状態マネージャーを使用する代わりに、単純にストレージから予報を直接読み取ることができます。 この記事のコード例では、**メモリ ストレージ**、**Cosmos DB**、**Blob Storage**、および **Azure Blob Transcript Store** を使用してストレージに対するデータの読み取りや書き込みを行う方法を示します。 

## <a name="prerequisites"></a>前提条件
- Azure サブスクリプションをお持ちでない場合は、開始する前に[無料](https://azure.microsoft.com/en-us/free/)アカウントを作成してください。
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started) のインストール

## <a name="memory-storage"></a>メモリ ストレージ

メモリ ストレージに対するデータの読み取りや書き込みを行うボットを最初に作成します。 メモリ ストレージはテストにのみ使用され、実稼働を目的としたものではありません。 ボットを発行する前に、ストレージを Cosmos DB または Blob Storage に設定してください。

#### <a name="build-a-basic-bot"></a>基本のボットを作成する

このトピックの残りの部分では、エコー ボットについては取り上げません。 このプロジェクトの作成に使用するエコー ボットのサンプル コードは、[こちら (C# のサンプル)](https://aka.ms/cs-echobot-sample) と[こちら (JS のサンプル)](https://aka.ms/js-echobot-sample) にあります。 Bot Framework Emulator を使用して、ボットへの接続、対話、およびテストが可能です。 次のサンプルでは、ユーザーからのすべてのメッセージをリストに追加します。 このリストを含むデータ構造は、その後、お使いのストレージに保存されます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{     
   if (turnContext.Activity.Type == ActivityTypes.Message)
   {
      // Replace the two lines of code from original MyBot code with the following:
      
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
            // add the current utterance to a new object.
            logItems = new UtteranceLog();
            logItems.UtteranceList.Add(utterance);
            // set initial turn counter to 1.
            logItems.TurnNumber++;

            // Show user new user message.
            await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

            // Create Dictionary object to hold received user messages.
            var changes = new Dictionary<string, object>();
            {
               changes.Add("UtteranceLog", logItems);
            }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
   }
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**
```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Create server.
// let server = restify.createServer();
// server.listen(process.env.port || process.env.PORT || 3978, function () {
//    console.log(`${server.name} listening to ${server.url}`);
// });

// Create adapter.
// const adapter = new BotFrameworkAdapter({
//    appId: process.env.MICROSOFT_APP_ID,
//    appPassword: process.env.MICROSOFT_APP_PASSWORD
//  });

// Add memory storage.
var storage = new MemoryStorage();

// const conversationState = new ConversationState(storage);
// adapter.use(conversationState);

// Listen for incoming requests - adds storage for messages.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {

        if (context.activity.type === 'message') {
            // Route to main dialog.
            await myBot.onTurn(context);
            // Save updated utterance inputs.
            await logMessageText(storage, context);
        }
        else {
            // Just route to main dialog.
            await myBot.onTurn(context);
        } 
    });
});

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];

        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }

         } else {
            context.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}

```

---

### <a name="start-your-bot"></a>ボットの起動
ボットをローカルで実行します。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. プロジェクトを作成したディレクトリにある .bot ファイルを選択します。

### <a name="interact-with-your-bot"></a>ボットでのやり取り
メッセージをボットに送信します。ボットは受信したメッセージを表示します。
![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>Cosmos DB の使用
メモリ ストレージを使用しているので、Azure Cosmos DB を使用するようにコードを更新します。 Cosmos DB は、Microsoft のグローバル分散型マルチモデル データベースです。 Azure Cosmos DB では、Azure のリージョンをいくつでもまたいでスループットとストレージを柔軟かつ個別にスケーリングすることができます。 このサービスは包括的なサービス レベル アグリーメント (SLA) により、スループット、待機時間、可用性、一貫性が保証されています。 

### <a name="set-up"></a>セットアップ
ボットで Cosmos DB を使用するには、コードに取り組む前に、いくつかの設定を行う必要があります。

#### <a name="create-your-database-account"></a>データベース アカウントの作成
1. 新しいブラウザー ウィンドウで、[Azure Portal](http://portal.azure.com) にサインインします。
2. **[リソースの作成]、[データベース]、[Azure Cosmos DB]** の順にクリックします。
3. **[新規アカウント] ページ**の **[ID]** フィールドに一意の名前を指定します。 **[API]** として **[SQL]** を選択し、**[サブスクリプション]**、**[場所]**、および **[リソース グループ]** に情報を指定します。
4. **[Create]** をクリックします。

アカウントの作成には数分かかります。 ポータルに " Azure Cosmos DB アカウントが作成されました" ページが表示されるまで待機します。

##### <a name="add-a-collection"></a>コレクションの追加
1. **[設定]、[新しいコレクション]** の順にクリックします。 **[コレクションの追加]** 領域が右端に表示されます。表示するには、右にスクロールする必要がある場合があります。 Cosmos DB が最近更新されたため、_/id_ でパーティション キーを 1 つ必ず追加します。このキーにより、クロス パーティション クエリのエラーが回避されます。

![Cosmos DB コレクションの追加](./media/add_database_collection.png)

2. 新しいデータベース コレクションは "bot-cosmos-sql-db" です。コレクション ID は "bot-storage" になります。 これらの値は、以降に示すコード例で使用します。
 -
![Cosmos DB](./media/cosmos-db-sql-database.png)

3. データベース設定の **[キー]** タブで、エンドポイント URI とキーが利用可能になります。 これらの値は、この記事の後半のコードの構成で必要になります。 

![Cosmos DB のキー](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>構成情報の追加
Cosmos DB ストレージを追加するための構成データは短くシンプルなものであり、ボットがより複雑になった場合でも、同じ方法を使用して構成設定を追加できます。 この例では、上記の例の Cosmos DB データベースとコレクションの名前を使用します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`.env` ファイルに以下の情報を追加します。

**.env**
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=bot-cosmos-sql-db
COLLECTION=bot-storage
```
---

#### <a name="installing-packages"></a>パッケージのインストール
Cosmos DB に必要なパッケージがあることを確認します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

npm を使用して、botbuilder-azure への参照をプロジェクトで追加します。


```powershell
npm install --save botbuilder-azure 
```

.env 構成ファイルを使用するには、ボットに追加のパッケージが含まれている必要があります。 最初に、dotnet パッケージを npm から取得します。

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>実装 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

次のサンプル コードは、上記の[メモリ ストレージ](#memory-storage)のサンプルと同じボット コードを使用して実行されます。
次のコード スニペットは、ローカルのメモリ ストレージを置き換える "_myStorage_" に対する Cosmos DB ストレージの実装を示しています。

**MyBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

// Create local Memory Storage - commented out.
// private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

次のサンプル コードは[メモリ ストレージ](#memory-storage)と似ていますが、わずかに変更されています。

botbuilder-azure の `CosmosDbStorage` が必要です。また、`.env` ファイルを読み取るように dotenv を構成します。

**index.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
メモリ ストレージをコメント アウトし、Cosmos DB への参照で置き換えます。

**index.js**
```javascript
// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to Cosmos DB storage.
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
     collectionId: process.env.COLLECTION
})

```

---

## <a name="start-your-bot"></a>ボットの起動
ボットをローカルで実行します。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. プロジェクトを作成したディレクトリにある .bot ファイルを選択します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り
メッセージをボットに送信します。ボットは受信したメッセージを表示します。
![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>データの表示
ボットを実行して情報を保存した後は、Azure portal の **[データ エクスプローラー]** タブで、格納データを表示できます。 

![データ エクスプローラーの例](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>eTag を使用してコンカレンシーを管理する
ボット コード例では、各 `IStoreItem` の `eTag` プロパティを `*` に設定します。 ストア オブジェクトの `eTag` (エンティティ タグ) メンバーは、コンカレンシーを管理するために Cosmos DB 内で使用されます。 `eTag` は、自分のボットが書き込んでいる同じストレージ内のオブジェクトをボットの別のインスタンスが変更した場合の対処方法をデータベースに通知します。 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最後の書き込みが有効 - 上書きを許可する
`eTag` プロパティの値にアスタリスク (`*`) を指定すると、最後の書き込みが有効であることを示します。 新しいデータ ストアを作成するときは、プロパティの `eTag` を `*` に設定して、以前に保存したことのないデータを書き込んでいること、または以前に保存されたすべてのプロパティを最後の書き込みで上書きすることを示します。 コンカレンシーがボットにとって問題にならない場合は、書き込んでいるすべてのデータについて `eTag` プロパティを `*` に設定して上書きを許可します。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>コンカレンシーを維持して上書きを禁止する
データを Cosmos DB に保存する際に、プロパティへの同時アクセスを禁止し、変更をボットの別のインスタンスによって上書きされないようにする場合は、`*` 以外の値を `eTag` に使います。 ボットが状態データを保存しようとして、`eTag` がストレージ内の `eTag` と同じ値でない場合、ボットは `etag conflict key=` というメッセージのエラー応答を受け取ります。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

既定では、Cosmos DB ストアはボットがストレージ オブジェクトを書き込むたびにその項目の `eTag` プロパティが等しいかどうかを確認し、各書き込みの後で新しい一意の値に更新します。 書き込みの `eTag` プロパティがストレージの `eTag` と一致しない場合は、別のボットまたはスレッドがデータを変更したことを意味します。 

たとえば、保存されたメモをボットで編集しますが、ボットの別のインスタンスが行った変更を上書きしないようにする必要があるものとします。 ボットの別のインスタンスが編集を行った場合は、ユーザーに最新の更新でバージョンを編集させます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

最初に、`IStoreItem` を実装するクラスを作成します。

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

次に、ストレージ オブジェクトを作成することによって最初のメモを作成し、オブジェクトをストアに追加します。

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

後でメモにアクセスして更新し、ストアから読み取ったその `eTag` を維持します。

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

変更を書き込む前にストアのメモが更新されていた場合、`Write` の呼び出しでは例外がスローされます。


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

データ ストアにサンプルのメモを書き込むヘルパー関数をボットの末尾に追加します。
最初に、`myNoteData` オブジェクトを作成します。

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

`createSampleNote` ヘルパー関数内で、`changes` オブジェクトを初期化し、"*メモ*" をそれに追加してから、ストレージに書き込みます。

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

次に、メモに後からアクセスして更新するために、ユーザーが "メモの更新" を入力するとアクセス可能な別のヘルパー関数を作成します。

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

変更を書き込む前にストアのメモが更新されていた場合、`write` の呼び出しでは例外がスローされます。

---

コンカレンシーを維持するには、常にストレージからプロパティを読み取った後、読み取ったプロパティを変更して、`eTag` が維持されるようにします。 ストアからユーザー データを読み取る場合、応答には eTag プロパティが含まれます。 データを変更して更新後のデータをストアに書き込む場合、要求に含まれる eTag プロパティでは、前に読み取ったのと同じ値が指定されている必要があります。 ただし、`eTag` を `*` に設定してオブジェクトを書き込むと、書き込みで他の変更を上書きできます。

## <a name="using-blob-storage"></a>Blob Storage の使用 
Azure Blob Storage は、Microsoft のクラウド用オブジェクト ストレージ ソリューションです。 BLOB ストレージは、テキスト データやバイナリ データなどの大量の非構造化データを格納するために最適化されています。

### <a name="create-your-blob-storage-account"></a>Blob Storage アカウントの作成
ボットで Blob Storage を使用するには、コードに取り組む前に、いくつかの設定を行う必要があります。
1. 新しいブラウザー ウィンドウで、[Azure Portal](http://portal.azure.com) にサインインします。
2. **[リソースの作成]、[Storage]、[ストレージ アカウント - Blob、File、Table、Queue]** の順にクリックします。
3. **[新規アカウント] ページ**で、ストレージ アカウントの **[名前]** を入力し、**[アカウントの種類]** として **[Blob ストレージ]** を選択し、**[場所]**、**[リソース グループ]**、および **[サブスクリプション]** の情報を指定します。  
4. **[Create]** をクリックします。

![Blob Storage の作成](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>構成情報の追加

上記のボットの Blob Storage の構成に必要な Blob Storage キーを検索します。
1. Azure portal で、Blob Storage アカウントを開き、**[設定]、[アクセス キー]** の順に選択します。

![Blob Storage キーの検索](./media/find-blob-storage-keys.png)

ここでは 2 つのキーを使用して、Blob Storage アカウントへのアクセス権をコードに提供します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
"_myStorage_" が既存の Blob Storage アカウントを指すようにコード行を更新します。

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

Blob Storage アカウントを指すように "_myStorage_" が設定されると、ボット コードは Blob Storage からデータを保存および取得するようになります。

## <a name="start-your-bot"></a>ボットの起動
ボットをローカルで実行します。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[Open Bot]\(ボットを開く\)** リンクをクリックします。 
2. プロジェクトを作成したディレクトリにある .bot ファイルを選択します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り
メッセージをボットに送信します。ボットは受信したメッセージを表示します。
![実行中のエミュレーター](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>データの表示
ボットを実行して情報を保存したら、Azure portal の **[ストレージ エクスプローラー]** タブの下にその情報を表示できます。

## <a name="blob-transcript-storage"></a>Blob トランスクリプト ストレージ
Azure Blob トランスクリプト ストレージには、特殊なストレージ オプションが用意されています。このオプションを使用すると、記録されたトランスクリプトの形式でユーザーの会話を簡単に保存および取得できます。 Azure Blob トランスクリプト ストレージは、ボットのパフォーマンスをデバッグする際に、調べるユーザー入力を自動的にキャプチャする場合には特に便利です。

### <a name="set-up"></a>セットアップ
Azure Blob トランスクリプト ストレージでは、上記の「_Blob Storage アカウントの作成_」および「_構成情報の追加_」の手順に従って作成したのと同じ Blob Storage アカウントを使用できます。 ここでは、新しい BLOB コンテナーである "_mybottranscripts_" を追加しました。 

### <a name="implementation"></a>実装 
次のコードでは、トランスクリプト ストレージのポインターである "_transcriptStore_" を新しい Azure BLOB トランスクリプト ストレージ アカウントに接続します。 ここに示されているユーザーの会話を格納するソース コードは、[会話履歴](https://aka.ms/bot-history-sample-code)サンプルに基づいています。 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Azure Blob トランスクリプトへのユーザーの会話の保存
BLOB コンテナーでトランスクリプトを保存できるようになったら、ボットとユーザーの会話の保持を開始できます。 このような会話を後からデバッグ ツールとして使用しすると、ユーザーがボットとどのようにやり取りするかを確認できます。 次のコードは、activity.text が入力メッセージ _!history_ を受け取ったときにユーザー会話の入力を保持します。


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id, continuationToken);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>チャネルに対して保存されているすべてのトランスクリプトの検索
保存したデータの内容を確認するために、次のコードを使用して、保存したすべてのトランスクリプトの "_ConversationID_" をプログラムによって検索できます。 ボット エミュレーターを使用してコードをテストする場合に "_やり直す_" を選択すると、新しいトランスクリプトが新しい "_ConversationID_" で開始します。

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>Azure Blob トランスクリプト ストレージからのユーザーの会話の取得
ボット対話のトランスクリプトが Azure Blob Transcript Store に保存されたら、それらをプログラムによって取得し、AzureBlobTranscriptStorage メソッドである "_GetTranscriptActivities_" を使用してテストまたはデバッグできます。 次のコード スニペットでは、保存された各トランスクリプトから過去 24 時間以内に受信して保存されたユーザー入力のトランスクリプトをすべて取得します。


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>保存されたトランスクリプトの Azure Blob トランスクリプト ストレージからの削除
ユーザー入力データを使用したボットのテストまたはデバッグが完了したら、保存されたトランスクリプトをプログラムによって Azure Blob Transcript Store から削除できます。 次のコード スニペットでは、保存されたトランスクリプトをボットの Transcript Store からすべて削除します。


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


次に示すリンク先には、[Azure Blob トランスクリプト ストレージ](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)に関するその他の情報が記載されています。  

## <a name="next-steps"></a>次の手順
ストレージを直接読み書きする方法がわかったので、状態マネージャーを使ってそれを行う方法を確認してください。

> [!div class="nextstepaction"]
> [会話とユーザーのプロパティを使用して状態を保存する](bot-builder-howto-v4-state.md)

