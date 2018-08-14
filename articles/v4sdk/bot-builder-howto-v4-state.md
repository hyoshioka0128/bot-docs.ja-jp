---
title: 会話とユーザーのプロパティを使用して状態を保存する | Microsoft Docs
description: Bot Builder SDK for .NET の V4 を使用してデータを保存および取得する方法について説明します。
keywords: 会話状態, ユーザー状態, 状態ミドルウェア, 会話フロー, File Storage, Azure Table Storage
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16df371b1cabb4b3eb47d1f491a5d45e26627d38
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352851"
---
# <a name="save-state-using-conversation-and-user-properties"></a>会話とユーザーのプロパティを使用して状態を保存する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ボットで会話とユーザーの状態を保存するには、最初に状態マネージャー ミドルウェアを初期化した後、会話とユーザーの状態プロパティを使用します。
状態がどのように使われるかの詳細については、[状態とストレージ](./bot-builder-storage-concept.md)に関するページをご覧ください。

## <a name="initialize-state-manager-middleware"></a>状態マネージャー ミドルウェアを初期化する

SDK で会話またはユーザーのプロパティ ストアを使用するには、その前に、状態マネージャー ミドルウェアを使うようにボット アダプターを初期化する必要があります。 "_会話状態_" は会話のプロパティに使用され、"_ユーザー状態_" はユーザーのプロパティに使用されます。 (ユーザー状態プロパティには、複数の会話でアクセスできます。)状態マネージャー ミドルウェアが提供する抽象化により、基になっているストレージの種類とは関係なく、単純なキーと値またはオブジェクト ストアを使用してプロパティにアクセスできます。 基になるストレージの種類がメモリ内、ファイル ストレージ、Azure Table Storage のいずれであっても、状態マネージャーがストレージへのデータの書き込みと同時実行の管理の面倒を見ます。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConversationState` の初期化方法については、Microsoft.Bot.Samples.EchoBot-AspNetCore サンプルの `Startup.cs` をご覧ください。

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

行 `options.Middleware.Add(new ConversationState<EchoState>(dataStore));` で、`ConversationState` は会話状態マネージャー オブジェクトであり、ミドルウェアとしてボットに追加されます。 `EchoState` 型パラメーターは、会話状態情報の格納方法を表す型です。 ボットは、会話またはユーザーの状態データに対して任意のクラス型を使用できます。

`EchoState` の実装は `EchoBot.cs` 内にあります。
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

ストレージ プロバイダーを定義し、使用する状態マネージャーにそれを割り当てます。
`ConversationState` と `UserState` の両方の管理ミドルウェアに同じストレージ プロバイダーを使用できます。
その後、`BotStateSet` ライブラリを使用して、データの保持を管理するミドルウェア レイヤーにそれを接続します。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> メモリ内データ ストレージはテスト専用です。 このストレージは揮発性であり、一時的なものです。 ボットの再起動ごとにデータがクリアされます。 他の基になるストレージ メディアを会話状態とユーザー状態用に設定する方法については、後の「[File Storage](#file-storage)」および「[Azure Table Storage](#azure-table-storage)」をご覧ください。 

### <a name="configuring-state-manager-middleware"></a>状態マネージャー ミドルウェアの構成

状態ミドルウェアを初期化するときに、省略可能な "_状態設定_" パラメーターを使用して、プロパティの保存方法の既定の動作を変更できます。 既定の設定は次のとおりです。

* ターン コンテキストの有効期間を超えてプロパティを保持します。
* ボットの複数のインスタンスがプロパティに書き込む場合、ボットの最後のインスタンスによる前のインスタンスの上書きを許可します。

## <a name="use-conversation-and-user-state-properties"></a>会話とユーザーの状態プロパティを使用する 
<!-- middleware and message context properties -->

状態マネージャー ミドルウェアの構成が済むと、コンテキスト オブジェクトから会話状態とユーザー状態のプロパティを取得できます。
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot Builder SDK の `Microsoft.Bot.Samples.EchoBot` サンプルを使用して、処理方法を確認できます。 

`OnTurn` ハンドラーでは、定義されているデータにアクセスするための会話状態を `context.GetConversationState` が取得し、ユーザーはプロパティを使用して状態を変更することができます (この場合は、`TurnNumber` の増分)。

```csharp
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Yeoman ジェネレーター サンプルから生成された EchoBot を使用して、この動作を確認できます。

このコード サンプルでは、会話状態にターン カウンターを格納する方法が示されています。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

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

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>会話状態を使用して会話フローを制御する

会話フローの設計では、状態フラグを定義して会話フローを制御すると便利です。 フラグは、単純な**ブール**型でも、現在のトピックの名前を含む型でもかまいません。 フラグは、会話内での現在位置を追跡するのに役立ちます。 たとえば、**ブール**型フラグはユーザーが会話内にいるかどうかを示すことができます。 一方、トピック名のプロパティは、ユーザーが現在どの会話にいるのかを示すことができます。

次の例では、ブール型の _have asked name_ プロパティを使用して、ボットがユーザーに名前を要求したときにフラグを設定します。 次のメッセージを受信すると、ボットはプロパティを確認します。 プロパティが `true` に設定されている場合、ボットはユーザーが名前を要求されたばかりであることを知り、受信メッセージをユーザー プロパティとして保存する名前と解釈します。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

