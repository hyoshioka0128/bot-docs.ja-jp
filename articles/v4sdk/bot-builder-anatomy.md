---
title: ボットの構造 | Microsoft Docs
description: ボットのさまざまなパーツとそのしくみについて詳しく説明します
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928290"
---
# <a name="anatomy-of-a-bot"></a>ボットの構造

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

最も[基本的な意味](bot-builder-basics.md)では、ボットとは、ユーザーがメッセージを使ってやり取りする会話型アプリです。 ボットはそれぞれの言語で Web アプリの基本構造に従っており、Bot Framework SDK を使用して、その構造に基づいて構築されます。

ボットはさまざまなコンポーネントで構成され、これによりユーザーが[メッセージを送信](./bot-builder-howto-send-messages.md)したり、[状態](./bot-builder-storage-concept.md)を追跡したりできます。 ボットとのコミュニケーションには[エミュレーター](../bot-service-debug-emulator.md)や [WebChat](../bot-service-manage-test-webchat.md) が使用されます。また、運用環境では、いずれかの[チャネル](bot-concepts.md)を使用できます。 

ここでは、基本的なエコー ボットについて順を追って説明しながら、ボットが機能するよう連携する各パーツを詳しく確認していきます。

## <a name="files-created-for-echo-bot"></a>エコー ボット用に作成されるファイル

プログラミング言語ごとに、異なるファイルがエコー ボット用に作成されます。この例ではエコー ボットに **BasicEcho** という名前が付けられ、これらのファイルは、システム セクション、ヘルパーの項目、ボットのコアの 3 つのセクションに分類されています。 以下の表は、C# と JavaScript の両方の主要ファイルを示しています。

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

ここでは、これらのファイルの主な機能をセクションごとに説明します。 これらのファイルの中には独自のインクルード セットを持つものがあります。ボット固有のプログラミング ライブラリと一般的なプログラミング ライブラリです。 これらのインクルードには、必要な関数セットと、ご自身のアプリケーションで必要になる可能性がある関数がいくつか含まれています。 これらのインクルードについては詳しく説明しませんが、興味がある方は、Visual Studio を使用して、その定義やどの名前空間に含まれるかをご確認いただけます。

## <a name="system-section"></a>システム セクション

システム セクションには、ご自身のボットを標準アプリケーションとして機能させるうえで必要なものが含まれます。 たとえば、構成ファイル、アダプター、JSON ファイルです。 これらの項目は、特定の情報を必要とするいくつかの構成ファイルを除き、そのままにしておくことができます (多くの場合、その必要があります)。

# <a name="ctabcs"></a>[C#](#tab/cs)

ボットは、一種の [ASP.NET Core Web](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1) アプリケーション フレームワークです。 [ASP.NET の基礎](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x)に関する記事を確認すると、以下で説明する appsettings.json、Program.cs、Startup.cs などのファイルにあるのと同様のコードが見られます。 これらのファイルはすべての Web アプリに必須で、ボット固有のものではありません。 これら一部のファイルのコードは、ここにはコピーしませんが、ボットを実行する際に確認できます。

### <a name="appsettingsjson"></a>appsettings.json

このファイルには、お使いのボットの設定情報、主にアプリ ID とパスワードが基本的な JSON 形式で含まれているだけです。  [エミュレーター](../bot-service-debug-emulator.md)でテストする場合、これらの情報は不要なので空白のままにできますが、運用環境では必要になります。

### <a name="programcs"></a>Program.cs

このファイルは、ご自身の ASP.NET Web アプリを動作させるために必要です。実行するには `Startup` クラスを指定します。

### <a name="startupcs"></a>Startup.cs

**Startup.cs** には興味深い他のパーツが含まれていますが、これについては後ほど触れます。ここでは、このファイルのシステム セクションについて説明します。

`Startup` のコンストラクターでは、アプリの設定と環境変数が指定され、お使いの Web アプリの構成が作成されます。

