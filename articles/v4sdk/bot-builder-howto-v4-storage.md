---
title: ストレージに直接書き込む - Bot Service
description: Bot Framework SDK for .NET でストレージに直接書き込む方法について説明します。
keywords: ストレージ, 読み取りと書き込み, メモリ ストレージ, eTag
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 44f3e532459529c02d4ce5ea06d762f28ab8796a
ms.sourcegitcommit: 2412f96ad8f74dfa615c71f566c5befffb920658
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/25/2020
ms.locfileid: "82158849"
---
# <a name="write-directly-to-storage"></a>ストレージに直接書き込む

[!INCLUDE[applies-to](../includes/applies-to.md)]

ミドルウェアまたはコンテキスト オブジェクトを使用せずに、ストレージ オブジェクトに対して直接読み取りや書き込みを行うことができます。 ボットが会話の保持に使用するデータ、またはボットの会話フロー外にあるソースのデータについては、この方法が適切な可能性があります。 このデータ ストレージ モデルでは、データは、状態マネージャーを使用せずに、ストレージから直接データを読み取られます。 この記事のコード例では、**メモリ ストレージ**、**Cosmos DB**、**Blob Storage**、**Azure Table Storage**、**Azure Blob Transcript Store** を使用してストレージに対するデータの読み取りや書き込みを行う方法を示します。

## <a name="prerequisites"></a>前提条件

