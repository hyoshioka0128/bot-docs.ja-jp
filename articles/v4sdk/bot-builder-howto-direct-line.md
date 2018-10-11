---
title: Direct Line ボットおよびクライアントを作成する | Microsoft Docs
description: Bot Builder SDK for .NET V4 を使用して Direct Line ボットおよびクライアントを作成する方法について説明します。
keywords: direct line ボット, direct line クライアント, カスタム チャネル, コンソールベース, 発行
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dee0f9700fefede2a231ff2395e50ff17522806e
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707398"
---
# <a name="create-a-direct-line-bot-and-client"></a>Direct Line ボットおよびクライアントを作成する

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Microsoft Bot Framework Direct Line ボットは、独自の設計のカスタム クライアントで機能するボットです。 Direct Line ボットは通常のボットとよく似ています。 どちらも用意されているチャネルを使用する必要はありません。

Direct Line クライアントは、目的どおりに作成できます。 Android クライアント、iOS クライアント、またはコンソールベースのクライアント アプリケーションを作成できます。

このトピックでは、Direct Line ボットの作成および展開方法、およびコンソールベースの Direct Line クライアント アプリケーションの作成および実行方法について説明します。

## <a name="create-your-bot"></a>ボットを作成する

ボットの作成パスは、言語によって異なります。 JavaScript の場合は Azure にボットを作成してからコードを変更し、C# の場合はローカルでボットを作成してから Azure に発行しますが、いずれも有効な方法であり、どちらの言語でも使用できます。 ボットを Azure に発行する方法の詳細については、[Azure のボットを展開する方法](../bot-builder-howto-deploy-azure.md)のページを参照してください。

# <a name="ctabcscreatebot"></a>[C# を選択した場合](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>Visual Studio でソリューションを作成する

Visual Studio 2015 以降で Direct Line ボットのソリューションを作成するには:

1. **Visual C#** > **.NET Core** で新しい **ASP.NET Core Web アプリケーション**を作成します。

1. 名前には「**DirectLineBotSample**」と入力し、**[OK]** をクリックします。

1. **.NET Core** と **ASP.NET Core 2.0** が選択されていることを確認し、**空の**プロジェクト テンプレートを選択し、**[OK]** をクリックします。

#### <a name="add-dependencies"></a>依存関係を追加する

1. **[ソリューション エクスプローラー]** で **[依存関係]** を右クリックし、**[NuGet パッケージの管理]** を選択します。

1. **[参照]** をクリックし、**[プレリリースを含める]** チェック ボックスがオンになっていることを確認します。

1. 以下の NuGet パッケージを検索してインストールします。
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>appsettings.json ファイルを作成する

appsettings.json ファイルには、Microsoft アプリ ID、アプリ パスワード、およびデータ接続文字列が含まれています。 このボットは状態情報を保存しないので、データ接続文字列は空のままです。 また、Bot Framework Emulator のみを使用している場合は、これらすべてが空のままの可能性があります。

**appsettings.json** ファイルを作成するには:

1. **DirectLineBotSample** プロジェクトを右クリックし、**[追加]** > **[新しいアイテム]** の順に選択します。

1. **[ASP.NET Core]** で、**[ASP.NET 構成ファイル]** をクリックします。

1. 名前に「**appsettings.json**」と入力し、**[追加]** をクリックします。

appsettings.json ファイルの内容を次のように置き換えます。