`Configure` メソッドでは、ご自身のアプリで Bot Framework とその他のいくつかのファイルが使用されることを指定します。これにより、アプリの構成が完了します。 Bot Framework を使用するすべてのボットで、その構成の呼び出しが必要です。

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();

            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json および readme.md

**launchSettings.json** には、Web アプリ用の設定構成がいくつか含まれるだけです。**readme.md** は、エコー ボットに関する簡単な 1 行の説明です。 これらのファイルの内容は、このボットを理解するうえで重要ではありません。 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

システム セクションには、主に **package.json**、**.env** ファイル、**app.js**、および **README.md** が含まれます。 一部のファイルのコードは、ここにはコピーしませんが、ボットを実行する際に確認できます。

### <a name="packagejson"></a>package.json

**package.json** では、お使いのボットの依存関係とそれに関連付けられているバージョンが指定されます。 これはすべて、テンプレートとご自身のシステムによって設定されます。

### <a name="env-file"></a>.env ファイル

**.env** ファイルでは、ポート番号、アプリ ID、パスワードなど、お使いのボットの構成情報が指定されます。 特定のテクノロジを使用している場合、または運用環境でこのボットを使用している場合は、ご自身の特定のキーまたは URL をこの構成に追加する必要があります。 ただし、このエコー ボットについては、今のところ、ここでは何も行う必要はありません。アプリ ID およびパスワードは、現時点では未定義のままにしておくことができます。 

**.env** 構成ファイルを使用するには、テンプレートに、追加のパッケージが含まれている必要があります。  最初に、`dotenv` パッケージを npm から取得します。

`npm install dotenv`

次に、以下の行を、他の必須ライブラリと共にご自身のボットに追加します。

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

`app.js` ファイルの上部には、サーバーとアダプターが含まれます。このサーバーとアダプターにより、お使いのボットがユーザーとやり取りして、応答を送信できます。 サーバーは、**.env** 構成で指定したポートでリッスンするか、エミュレーターとの接続用の 3978 にフォールバックします。 アダプターはご自身のボットのコンダクタとして機能し、送受信の通信、認証などを指示します。 

```javascript
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
``` 

### <a name="readmemd"></a>README.md

**README.md** は、ボットの基本構造、"*ダイアログ*" の機能など、ボット構築の側面の一部に関する有用な情報を提供しますが、ご自身のボットを理解するうえで重要ではありません。

---


## <a name="helper-items"></a>ヘルパーの項目

すべてのプログラムと同様、ヘルパー メソッドと他の定義を使用することによりコードを簡略化できます。 `.bot` ファイルなど、このカテゴリに分類されるボットにとって重要なボット パーツがいくつかますが、これは必ずしもボットのコア ロジックを構成するパーツではありません。

### <a name="bot-file"></a>.bot ファイル

`.bot` ファイルには、エンドポイント、アプリ ID、パスワードなどの情報が含まれます。この情報により、ご自身のボットに[チャネル](bot-concepts.md)が接続できます。 このファイルは、テンプレートからボットの作成を開始すると、自動的に作成されますが、エミュレーターまたはその他のツールを使用して、独自のものを作成することもできます。

ご自身のファイルの内容は、ご自身のボットに *BasicEcho* 以外の名前を付けていない限り、ここの内容と一致する必要があります。 この情報は、ご自身のボットをどのように使用するかに従って、変更および更新しなければならない場合がありますが、このエコー ボットの実行では、今のところ変更は不要です。

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

**EchoState.cs** には、現在の状態を維持するために、ご自身のボットによって使用されるシンプルなクラスが含まれます。 これには、お使いのターン カウンターのインクリメントに使用する `int` のみが含まれます。 状態については、次のセクションでさらに詳しく説明します。ここで理解する必要があるのは、`EchoState` がターン アカウントを含むクラスであるということだけです。

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