`UserState<UserInfo>.Get(context)` によって返すことができるようにユーザー状態を設定するには、ユーザー状態ミドルウェアを追加します。 たとえば、ASP .NET Core EchoBot の `Startup.cs` で、ConfigureServices.cs のコードを変更します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

または、ダイアログの "_ウォーターフォール_" モデルを使用します。 ダイアログは会話の状態を自動的に追跡するので、ユーザーは状態を追跡するためのフラグを作成する必要はありません。 詳細については、「[ダイアログで会話フローを管理する](bot-builder-dialog-manage-conversation-flow.md)」を参照してください。

## <a name="file-storage"></a>File Storage

メモリ ストレージ プロバイダーは、ボットが再起動されると破棄されるメモリ内ストレージを使用します。 これは、テストが目的の場合にのみ適しています。 データを保持する必要はあるが、データベースにボットをフックしたくない場合は、File Storage プロバイダーを使用できます。 このプロバイダーもテストを目的としたものですが、状態データをユーザーが検査できるようにファイルに保持します。 データは JSON 形式を使ってファイルに書き込まれます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Microsoft.Bot.Samples.EchoBot-AspNetCore サンプルの `Startup.cs` に移動し、`ConfigureServices` メソッドのコードを編集します。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

コードを実行し、echobot に入力を何回かエコーバックさせます。

その後、`System.IO.Path.GetTempPath()` で指定されているディレクトリに移動します。 名前が "conversation" で始まるファイルが見つかるはずです。 それを開いて JSON を確認します。 ファイルには次のような内容が含まれます。
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

`$type` は、会話状態を格納するためにボットで使用しているデータ構造の型です。 `TurnNumber` フィールドは、`EchoState` クラスの `TurnNumber` プロパティに対応します。 `eTag` フィールドは `IStoreItem` から継承され、ボットが会話の状態を更新するたびに自動的に更新される一意の値です。  eTag フィールドにより、ボットはオプティミスティック同時実行制御を有効にできます。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`FileStorage` を使用するには、前の「[会話とユーザーの状態プロパティを使用する](#use-conversation-and-user-state-properties)」セクションで説明されているエコー ボットのサンプルを更新します。 `storage` を `MemoryStorage` ではなく `FileStorage` に設定し、botbuilder からの `FileStorage` が必要です。 必要な変更はこれだけです。 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

`FileStorage` プロバイダーは、パラメーターとして "パス" を受け取ります。 パスを指定することで、ユーザーはボットからの情報が格納されたファイルを簡単に見つけることができます。 "*会話*" ごとに新しいファイルが作成されます。 そのため、"*パス*" では名前が `conversation!` で始まる複数のファイルが見つかることがあります。 日付で並べ替えることにより、最新の会話を簡単に検索できます。 一方、"*ユーザー*" 状態については見つかるファイルは 1 つだけです。 ファイル名は `user!` で始まっています。 これらのオブジェクトのいずれかの状態が変化すると常に、状態マネージャーは変更を反映するようにファイルを更新します。

ボットを実行し、いくつかのメッセージを送信します。 次に、ストレージ ファイルを探して開きます。 ターン カウンターを追跡しているエコー ボットの JSON の内容の例を次に示します。

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Azure テーブル ストレージ

Azure Table Storage もストレージ メディアとして使用できます。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Microsoft.Bot.Samples.EchoBot-AspNetCore サンプルで、`Microsoft.Bot.Builder.Azure` NuGet パッケージへの参照を追加します。

次に、`Startup.cs` に移動し、`using Microsoft.Bot.Builder.Azure;` ステートメントを追加して、`ConfigureServices` メソッドのコードを編集します。
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true` は、[Azure ストレージ エミュレーター][AzureStorageEmulator]で使用できる接続文字列です。 エミュレーターを使用していない場合は、独自の接続文字列に置き換えます。

コンストラクターで `AzureTableStorage` に対して指定した名前を持つテーブルが存在しない場合は、作成されます。

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

echobot サンプルの `app.js` では、`AzureTableStorage` を使用して ConversationState を作成できます。 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true` は、[Azure ストレージ エミュレーター][AzureStorageEmulator]で使用できる接続文字列です。
コンストラクターで `AzureTableStorage` に対して指定した名前を持つテーブルが存在しない場合は、作成されます。

---

保存されている会話状態データを見るには、サンプルを実行し、[Azure Storage Explorer][AzureStorageExplorer] を使用してテーブルを開きます。

![Azure Storage Explorer での echobot 会話状態データ](media/how-to-state/echostate-azure-storage-explorer.png)


**[パーティション キー]** は、現在の会話に固有の一意に生成されるキーです。 ボットを再起動するか、新しい会話を開始すると、新しい会話の行に独自のパーティション キーが表示されます。 `EchoState` データ構造は JSON にシリアル化されて、Azure Table の **Json** 列に保存されます。 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

BotBuilder SDK によって、`$type` フィールドと `eTag` フィールドが追加されます。 eTag の詳細については、[オプティミスティック同時実行制御の管理](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)に関するページをご覧ください。


## <a name="next-steps"></a>次の手順

状態を使用してストレージのボット データを読み書きする方法がわかったので、ストレージを直接読み書きする方法を確認してください。

> [!div class="nextstepaction"]
> [ストレージに直接書き込む方法](bot-builder-howto-v4-storage.md)。

## <a name="additional-resources"></a>その他のリソース
ストレージの背景の詳細については、[Bot Builder SDK でのストレージ](bot-builder-storage-concept.md)に関するページをご覧ください

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