- Azure サブスクリプションをお持ちでない場合は、開始する前に[無料](https://azure.microsoft.com/free/)アカウントを作成してください。
- 記事に関する知識: [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart)、[nodeJS](https://aka.ms/bot-framework-www-node-js-quickstart)、または [Python](https://aka.ms/bot-framework-www-node-python-quickstart) 用にボットをローカルで作成します。
- [C# テンプレート](https://aka.ms/bot-vsix)、[nodeJS](https://nodejs.org) および [yeoman](http://yeoman.io) 用の Bot Framework SDK v4 テンプレート。

## <a name="about-this-sample"></a>このサンプルについて

この記事のサンプル コードは、基本的なエコー ボットの構造で始まり、コードを追加することでそのボットの機能を拡張します (下記を参照)。 拡張されたこのコードにより、受信したユーザー入力が保持されるリストが作成されます。 ユーザー入力の完全なリストは、ターンごとにユーザーにエコー バックされます。 この入力リストを含むデータ構造は、その後、そのターンの最後にストレージに保存されます。 このサンプル コードには追加機能が追加されているため、さまざまな種類のストレージについて説明します。

## <a name="memory-storage"></a>メモリ ストレージ

Bot Framework SDK を使用すると、メモリ内ストレージを使用してユーザー入力を格納することができます。 メモリ ストレージはテストにのみ使用され、実稼働を目的としたものではありません。 メモリ内ストレージは揮発性で一時的なものです。ボットの再起動ごとにデータがクリアされます。 データベース ストレージなどの永続的なストレージ類は運用環境のボットに最適です。 ボットを発行する前に、ストレージを **Cosmos DB**、**Blob Storage**、[**Azure Table Storage**](~/nodejs/bot-builder-nodejs-state-azure-table-storage.md) のいずれかに設定してください。

## <a name="build-a-basic-bot"></a>基本のボットを作成する

このトピックの残りの部分では、エコー ボットについては取り上げません。 エコー ボットのサンプル コードをローカルで構築するには、[C# EchoBot](https://aka.ms/bot-framework-www-c-sharp-quickstart)、[JS EchoBot](https://aka.ms/bot-framework-www-node-js-quickstart)、または [Python EchoBot](https://aka.ms/bot-framework-www-node-python-quickstart) のいずれかを構築するためのクイック スタート手順に従ってください。

### <a name="c"></a>[C#](#tab/csharp)

**EchoBot.cs**

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
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

   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
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
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

.env 構成ファイルを使用するには、お使いのボットに追加のパッケージが含まれている必要があります。 dotnet パッケージを npm から取得します (インストールされていない場合)。

```powershell
npm install --save dotenv
```

**bot.js**

```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)');
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`);
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)');
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
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
                await turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                await turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            await turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                await turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        await turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```

### <a name="python"></a>[Python](#tab/python)

**bot.py**

```py
from botbuilder.core import ActivityHandler, TurnContext, StoreItem, MemoryStorage


class UtteranceLog(StoreItem):
    """
    Class for storing a log of utterances (text of messages) as a list.
    """

    def __init__(self):
        super(UtteranceLog, self).__init__()
        self.utterance_list = []
        self.turn_number = 0
        self.e_tag = "*"


class MyBot(ActivityHandler):
    """
    Represents a bot saves and echoes back user input.
    """

    def __init__(self):
        self.storage = MemoryStorage()

    async def on_message_activity(self, turn_context: TurnContext):
        utterance = turn_context.activity.text

        # read the state object
        store_items = await self.storage.read(["UtteranceLog"])

        if "UtteranceLog" not in store_items:
            # add the utterance to a new state object.
            utterance_log = UtteranceLog()
            utterance_log.utterance_list.append(utterance)
            utterance_log.turn_number = 1
        else:
            # add new message to list of messages existing state object.
            utterance_log: UtteranceLog = store_items["UtteranceLog"]
            utterance_log.utterance_list.append(utterance)
            utterance_log.turn_number = utterance_log.turn_number + 1

        # Show user list of utterances.
        await turn_context.send_activity(f"{utterance_log.turn_number}: "
                                         f"The list is now: {','.join(utterance_log.utterance_list)}")

        try:
            # Save the user message to your Storage.
            changes = {"UtteranceLog": utterance_log}
            await self.storage.write(changes)
        except Exception as exception:
            # Inform the user an error occurred.
            await turn_context.send_activity("Sorry, something went wrong storing your message!")
```

---

### <a name="start-your-bot"></a>ボットの起動

ボットをローカルで実行します。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

Bot Framework [Emulator](https://aka.ms/bot-framework-emulator-readme) をインストールしてから、エミュレーターを起動し、そのエミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。
2. ボットを起動したときに表示される Web ページの情報を踏まえ、ご自身のボットに接続するためのフィールドに入力します。

### <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをお使いのボットに送信します。 ボットには、受信したメッセージが一覧表示されます。

![エミュレーター テスト ストレージ ボット](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>Cosmos DB の使用

>[!IMPORTANT]
> _Cosmos DB ストレージ_ クラスは非推奨となりました。 "_Cosmos DB ストレージ_" で作成されたコンテナーは、`compatibilityMode` [フラグ](https://aka.ms/azure-dotnet-cosmosdb-partitionedstorage#L289) を追加して "_Cosmos DB パーティション分割ストレージ_" と共に使用できます。 詳細については、[Azure Cosmos DB でのパーティション分割](https://aka.ms/azure-cosmosdb-partitioning-overview)に関するページを参照してください。

メモリ ストレージを使用しているので、Azure Cosmos DB を使用するようにコードを更新します。 Cosmos DB は、Microsoft のグローバル分散型マルチモデル データベースです。 Azure Cosmos DB では、Azure のリージョンをいくつでもまたいでスループットとストレージを柔軟かつ個別にスケーリングすることができます。 このサービスは包括的なサービス レベル アグリーメント (SLA) により、スループット、待機時間、可用性、一貫性が保証されています。

### <a name="set-up"></a>設定

ボットで Cosmos DB を使用するには、コードに取り組む前に、データベース リソースを作成する必要があります。 Cosmos DB のデータベースとアプリ作成の詳細については、こちらの [Cosmos DB dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) または [Cosmos DB nodejs](https://aka.ms/Bot-framework-create-nodejs-cosmosdb) に関するドキュメントを参照してください。

### <a name="create-your-database-account"></a>データベース アカウントの作成

1. 新しいブラウザー ウィンドウで、[Azure Portal](https://portal.azure.com) にサインインします。

    ![Cosmos DB データベース アカウントの作成](./media/create-cosmosdb-database.png)

2. **[リソースの作成]、[データベース]、[Azure Cosmos DB]** の順にクリックします。

    ![Cosmos DB の[新規アカウント] ページ](./media/cosmosdb-new-account-page.png)

3. **[新規アカウント] ページ**で、 **[サブスクリプション]** および **[リソース グループ]** に情報を指定します。 **[アカウント名]** フィールドに一意の名前を指定します。この名前は、最終的にはご自身のデータ アクセス URL の一部になります。 **API** には **Core(SQL)** を選択し、データ アクセス時間を短縮するために近くの**場所**を指定します。
4. 次に、 **[確認および作成]** をクリックします。
5. 検証が適切に行われたら、 **[作成]** をクリックします。

アカウントの作成には数分かかります。 ポータルに " Azure Cosmos DB アカウントが作成されました" ページが表示されるまで待機します。

### <a name="add-a-database"></a>データベースを追加する

1. 新しく作成した Cosmos DB アカウント内の **[データ エクスプローラー]** ページに移動し、 **[コンテナーの作成]** ボタンの横にあるドロップダウン ボックスから **[データベースの作成]** を選択します。 ウィンドウの右側にパネルが表示され、新しいデータベースの詳細を入力することができます。

    ![Cosmos DB](./media/create-cosmosdb-database-resource.png)

2. 新しいデータベースの ID を入力し、必要に応じてスループットを設定し (これは後で変更できます)、最後に **[OK]** をクリックしてデータベースを作成します。 このデータベース ID は、後でボットを構成するときに使用するためにメモしておいてください。

    ![Cosmos DB](./media/create-cosmosdb-database-resource-details.png)

3. Cosmos DB アカウントとデータベースを作成したので、新しいデータベースを bot に統合するために、いくつかの値をコピーする必要があります。  これらの値を取得するには、Cosmos DB アカウントのデータベースの設定セクション内にある **[キー]** タブに移動します。  このページから、Cosmos DB エンドポイント (**URI**) と承認キー (**主キー**) が必要になります。

    ![Cosmos DB のキー](./media/comos-db-keys.png)

これで、データベースを含む Cosmos DB アカウントを作成し、bot を構成するための次の詳細情報が準備できているはずです。

- Cosmos DB エンドポイント
- 承認キー
- データベース ID

### <a name="add-configuration-information"></a>構成情報の追加

Cosmos DB ストレージを追加するための構成データは短く簡単です。  この記事の前の部分でメモした詳細情報を使用して、エンドポイント、承認キー、およびデータベース ID を設定します。  最後に、ボットの状態を格納するためにデータベース内に作成されるコンテナーに適切な名前を選択する必要があります。 次の例では、コンテナーは "bot-storage" と呼びます。

> [!NOTE]
> コンテナーを自分で作成しないでください。 ボットは、その内部の Cosmos DB クライアントを作成するときにそれを作成し、ボットの状態を格納するために正しく構成されていることを確認します。

### <a name="c"></a>[C#](#tab/csharp)

構成ファイルに以下の情報を追加します。

**appsettings.json**

```json
"CosmosDbEndpoint": "<your-cosmosdb-uri>",
"CosmosDbAuthKey": "<your-authorization-key>",
"CosmosDbDatabaseId": "<your-database-id>",
"CosmosDbContainerId": "<your-container-id>"
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

`.env` ファイルに以下の情報を追加します。

**.env**

```javascript
CosmosDbEndpoint="<your-cosmos-db-uri>"
CosmosDbAuthKey="<your-authorization-key>"
CosmosDbDatabaseId="<your-database-id>"
CosmosDbContainerId="<your-container-id>"
```

### <a name="python"></a>[Python](#tab/python)

構成ファイルに以下の情報を追加します。

**config.py**

```python
COSMOS_DB_ENDPOINT = "<your-cosmos-db-uri>"
COSMOS_DB_AUTH_KEY="<your-authorization-key>"
COSMOS_DB_DATABASE_ID="<your-database-id>"
COSMOS_DB_CONTAINER_ID="<your-container-id>"
```

---

#### <a name="installing-packages"></a>パッケージのインストール

Cosmos DB に必要なパッケージがあることを確認します。

### <a name="c"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

npm を使用して、botbuilder-azure への参照をご自身のプロジェクトに追加できます。
>**注**: この npm パッケージは、お使いの開発用マシンにインストールされている Python に依存します。 Python をまだインストールしていない場合、お使いのマシン用のインストール リソースが [python.org](https://www.python.org/downloads/) にあります。
```powershell
npm install --save botbuilder-azure
```

ご自身の `.env` ファイル設定にアクセスするために dotnet パッケージを npm から取得します (インストールされていない場合)。

```powershell
npm install --save dotenv
```

### <a name="python"></a>[Python](#tab/python)

pip を使用して、botbuilder-azure への参照をご自身のプロジェクトに追加できます。

```powershell
pip install botbuilder-azure
```

---

### <a name="implementation"></a>実装

> [!NOTE]
> バージョン 4.6 では、新しい Cosmos DB ストレージ プロバイダーである "_Cosmos DB パーティション分割ストレージ_" クラスが導入され、元の "_Cosmos DB ストレージ_" クラスは非推奨となりました。 "_Cosmos DB ストレージ_" で作成されたコンテナーは、`compatibilityMode` [フラグ](https://aka.ms/azure-dotnet-cosmosdb-partitionedstorage#L289) を追加して "_Cosmos DB パーティション分割ストレージ_" と共に使用できます。

### <a name="c"></a>[C#](#tab/csharp)

次のサンプル コードは、上記の[メモリ ストレージ](#memory-storage)のサンプルと同じボット コードを使用して実行されます。
次のコード スニペットは、ローカルのメモリ ストレージを置き換える "_myStorage_" に対する Cosmos DB ストレージの実装を示しています。 メモリ ストレージはコメント アウトされ、Cosmos DB への参照で置き換えられています。

**Startup.cs**

```csharp
using Microsoft.Bot.Builder.Azure;
```

`ConfigureServices` で、CosmosDB パーティション分割ストレージのストレージ インスタンスを作成します。

```csharp
// Use partitioned CosmosDB for storage, instead of in-memory storage.
services.AddSingleton<IStorage>(
    new CosmosDbPartitionedStorage(
        new CosmosDbPartitionedStorageOptions
        {
            CosmosDbEndpoint = Configuration.GetValue<string>("CosmosDbEndpoint"),
            AuthKey = Configuration.GetValue<string>("CosmosDbAuthKey"),
            DatabaseId = Configuration.GetValue<string>("CosmosDbDatabaseId"),
            ContainerId = Configuration.GetValue<string>("CosmosDbContainerId"),
            CompatibilityMode = false,
        }));
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

次のサンプル コードは[メモリ ストレージ](#memory-storage)と似ていますが、わずかに変更されています。

`botbuilder-azure` の `CosmosDbPartitionedStorage` が必要です。また、`.env` ファイルを読み取るように dotenv を構成します。

**bot.js**

```javascript
const { CosmosDbPartitionedStorage } = require('botbuilder-azure');
```

メモリ ストレージをコメント アウトし、Cosmos DB への参照で置き換えます。

**bot.js**

```javascript
...
const { CosmosDbPartitionedStorage } = require('botbuilder-azure');
...
const storage = new CosmosDbPartitionedStorage({
    cosmosDbEndpoint: process.env.CosmosDbEndpoint,
    authKey: process.env.CosmosDbAuthKey,
    databaseId: process.env.CosmosDbDatabaseId,
    containerId: process.env.CosmosDbContainerId,
    compatibilityMode: false
});
...
```

### <a name="python"></a>[Python](#tab/python)

次のサンプル コードは[メモリ ストレージ](#memory-storage)と似ていますが、わずかに変更されています。

`botbuilder-azure` からの `CosmosDbPartitionedStorage` および `CosmosDbPartitionedConfig` が必要で、CosmosDBStorage オブジェクトを作成します。

**bot.py**

```py
from botbuilder.azure import CosmosDbPartitionedStorage, CosmosDbPartitionedConfig
```

`__init__` でメモリ ストレージをコメント アウトし、Cosmos DB への参照に置き換えます。  上で使用したエンドポイント、認証キー、データベース ID、コンテナー ID を使用します。

**bot.py**

```py
def __init__(self):
    cosmos_config = CosmosDbPartitionedConfig(
        cosmos_db_endpoint=COSMOSDB_SERVICE_ENDPOINT,
        auth_key=COSMOSDB_KEY,
        database_id=COSMOSDB_DATABASE_ID,
        container_id=COSMOSDB_CONTAINER_ID,
        compatibility_mode = False
    )
    self.storage = CosmosDbPartitionedStorage(cosmos_config)
```

---

## <a name="start-your-bot"></a>ボットの起動

ボットをローカルで実行します。

## <a name="test-your-bot-with-bot-framework-emulator"></a>Bot Framework Emulator でのボットのテスト

Bot Framework Emulator を起動して、ご自身のボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。
2. ボットを起動したときに表示される Web ページの情報を踏まえ、ご自身のボットに接続するためのフィールドに入力します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信します。ボットは受信したメッセージを表示します。
![実行中のエミュレーター](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>データの表示

ボットを実行して情報を保存した後は、Azure Portal の **[データ エクスプローラー]** タブで、格納データを表示できます。

![データ エクスプローラーの例](./media/data_explorer.PNG)

## <a name="using-blob-storage"></a>Blob Storage の使用

Azure Blob Storage は、Microsoft のクラウド用オブジェクト ストレージ ソリューションです。 BLOB ストレージは、テキスト データやバイナリ データなどの大量の非構造化データを格納するために最適化されています。

### <a name="create-your-blob-storage-account"></a>Blob Storage アカウントの作成

ボットで Blob Storage を使用するには、コードに取り組む前に、いくつかの設定を行う必要があります。

1. 新しいブラウザー ウィンドウで、[Azure Portal](https://portal.azure.com) にサインインします。

    ![Blob Storage の作成](./media/create-blob-storage.png)

2. **[リソースの作成]、[Storage]、[ストレージ アカウント - Blob、File、Table、Queue]** の順にクリックします。

    ![Blob Storage の [新規アカウント] ページ](./media/blob-storage-new-account.png)

3. **[新規アカウント] ページ**で、ストレージ アカウントの **[名前]** を入力し、 **[アカウントの種類]** として **[Blob ストレージ]** を選択し、 **[場所]** 、 **[リソース グループ]** 、および **[サブスクリプション]** の情報を指定します。
4. 次に、 **[確認および作成]** をクリックします。
5. 検証が適切に行われたら、 **[作成]** をクリックします。

### <a name="create-blob-storage-container"></a>Blob Storage コンテナーの作成

ご自身の Blob storage アカウントが作成されたら、次の手順でこのアカウントを開きます

1. リソースを選択します。
2. Storage Explorer (プレビュー) を使用して "オープン" します

    ![Blob Storage コンテナーの作成](./media/create-blob-container.png)

3. [BLOB コンテナー] を右クリックし、"_BLOB コンテナーの作成_" を選択します。
4. メモを追加します。 Blob Storage アカウントにアクセスできるようにするには、"your-blob-storage-container-name" の値にこの名前を使用します。

#### <a name="add-configuration-information"></a>構成情報の追加

上記のボットの Blob Storage の構成に必要な Blob Storage キーを検索します。

1. Azure Portal で、Blob Storage アカウントを開き、 **[設定]、[アクセス キー]** の順に選択します。

    ![Blob Storage キーの検索](./media/find-blob-storage-keys.png)

Blob Storage アカウントにアクセスできるようにするには、"your-blob-storage-account-string" の値に key1 _Connection string_ を使用します。

#### <a name="installing-packages"></a>パッケージのインストール

Cosmos DB を使用するには、次のパッケージをインストールします (まだインストールされていない場合)。

### <a name="c"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

npm を使用して、botbuilder-azure への参照をご自身のプロジェクトに追加します。
>**注**: この npm パッケージは、お使いの開発用マシンにインストールされている Python に依存します。 Python をまだインストールしていない場合、お使いのマシン用のインストール リソースはこちらにあります: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure
```

ご自身の `.env` ファイル設定にアクセスするために dotnet パッケージを npm から取得します (インストールされていない場合)。

```powershell
npm install --save dotenv
```

### <a name="python"></a>[Python](#tab/python)

pip を使用して、botbuilder-azure への参照をご自身のプロジェクトに追加できます。

```powershell
pip install botbuilder-azure
```

---

### <a name="implementation"></a>実装

### <a name="c"></a>[C#](#tab/csharp)

**EchoBot.cs**

```csharp
using Microsoft.Bot.Builder.Azure;
```

"_myStorage_" が既存の Blob Storage アカウントを指すようにコード行を更新します。

**EchoBot.cs**

```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

`.env` ファイルに以下の情報を追加します。

**.env**

```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

ご自身の `bot.js` ファイルを次のように更新します。 `botbuilder-azure` の `BlobStorage` が必要です

**bot.js**

```javascript
const { BlobStorage } = require("botbuilder-azure");
```

コードを追加していない場合、ご自身の `.env` ファイルを読み込んで Cosmos DB ストレージを実装するには、ここでコードを追加します。

```javascript
// initialized to access values in .env file.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });
```

次に、"_ストレージ_" が既存の Blob Storage アカウントを指すように、ご自身のコードを更新します。それには、以前のストレージ定義をコメント アウトして、次を追加します。

**bot.js**

```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```

### <a name="python"></a>[Python](#tab/python)

次のサンプル コードは[メモリ ストレージ](#memory-storage)と似ていますが、わずかに変更されています。

`botbuilder-azure` からの `BlobStorage` が必要で、CosmosDBStorage オブジェクトを作成します。

**bot.py**

```py
from botbuilder.azure import BlobStorage, BlobStorageSettings
```

`__init__` でメモリ ストレージをコメント アウトし、Cosmos DB への参照に置き換えます。  上で使用したコンテナー名と接続文字列を使用します。

**bot.py**

```py
def __init__(self):
    blob_settings = BlobStorageSettings(
        container_name="<your_container_name>",
        connection_string="<your_connection_string>"
    )
    self.storage = BlobStorage(blob_settings)
```

---

ご自身の Blob Storage アカウントを指すようにストレージが設定されると、お使いのボット コードは Blob Storage からデータを保存および取得するようになります。

## <a name="start-your-bot"></a>ボットの起動

ボットをローカルで実行します。

## <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。
2. ボットを起動したときに表示される Web ページの情報を踏まえ、ご自身のボットに接続するためのフィールドに入力します。

## <a name="interact-with-your-bot"></a>ボットでのやり取り

メッセージをボットに送信すると、受信したメッセージがボットに一覧表示されます。

![エミュレーター テスト ストレージ ボット](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>データの表示

ボットを実行して情報を保存したら、Azure Portal の **[Storage Explorer]** タブの下にその情報を表示できます。

## <a name="blob-transcript-storage"></a>Blob トランスクリプト ストレージ

Azure Blob トランスクリプト ストレージには、特殊なストレージ オプションが用意されています。このオプションを使用すると、記録されたトランスクリプトの形式でユーザーの会話を簡単に保存および取得できます。 Azure Blob トランスクリプト ストレージは、ボットのパフォーマンスをデバッグする際に、調べるユーザー入力を自動的にキャプチャする場合に特に便利です。

**注: Javascript と Python では、現在、AzureBlobTranscriptStore はサポートされていません。次の説明は、C# のみを対象としています。**

### <a name="set-up"></a>設定

Azure Blob トランスクリプト ストレージでは、上記の「_Blob Storage アカウントの作成_」および「_構成情報の追加_」の手順に従って作成したのと同じ Blob Storage アカウントを使用できます。 ここで、トランスクリプトを保持するためのコンテナーを追加します

![トランスクリプト コンテナーの作成](./media/create-blob-transcript-container.png)

1. Azure Blob Storage アカウントを開きます。
1. "_Storage Explorer_" をクリックします。
1. "_BLOB コンテナー_" を右クリックし、"_BLOB コンテナーの作成_" を選択します。
1. トランスクリプト コンテナーの名前を入力し、 _[OK]_ を選択します (mybottranscripts を入力しました)。

### <a name="implementation"></a>実装

次のコードでは、トランスクリプト ストレージのポインター `_myTranscripts` を新しい Azure Blob トランスクリプト ストレージ アカウントに接続します。 新しいコンテナー名 <your-blob-transcript-container-name> でこのリンクを作成するには、トランスクリプト ファイルを保持する新しいコンテナーを Blob Storage に作成します。

**echoBot.cs**

```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...

   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");

   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Azure Blob トランスクリプトへのユーザーの会話の保存

BLOB コンテナーでトランスクリプトを保存できるようになったら、ボットとユーザーの会話の保持を開始できます。 このような会話を後からデバッグ ツールとして使用しすると、ユーザーがボットとどのようにやり取りするかを確認できます。 各エミュレーター "_会話の再開_" によって、新しいトランスクリプト会話リストの作成が開始されます。 次のコードでは、ユーザーの会話の入力を、保存されているトランスクリプト ファイルに保持します。

- 現在のトランスクリプトは `LogActivityAsync` を使用して保存されます。
- 保存されたトランスクリプトは `ListTranscriptsAsync` を使用して取得されます。
このサンプル コードでは、保存されている各トランスクリプトの ID が "storedTranscripts" という名前のリストに保存されます。 このリストは後で、保持する保存済み BLOB トランスクリプトの数を管理するときに使用します。

**echoBot.cs**

```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);

    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>保存済み BLOB トランスクリプトの管理

保存済みトランスクリプトをデバッグ ツールとして使用できると、保存済みトランスクリプトの数が時間経過と共に増加し、保持したい数を超えることがあります。 以下の追加コードでは、`DeleteTranscriptAsync` を使用して、取得された最新の 3 つのトランスクリプト項目を除くすべての項目を、BLOB トランスクリプト ストアから削除します。

**echoBot.cs**

```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);

    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

次に示すリンク先には、[Azure Blob トランスクリプト ストレージ](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)に関するその他の情報が記載されています

## <a name="additional-information"></a>追加情報

### <a name="manage-concurrency-using-etags"></a>eTag を使用してコンカレンシーを管理する
ボット コード例では、各 `IStoreItem` の `eTag` プロパティを `*` に設定します。 ストア オブジェクトの `eTag` (エンティティ タグ) メンバーは、コンカレンシーを管理するために Cosmos DB 内で使用されます。 `eTag` は、自分のボットが書き込んでいる同じストレージ内のオブジェクトをボットの別のインスタンスが変更した場合の対処方法をデータベースに通知します。

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最後の書き込みが有効 - 上書きを許可する
`eTag` プロパティの値にアスタリスク (`*`) を指定すると、最後の書き込みが有効であることを示します。 新しいデータ ストアを作成するときは、プロパティの `eTag` を `*` に設定して、以前に保存したことのないデータを書き込んでいること、または以前に保存されたすべてのプロパティを最後の書き込みで上書きすることを示します。 コンカレンシーがボットにとって問題にならない場合は、書き込んでいるすべてのデータについて `eTag` プロパティを `*` に設定して上書きを許可します。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>コンカレンシーを維持して上書きを禁止する
データを Cosmos DB に保存する際に、プロパティへの同時アクセスを禁止し、変更をボットの別のインスタンスによって上書きされないようにする場合は、`*` 以外の値を `eTag` に使います。 ボットが状態データを保存しようとして、`eTag` がストレージ内の `eTag` と同じ値でない場合、ボットは `etag conflict key=` というメッセージのエラー応答を受け取ります。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

既定では、Cosmos DB ストアはボットがストレージ オブジェクトを書き込むたびにその項目の `eTag` プロパティが等しいかどうかを確認し、各書き込みの後で新しい一意の値に更新します。 書き込みの `eTag` プロパティがストレージの `eTag` と一致しない場合は、別のボットまたはスレッドがデータを変更したことを意味します。

たとえば、保存されたメモをボットで編集しますが、ボットの別のインスタンスが行った変更を上書きしないようにする必要があるものとします。 ボットの別のインスタンスが編集を行った場合は、ユーザーに最新の更新でバージョンを編集させます。

### <a name="c"></a>[C#](#tab/csharp)

最初に、`IStoreItem` を実装するクラスを作成します。

**EchoBot.cs**

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

次に、ストレージ オブジェクトを作成することによって最初のメモを作成し、オブジェクトをストアに追加します。

**EchoBot.cs**

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

**EchoBot.cs**

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

### <a name="javascript"></a>[JavaScript](#tab/javascript)

データ ストアにサンプルのメモを書き込むヘルパー関数をボットの末尾に追加します。
最初に、`myNoteData` オブジェクトを作成します。

**bot.js**

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

**bot.js**

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
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

お使いのボットのロジック内からヘルパー関数にアクセスするには、次の呼び出しを追加します。

**bot.js**

```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

作成されたら、後でメモを取得して更新するために、ユーザーが「メモの更新」と入力したときにアクセス可能になるヘルパー関数を別に作成します。

**bot.js**

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
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

お使いのボットのロジック内からこのヘルパー関数にアクセスするには、次の呼び出しを追加します。

**bot.js**

```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

ご自身の変更の書き戻しを試みる前に、他のユーザーによってストア内のメモが更新されていると、`eTag` 値は一致しなくなり、`write` への呼び出しによって例外がスローされます。

### <a name="python"></a>[Python](#tab/python)

最初に、`StoreItem` を実装するクラスを作成します。

**bot.py**

```py
class Note(StoreItem):
    def __init__(self, name: str, contents: str, e_tag="*"):
        super(Note, self).__init__()
        self.name = name
        self.contents = contents
        self.e_tag = e_tag
```

次に、ストレージ オブジェクトを作成することによって最初のメモを作成し、オブジェクトをストアに追加します。

**bot.py**

```py
# create a note for the first time, with a non-null, non-* ETag.
changes = {"Note": Note(name="Shopping List", contents="eggs", e_tag="x")}

await self.storage.write(changes)
```

後でメモにアクセスして更新し、ストアから読み取ったその `eTag` を維持します。

**bot.py**

```py
store_items = await self.storage.read(["Note"])
    note = store_items["Note"]
    note.contents = note.contents + ", bread"

    changes = {"Note": note}
    await self.storage.write(changes)
```

変更を書き込む前にストアのメモが更新されていた場合、`write` の呼び出しでは例外がスローされます。

---

コンカレンシーを維持するには、常にストレージからプロパティを読み取った後、読み取ったプロパティを変更して、`eTag` が維持されるようにします。 ストアからユーザー データを読み取る場合、応答には eTag プロパティが含まれます。 データを変更して更新後のデータをストアに書き込む場合、要求に含まれる eTag プロパティでは、前に読み取ったのと同じ値が指定されている必要があります。 ただし、`eTag` を `*` に設定してオブジェクトを書き込むと、書き込みで他の変更を上書きできます。

## <a name="next-steps"></a>次のステップ

ストレージを直接読み書きする方法がわかったので、状態マネージャーを使ってそれを行う方法を確認してください。

> [!div class="nextstepaction"]
> [会話とユーザーのプロパティを使用して状態を保存する](bot-builder-howto-v4-state.md)