**default.htm** は、ご自身のボットを実行したときに表示される Web ページで、`html` で記述されています。 お使いのボットに接続するための有用な情報や、ボットの操作方法が表示されますが、ボットの動作が、そのページのコンテンツに影響されることはありません。 ボットを実行すると、コードがポップアップ表示されます。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>必要なライブラリ

`app.js` ファイルの最上部に、必要な一連のモジュールまたはライブラリが表示されます。 これらのモジュールにより、お使いのアプリケーションに含める必要がある関数セットにアクセスできるようになります。 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>ボットのコア

ようやく、本当に興味ある内容にたどりつきました。 ボットのコアでは、ボットがユーザーと対話する方法が定義され、ミドルウェアとボット ロジックが含まれます。

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>ミドルウェア

**Startup.cs** 内には、`ConfigureServices` と呼ばれる 1 つのメソッドがあります。 このメソッドにより、ご自身の資格情報プロバイダーが設定され、ミドルウェアが追加されます。

ここで追加される[ミドルウェア](bot-builder-concept-middleware.md)では、2 つ処理が実行されます。 1 つ目は例外処理で、これによりお使いのボットが失敗時に正常に終了されます。 ラムダ式としてインラインで定義され、単にターミナルに例外を出力し、問題が発生していることをユーザーに伝えます。

2 番目のミドルウェアによってご自身の状態が維持されます。直前に定義された `MemoryStorage` が使用されます。 この状態は `ConversationState` として定義されています。これは、単にご自身の会話の状態が保持されていることを意味します。 ご自身のターン カウンターで定義したクラス `EchoState` が、関心のある情報を保存するために使用されます。

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>ボット ロジック

主なボット ロジックは、ボットの `OnTurn` ハンドラー内で定義されます。 `OnTurn` には引数として[コンテキスト](./bot-builder-concept-activity-processing.md#turn-context)変数が指定され、受信[アクティビティ](bot-builder-concept-activity-processing.md)、会話などに関する情報が提供されます。

受信アクティビティには[さまざまな種類](../bot-service-activities-entities.md#activity-types)があるため、まず、ご自身のボットがメッセージを受信したかどうかを確認します。 それがメッセージの場合は、ご自身のボットの会話に関する現在の情報を保持する状態変数を作成します。 その後、カウントと送信メッセージをユーザーにエコーバックする前に、お使いのターン カウントを追跡する `int` をインクリメントします。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
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
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>ミドルウェア

**app.js** では、ご自身の状態を維持するために、このボット内で[ミドルウェア](bot-builder-concept-middleware.md)を使用します。その際 `botbuilder` の上部で必須モジュールの 1 つとして定義した `MemoryStorage` が使用されます。 この状態は `ConversationState` として定義されています。これは、単にご自身の会話の状態が保持されていることを意味します。 `ConversationState` には、ご自身が興味のある情報が保存されます。この場合は、単なるメモリ内のターン カウンターです。

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>ボット ロジック

*processActivity* 内の 3 番目のパラメーターは、受信された[アクティビティ](bot-builder-concept-activity-processing.md)がアダプターによって事前処理され、任意のミドルウェアを介してルーティングされた後、ボットのロジックを実行するために呼び出される関数ハンドラーです。 [コンテキスト](./bot-builder-concept-activity-processing.md#turn-context)変数は引数として関数ハンドラーに渡され、受信アクティビティ、送信者と受信者、チャネル、会話などに関する情報を提供する目的で使用できます。

最初に、ボットがメッセージを受信したかどうかを確認します。 ボットがメッセージを受信しなかった場合は、受信したアクティビティの種類がエコー バックされます。 次に、ボットの会話に関する情報が保持される状態変数を作成します。 その後、未定義の場合はカウント変数を 0 に設定します (ボットを開始すると、これが発生します)。または、新しいメッセージごとにこれをインクリメントします。 最後に、カウントと送信メッセージをユーザーにエコー バックします。

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---