```json
{
    "Logging": {
        "IncludeScopes": false,
        "Debug": {
            "LogLevel": {
                "Default": "Warning"
            }
        },
        "Console": {
            "LogLevel": {
                "Default": "Warning"
            }
        }
    },

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>Startup.cs を編集する

Startup.cs ファイルの内容を次のように置き換えます。

**注**: **DirectBot** は次の手順で定義されるので、赤い下線は無視しても構いません。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>DirectBot クラスを作成する

DirectBot クラスには、このボットのロジックの大部分が含まれています。

1. **DirectLineBotSample を右クリックし、** > **[追加]** > **[新しいアイテム]** の順に選択します。

1. **[ASP.NET Core]** で、**[クラス]** をクリックします。

1. 名前に「**DirectBot.cs**」と入力し、**[追加]** をクリックします。

DirectBot.cs ファイルの内容を次のように置き換えます。

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**注:** 既存の Program.cs ファイルを変更する必要はありません。

準備が完了したことを確認するために、**F6** キーを押してプロジェクトをビルドします。 警告やエラーは表示されないはずです。

### <a name="publish-the-bot-from-visual-studio"></a>Visual Studio からボットを発行する

1. Visual Studio のソリューション エクスプローラーで、**DirectLineBotSample** プロジェクトを右クリックし、**[スタートアップ プロジェクトに設定]** をクリックします。

    ![Visual Studio の [発行] ページ](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. **DirectLineBotSample** を右クリックし、**[発行]** をクリックします。 Visual Studio の **[発行]** ページが表示されます。

1. このボットの発行プロファイルが既にある場合は、それを選択し、**[発行]** ボタンをクリックして次のセクションに進みます。

1. 新しい発行プロファイルを作成するには、**[Microsoft Azure App Service]** アイコンを選択します。

1. **[新規作成]** を選択します。

1. **[発行]** ボタンをクリックします。 **[App Service の作成]** ダイアログが表示されます。

    ![[App Service の作成] ダイアログ ボックス](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - [アプリ名] には、後で見つけやすい名前を付けます。 たとえば、アプリ名の先頭に自分の電子メール アドレス名を追加することができます。

    - 正しいサブスクリプションを使用していることを確認します。

    - [リソース グループ] で、正しいリソース グループを使用していることを確認します。 リソース グループは、ボットの **[概要]** ブレードにあります。 **注:** リソース グループを間違えると、修正は困難です。

    - 正しい App Service プランを使用していることを確認します。
    
1. **[作成]** ボタンをクリックします。 Visual Studio でボットの展開が開始されます。

ボットが発行されると、ボットの URL エンドポイントがブラウザーに表示されます。

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>Microsoft Azure 上で Bot Channels Registration ボットを作成する

Direct Line ボットは任意のプラットフォームでホストできます。 この例では、ボットは Microsoft Azure でホストされます。 

Microsoft Azure でボットを作成するには:

1. Microsoft Azure portal で、**[リソースの作成]** をクリックし、次に「Bot Channels Registration」を検索します。

1. **Create** をクリックしてください。 [Bot Channels Registration] ブレードが表示されます。

    ![ボット名、サブスクリプション、リソース グループ、場所、価格レベル、メッセージング エンドポイントなどのフィールドがある [Bot Channels Registration] ブレード。](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. [Bot Channels Registration] ブレードで、**[ボット名]**、**[サブスクリプション]**、**[リソース グループ]**、**[場所]**、および **[価格レベル]** を入力します。

1. **[Messaging endpoint]\(メッセージング エンドポイント\)** は空白のままにします。 この値は後で入力します。

1. **[Microsoft App ID and password]\(Microsoft アプリ ID とパスワード\)**、**[Auto create App ID and password]\(アプリ ID とパスワードの自動作成\)** の順にクリックします。

1. **[ダッシュボードにピン留めする]** チェック ボックスをオンにします。

1. **[作成]** ボタンをクリックします。

展開には数分かかりますが、完了すると、ダッシュボードに表示されます。

### <a name="update-the-appsettingsjson-file"></a>appsettings.json ファイルを更新する

1. ダッシュボードのボットをクリックするか、**[すべてのリソース]** をクリックして Bot Channels Registration の名前を検索して新しいリソースに移動します。

1. **[概要]** をクリックします。

1. リソース グループ名を **DirectLineClientSample** プロジェクトの **Program.cs** の **botId** 文字列にコピーします。

1. **[設定]** をクリックし、**[Microsoft App ID]\(Microsoft アプリ ID\)** フィールドの近くにある **[管理]** をクリックします。

1. アプリケーション ID をコピーし、**appsettings.json** ファイルの **"MicrosoftAppId"** フィールドに貼り付けます。

1. **[新しいパスワードを生成]** ボタンをクリックします。

1. 新しいパスワードをコピーし、**appsettings.json** ファイルの **"MicrosoftAppPassword"** フィールドに貼り付けます。

### <a name="set-the-messaging-endpoint"></a>メッセージング エンドポイントを設定する

ボットにエンドポイントを追加するには:

1. ボットを発行した後にポップアップ表示されるブラウザーから URL アドレスをコピーします。

    ![メッセージング エンドポイントが強調表示された Bot Channels Registration の [設定] ブレード](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. ボットの **[設定]** ブレードを開きます。

1. **[Messaging endpoint]\(メッセージング エンドポイント\)** にアドレスを貼り付けます。

1. "https://" から始まり、"/api/messages" で終わるようにアドレスを編集します。 たとえば、ブラウザーからコピーしたアドレスが "http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net" の場合は、"https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages" に編集します。

1. [設定]ブレードの **[保存]** ボタンをクリックします。

**注:** 発行が失敗した場合は、Azure 上でボットを停止してから、ボットに変更を発行する必要があります。

1. ("https://" と ".azurewebsites.net" の間にある) アプリ サービス名を見つけます。

1. Azure portal の [すべてのリソース] でそのリソースを検索します。

1. App Service リソースをクリックします。 [App Service] ブレードが表示されます。

1. [App Service] ブレードの **[停止]** ボタンをクリックします。

1. Visual Studio でボットをもう一度発行します。

1. [App Service] ブレードの **[開始]** ボタンをクリックします。

### <a name="test-your-bot-in-webchat"></a>Web チャットでボットをテストする

ボットが動作することを検証するには、Web チャットでボットを確認します。

1. ボットの [Bot Channels Registration] ブレードで、**[設定]**、**[Test in Web Chat]\(Web チャットでのテスト\)** の順にクリックします。

1. 「Hi」(こんにちは) と入力します。 ボットはウェルカム メッセージで応答します。

1. 「show me a hero card」(ヒーロー カードを見せて) と入力します。 ボットにヒーロー カードと表示されます。

1. 「send me a botframework image」(botframework 画像を送信して) と入力します。 ボットに Bot Framework ドキュメントの画像が表示されます。

1. その他の任意のメッセージを入力すると、ボットは「You said」(と言いましたね) と入力したメッセージを引用符で囲んで応答します。

### <a name="troubleshooting"></a>トラブルシューティング

ボットが動作していない場合は、Microsoft アプリ ID とパスワードを確認するか、入力し直します。 以前にコピーした場合でも、Bot Channels Registration の [設定] ブレードの Microsoft アプリ ID を appsettings.json ファイルの **"MicrosoftAppId"** フィールドの値と照合します。

パスワードを確認するか、新しいパスワードを作成して使用します。 

1. [Bot Channels Registration] ブレードの **[Microsoft App ID]\(Microsoft アプリ ID\)** フィールドの横にある **[管理]** をクリックします。

1. アプリケーション登録ポータルにログインします。

1. パスワードの最初の 3 文字が **appsettings.json** ファイルの **"MicrosoftAppPassword"** フィールドと一致することを確認します。 

1. 値が一致しない場合は、新しいパスワードを生成し、その値を **appsettings.json** ファイルの **"MicrosoftAppPassword"** フィールドに保存します。

# <a name="javascripttabjscreatebot"></a>[JavaScript を選択した場合](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>Microsoft Azure で Web App Bot を作成する

Microsoft Azure でボットを作成するには: 

1. Microsoft Azure portal で、**[リソースの作成]** をクリックし、次に「Web App Bot」を検索します。

1. **Create** をクリックしてください。 [Web App Bot] ブレードが表示されます。

![Web App Bot の登録](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. [Web App Bot] ブレードで、**[ボット名]**、**[サブスクリプション]**、**[リソース グループ]**、**[場所]**、**[価格レベル]**、および **[アプリ名]** を入力します。

1. 使用するボット テンプレートを選択します。 **SDK バージョン: SDK v4** と **SDK 言語: Node.js** 

1. 基本的な v4 プレビュー テンプレートを選択します。 

1. **[App Service プラン/場所]**、**[Azure Storage]**、および **[Application Insights の場所]** を選択します。

1. [ダッシュボードにピン留めする] チェック ボックスをオンにします。

1. [作成] ボタンをクリックします

1. ボットが展開されるまで待ちます。 [ダッシュボードにピン留めする] チェック ボックスをオンにしたので、ダッシュボードにボットが表示されます。

### <a name="edit-your-direct-line-bot"></a>Direct Line ボットを編集する

エコー ボットにいくつか機能を追加してみましょう。

1. [Web App Bot] ブレードで [ビルド] をクリックします。 

1. [Download zip file]\(zip ファイルのダウンロード\) をクリックします。 

1. .zip ファイルをローカル ディレクトリに抽出します。

1. 抽出したフォルダーに移動し、お気に入りの IDE でソース ファイルを開きます。

1. 次のコードをコピーして、app.js ファイルに貼り付けます。

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

準備ができたら、Azure にソースを発行できます。 Azure にボットを発行する方法については、[こちら](../bot-service-build-download-source-code.md)の手順を実行してください

---


## <a name="create-the-console-client-app"></a>コンソール クライアント アプリを作成する

# <a name="ctabcsclientapp"></a>[C# を選択した場合](#tab/csclientapp)

コンソール クライアント アプリケーションは 2 つのスレッドで動作します。 プライマリ スレッドはユーザー入力を受け付け、ボットにメッセージを送信します。 セカンダリ スレッドは、1 秒に 1 回ボットをポーリングし、ボットからのメッセージがあれば取得し、受信したメッセージを表示します。

コンソール プロジェクトを作成するには:

1. ソリューション エクスプローラーで**ソリューション 'DirectLineBotSample'** を右クリックします。

1. **[追加]** > **[新しいプロジェクト]** の順に選択します。

1. **Visual C#** > **[Windows クラシック デスクトップ]* で、**[コンソール アプリ (.NET Framework)]** を選択します。
 
    **注:** **[コンソール アプリ (.NET Core)]** は選択しないでください。

1. 名前に「**DirectLineClientSample**」と入力します。

1. Click **OK**.

### <a name="add-the-nuget-packages-to-the-console-app"></a>NuGet パッケージをコンソール アプリに追加する

1. **[参照]** を右クリックします。

1. **[NuGet パッケージの管理]** をクリックします。

1. **[参照]** をクリックします。 **[プレリリースを含める]** チェック ボックスがオンになっていることを確認します。

1. 以下の NuGet パッケージを検索してインストールします。
    - Microsoft.Bot.Connector.DirectLine (v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>Program.cs ファイルを編集する

DirectLineClientSample の **Program.cs** ファイルの内容を次のように置き換えます。

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

準備が完了したことを確認するために、**F6** キーを押してプロジェクトをビルドします。 警告やエラーは表示されないはずです。

# <a name="javascripttabjsclientapp"></a>[JavaScript を選択した場合](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>Direct Line クライアントを作成する 

Web アプリ ボットを展開したので、次は Direct Line クライアントを作成します。

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

次に npm から以下のパッケージをインストールします。

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

**app.js** ファイルを作成します。 次のコードをコピーして、このファイルに貼り付けます。

次の手順で directLineSecret を取得します。
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>Direct Line チャネルを構成する

Direct Line チャネルを追加すると、このボットは Direct Line ボットになります。

Direct Line チャネルを構成するには:

![Direct Line チャネル ボタンが強調表示された [Connect to Channels]\(チャネルに接続\) ブレード](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. **[Bot Channels Registration]** ブレードで **[チャネル]** をクリックします。 **[Connect to Channels]\(チャネルに接続\)** ブレードが表示されます。

    ![両方の秘密鍵が表示された [Configure Direct Line]\(Direct Line の構成\) ブレード](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. **[Connect to Channels]\(チャネルに接続\)** ブレードで **[Configure Direct Line channel]\(Direct Line チャネルの構成\)** ボタンをクリックします。 

1. **[3.0]** がオフの場合は、チェック ボックスをオンにします。

1. 少なくとも 1 つの秘密鍵について **[表示]** をクリックします。

1. 秘密鍵の 1 つをコピーし、クライアント アプリの **directLineSecret** 文字列に貼り付けます。

1. [Done] をクリックします。

## <a name="run-the-client-app"></a>クライアント アプリの実行

# <a name="ctabcsrunclient"></a>[C# を選択した場合](#tab/csrunclient)

ボットは、Direct Line コンソール クライアント アプリケーションと通信できるようになりました。 コンソール アプリを実行するには、次の手順を実行します。

1. Visual Studio で、**DirectLineClientSample** プロジェクトを右クリックし、**[スタートアップ プロジェクトに設定]** を選択します。

1. **DirectLineClientSample** プロジェクトの **Program.cs** ファイルを調べます。

    - 1 つの Direct Line のシークレット コードが **directLineSecret** 文字列に設定されていることを確認します。

1. **directLineSecret** が正しくない場合は、Azure portal を開き、Direct Line ボットをクリックし、**[チャネル]** をクリックし、**[Configure Direct Line channel]\(Direct Line チャネルの構成\)** (または **[編集]**) をクリックし、キーを表示し、そのキーを **directLineSecret** 文字列にコピーします。

    - リソース グループが **botId** 文字列に設定されていることを確認します。

1. 設定されていない場合は、**[概要]** ブレードの **[リソース グループ]** をコピーし、**botId** 文字列に貼り付けます。

1. F5 キーを押してデバッグを開始します。

コンソール クライアント アプリが起動します。 アプリをテストするには: 

1. 「Hi」(こんにちは) と入力します。 ボットには「You said "Hi"」("こんにちは" と言いましたね) と表示されます。

1. 「show me a hero card」(ヒーロー カードを見せて) と入力します。 ボットにヒーロー カードと表示されます。

1. 「send me a botframework image」(botframework 画像を送信して) と入力します。 ボットからブラウザーが起動され、Bot Framework ドキュメントの画像が表示されます。

1. その他の任意のメッセージを入力すると、ボットは「You said」(と言いましたね) と入力したメッセージを引用符で囲んで応答します。

# <a name="javascripttabjsrunclient"></a>[JavaScript を選択した場合](#tab/jsrunclient)

サンプルを実行するには、DirectLineClient アプリを実行する必要があります。

1. CMD コンソールを開き、CD で DirectLineClient ディレクトリに移動します。

1. `node app.js` を実行します。

カスタム メッセージをテストするには、クライアントのコンソールに「show me a hero card」(ヒーロー カードを見せて)、または「send me a botframework image」(botframework 画像を送信して) と入力します。次のような結果が表示されます。

![結果](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>次の手順
