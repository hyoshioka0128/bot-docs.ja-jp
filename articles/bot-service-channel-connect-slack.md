---
title: ボットを Slack に接続する - Bot Service
description: ボットの Slack への接続を構成する方法について説明します。
keywords: ボットの接続, ボット チャネル, Slack ボット, Slack メッセージング アプリ, Slack アダプター
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/09/2019
ms.openlocfilehash: 3147e202a615e29d51f1e3fa3a9d5d70ed54fe83
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071740"
---
# <a name="connect-a-bot-to-slack"></a>ボットを Slack に接続する

Slack メッセージング アプリは、次の 2 とおりの方法で構成できます。
- Azure Bot Service ポータルを使用してボットを接続する
- Slack アダプターを使用する

## <a name="azure-bot-service-portaltababs"></a>[Azure Bot Service ポータル](#tab/abs)
## <a name="create-a-slack-application-for-your-bot"></a>ボット用の Slack アプリケーションを作成する

[Slack](https://slack.com/signin) にログインし、[[create a Slack application]\(Slack アプリケーションの作成\)](https://api.slack.com/apps) チャネルに移動します。

![ボットを設定する](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>アプリを作成して開発 Slack チームを割り当てる

アプリ名を入力し、開発 Slack チームを選択します。 まだ開発 Slack チームのメンバーになっていない場合は、[作成または参加](https://slack.com/)します。

![アプリを作成する](~/media/channels/slack-CreateApp.png)

**[Create app (アプリの作成)]** をクリックします。 Slack でアプリが作成されて、クライアント ID とクライアント シークレットが生成されます。

## <a name="add-a-new-redirect-url"></a>新しいリダイレクト URL を追加する

次に、新しいリダイレクト URL を追加します。

1. **[OAuth & Permissions]\(OAuth とアクセス許可\)** タブを選択します。
2. **[Add a new Redirect URL]\(新しいリダイレクト URL の追加\)** をクリックします。
3. 「[https://slack.botframework.com](https://slack.botframework.com)」を入力します。
4. **[追加]** をクリックします。
5. **[Save URLs]\(URL の保存\)** をクリックします。

![リダイレクト URL を追加する](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Slack ボット ユーザーを作成する

ボット ユーザーを追加すると、ボットにユーザー名を割り当てて、ボットを常にオンラインとして表示するかどうかを選択できます。

1. **[Bot Users]\(ボット ユーザー\)** タブを選択します。
2. **[Add a Bot User]\(ボット ユーザーの追加\)** をクリックします。

![ボットを作成する](~/media/channels/slack-CreateBot.png)

**[Add Bot User]\(ボット ユーザーの追加\)** をクリックして設定を検証し、 **[Always Show My Bot as Online]\(ボットを常にオンラインとして表示する\)** をクリックして **[On]\(オン\)** に設定した後、 **[Save Changes]\(変更の保存\)** をクリックします。

![ボットを作成する](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>ボットのイベントをサブスクライブする

以下の手順に従って、ボットの 6 つの特定のイベントをサブスクライブします。 ボットのイベントをサブスクライブすると、指定した URL でのユーザー アクティビティがアプリに通知されます。

> [!TIP]
> ボット ハンドルはボットの名前です。 ボットのハンドルを調べるには、[https://dev.botframework.com/bots](https://dev.botframework.com/bots) にアクセスして、ボットを選択し、ボットの名前を記録します。

1. **[Event Subscriptions]\(イベント サブスクリプション\)** タブを選択します。
2. **[Enable Events]\(イベントを有効にする\)** をクリックして **[On]\(オン\)** にします。
3. **[要求 URL]** に「`https://slack.botframework.com/api/Events/{YourBotHandle}`」と入力します。`{YourBotHandle}` は、中かっこを除いたボット ハンドルです。 この例で使用されるボット ハンドルは **ContosoBot** です。

   ![イベントをサブスクライブする: 上](~/media/channels/slack-SubscribeEvents-a.png)

4. **[Subscribe to Bot Events]\(ボット イベントのサブスクライブ\)** で、 **[Add Bot User Event]\(ボット ユーザー イベントの追加\)** をクリックします。
5. イベントの一覧で、次の 6 つのイベント タイプを選択します。
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![イベントをサブスクライブする: 中](~/media/channels/slack-SubscribeEvents-b.png)

6. **[変更を保存]** をクリックします。

   ![イベントをサブスクライブする: 下](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>対話型メッセージを追加して構成する (省略可能)

ボットがボタンなどの Slack 固有の機能を使う場合は、次の手順のようにします。

1. **[Interactive Components]\(対話型コンポーネント\)** タブを選択し、 **[Enable Interactive Components]\(対話型コンポーネントを有効にする\)** をクリックします。
2. **[Request URL]\(要求 URL\)** として「`https://slack.botframework.com/api/Actions`」と入力します。
3. **[Save changes]\(変更の保存\)** ボタンをクリックします。

![メッセージを有効にする](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>資格情報を収集する

**[Basic Information]\(基本情報\)** タブを選択し、 **[App Credentials]\(アプリの資格情報\)** セクションまでスクロールします。
Slack ボットの構成に必要なクライアント ID、クライアント シークレット、検証トークンが表示されます。

![資格情報を収集する](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>資格情報を送信する

別のブラウザー ウィンドウで、Bot Framework サイト (`https://dev.botframework.com/`) に戻ります。

1. **[My bots]\(マイ ボット\)** を選択し、Slack に接続するボットを選択します。
2. **[チャネル]** セクションで、Slack アイコンをクリックします。
3. **[Enter your Slack credentials]\(Slack 資格情報の入力\)** セクションで、Slack の Web サイトからコピーしたアプリの資格情報を適切なフィールドに貼り付けます。
4. **[Landing Page URL]\(ランディング ページの URL\)** は省略可能です。 省略しても変更してもかまいません。
5. **[保存]** をクリックします。

![資格情報を送信する](~/media/channels/slack-SubmitCredentials.png)

指示に従って、開発 Slack チームへの Slack アプリのアクセスを承認します。

## <a name="enable-the-bot"></a>ボットを有効にする

[Configure Slack]\(Slack の構成\) ページで、[Save]\(保存\) ボタンのスライダーが **[Enabled]\(有効\)** に設定されていることを確認します。
ボットは、Slack のユーザーと通信するように構成されています。

## <a name="create-an-add-to-slack-button"></a>Slack に追加ボタンを作成する

[このページ](https://api.slack.com/docs/slack-button)の「*Add the Slack button*」(Slack ボタンを追加する) セクションでは、Slack ユーザーがボットを検索するときに使用できる HTML が提供されています。
ボットでこの HTML を使うには、(`https://` で始まる) href の値を、ボットの Slack チャネルの設定で示されている URL に置き換えます。
置き換える URL を取得するには、次の手順のようにします。

1. [https://dev.botframework.com/bots](https://dev.botframework.com/bots) で、対象のボットをクリックします。
2. **[チャネル]** をクリックし、 **[Slack]** というエントリを右クリックして、 **[Copy link]\(リンクのコピー\)** をクリックします。 この URL がクリップボードにコピーされます。
3. この URL をクリップボードから Slack ボタンの HTML に貼り付けます。 この URL で、このボットに対して Slack によって提供される href の値を置き換えます。

承認されたユーザーは、この変更した HTML で提供される **[Add to Slack]\(Slack に追加\)** ボタンをクリックして、Slack からボットにアクセスできます。

## <a name="slack-adaptertabadapter"></a>[Slack アダプター](#tab/adapter)
## <a name="connect-a-bot-to-slack-using-the-slack-adapter"></a>Slack アダプターを使用してボットを Slack に接続する

Slack にボットを接続するには、Azure Bot Service で用意されているチャネルを使用するほかに、Slack アダプターを使用する方法があります。 この記事では、アダプターを使用してボットを Slack に接続する方法について説明します。  この記事では、Slack アプリに接続するよう EchoBot サンプルに変更を加える手順を解説します。

> [!NOTE]
> 以下の手順では、Slack アダプターの C# 実装について説明しています。 JS アダプター (BotKit ライブラリに含まれます) の使用手順については、[BotKit Slack のドキュメント](https://botkit.ai/docs/v4/platforms/slack.html)を参照してください。

## <a name="prerequisites"></a>前提条件

* [EchoBot サンプル コード](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot)

* [https://api.slack.com/apps](https://api.slack.com/apps) でアプリケーションを作成、管理するのに十分なアクセス許可で Slack ワークスペースにアクセスします。 Slack 環境へのアクセス権がない場合は、ワークスペースを[無料](https://www.slack.com)で作成できます。

## <a name="create-a-slack-application-for-your-bot"></a>ボット用の Slack アプリケーションを作成する

[Slack](https://slack.com/signin) にログインし、[[create a Slack application]\(Slack アプリケーションの作成\)](https://api.slack.com/apps) チャネルに移動します。

![ボットを設定する](~/media/channels/slack-NewApp.png)

[Create new app]\(新しいアプリの作成\) ボタンをクリックします。

### <a name="create-an-app-and-assign-a-development-slack-team"></a>アプリを作成して開発 Slack チームを割り当てる

**アプリ名**を入力し、 **[Development Slack Workspace]\(開発用 Slack ワークスペース\)** を選択します。 まだ開発 Slack チームのメンバーになっていない場合は、[作成または参加](https://slack.com/)します。

![アプリを作成する](~/media/channels/slack-CreateApp.png)

**[Create app (アプリの作成)]** をクリックします。 Slack でアプリが作成されて、クライアント ID とクライアント シークレットが生成されます。

### <a name="gather-required-configuration-settings-for-your-bot"></a>ボットに必要な構成設定を収集する

アプリが作成されたら、次の情報を収集します。 これは、自分のボットを Slack に接続するために必要になります。

1. **[Basic Information]\(基本情報\)** タブの **[Verification Token]\(検証トークン\)** と **[Signing Secret]\(署名用のシークレット\)** をメモしておいてください。これらは後でボットの設定を構成するのに必要になります。

![Slack のトークン](~/media/bot-service-adapter-connect-slack/slack-tokens.png)

2. **[Settings]\(設定\)** メニューの **[Install App]\(アプリのインストール\)** ページに移動し、手順に従って Slack チームに自分のアプリをインストールします。  インストール後、 **[Bot User OAuth Access Token]\(ボット ユーザーの OAuth アクセス トークン\)** をコピーします。これも、後でボットの設定を構成する際に必要になるので保存しておいてください。

## <a name="wiring-up-the-slack-adapter-in-your-bot"></a>ボットに Slack アダプターを接続する

### <a name="install-the-slack-adapter-nuget-package"></a>Slack アダプターの NuGet パッケージをインストールする

[Microsoft.Bot.Builder.Adapters.Slack](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Slack/) NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、[Visual Studio でパッケージをインストールして管理する方法](https://aka.ms/install-manage-packages-vs)に関するページを参照してください

### <a name="create-a-slack-adapter-class"></a>Slack アダプター クラスを作成する

***SlackAdapter*** クラスを継承する新しいクラスを作成します。 Slack チャネルのアダプターとして機能するこのクラスには、エラー処理機能が含まれています (これは Azure Bot Service からの他の要求を処理するために使用される、サンプル内の ***BotFrameworkAdapterWithErrorHandler*** クラスに類似しています)。

```csharp
public class SlackAdapterWithErrorHandler : SlackAdapter
{
    public SlackAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
    : base(configuration, logger)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError(exception, $"[OnTurnError] unhandled error : {exception.Message}");

            // Send a message to the user
            await turnContext.SendActivityAsync("The bot encountered an error or bug.");
            await turnContext.SendActivityAsync("To continue to run this bot, please fix the bot source code.");

            // Send a trace activity, which will be displayed in the Bot Framework Emulator
            await turnContext.TraceActivityAsync("OnTurnError Trace", exception.Message, "https://www.botframework.com/schemas/error", "TurnError");
        };
    }
}
```

### <a name="create-a-new-controller-for-handling-slack-requests"></a>Slack の要求を処理するための新しいコントローラーを作成する

Azure Bot Service チャネルからの要求に使用される既定の "api/messages" ではなく、新しいエンドポイント "api/slack" に、自分の Slack アプリからの要求を処理する新しいコントローラーを作成します。  自分のボットに新しくエンドポイントを追加することによって、Bot Service チャネルからの要求だけでなく、Slack からの要求も、同じボットを使用して受け入れることができます。

```csharp
[Route("api/slack")]
[ApiController]
public class SlackController : ControllerBase
{
    private readonly SlackAdapter _adapter;
    private readonly IBot _bot;

    public SlackController(SlackAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost]
    [HttpGet]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```

### <a name="add-slack-app-settings-to-your-bots-configuration-file"></a>ボットの構成ファイルに Slack アプリの設定を追加する

自分のボット プロジェクトの appSettings.json ファイルに次の 3 つの設定を追加します。それぞれ、自分の Slack アプリの作成時に収集した値を設定してください。

```json
  "SlackVerificationToken": "",
  "SlackBotToken": "",
  "SlackClientSigningSecret": ""
```

### <a name="inject-the-slack-adapter-in-your-bot-startupcs"></a>ボットの startup.cs に Slack アダプターを挿入する

自分の startup.cs ファイル内の ***ConfigureServices*** メソッドに次の行を追加します。 これによって Slack アダプターが登録され、新しいコントローラー クラスで利用できるようになります。  前の手順で追加した構成設定が、アダプターによって自動的に使用されます。

```csharp
services.AddSingleton<SlackAdapter, SlackAdapterWithErrorHandler>();
```

追加後の ***ConfigureServices*** メソッドは次のようになります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Slack Adapter
    services.AddSingleton<SlackAdapter, SlackAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

## <a name="complete-configuration-of-your-slack-app"></a>Slack アプリの構成を完了する

### <a name="obtain-a-url-for-your-bot"></a>ボットの URL を取得する

自分のボット プロジェクトで Slack アプリを作成し、アダプターを接続したら、最後の手順となります。ボット上の正しいエンドポイントを指すように Slack アプリを設定し、目的のアプリをサブスクライブして、ボットがメッセージを確実に受信できるようにしましょう。  そのためには、自分のボットが実行されていて、エンドポイントへの URL が有効であることを Slack が検証できるようになっている必要があります。

この手順を実行するために、[ボットを Azure にデプロイ](https://aka.ms/bot-builder-deploy-az-cli)し、そのデプロイしたボットへの URL を書き留めてください。

> [!NOTE]
> 自分のボットを Azure にデプロイする準備が整っていない場合や、Slack アダプターを使用している状態でボットをデバッグしたい場合は、[ngrok](https://www.ngrok.com) (Bot Framework エミュレーターを過去に使用したことがある場合は、既にインストールされている可能性があります) などのツールを使用して、ローカルで実行されているボットにトンネル接続し、そのパブリック アクセス用の URL を取得してください。 
> 
> ngrok トンネルを作成して自分のボットへの URL を取得したい場合は、ターミナル ウィンドウで次のコマンドを使用します (これは、ローカル ボットがポート 3978 で実行されていることを想定しています。それとは異なるポートでボットが実行されている場合は、コマンド内のポート番号を変更してください)。
> 
> ```
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

### <a name="update-your-slack-app"></a>Slack アプリを更新する

[Slack API ダッシュボード]([https://api.slack.com/apps])に戻って、自分のアプリを選択します。  次に、自分のアプリの 2 つの URL を構成し、適切なイベントをサブスクライブする必要があります。

1. **[OAuth & Permissions]\(OAuth とアクセス許可\)** タブの **[Redirect URL]\(リダイレクト URL\)** に、自分のボットの URL と、新しく作成したコントローラーで指定した `api/slack` エンドポイントを入力します。 たとえば、「 `https://yourboturl.com/api/slack` 」のように入力します。

![Slack のリダイレクト URL](~/media/bot-service-adapter-connect-slack/redirect-url.png)

2. **[Event Subscriptions]\(イベント サブスクリプション\)** タブの **[Request URL]\(要求 URL\)** に、手順 1. で使用したのと同じ URL を入力します。

3. ページ上部のトグルを使用してイベントを有効にします。

4. **[Subscribe to bot events]\(ボット イベントのサブスクライブ\)** セクションを展開し、 **[Add Bot User Event]\(ボット ユーザー イベントの追加\)** ボタンを使用して、**im_created** イベントと **message.im** イベントをサブスクライブします。

![Slack イベントのサブスクリプション](~/media/bot-service-adapter-connect-slack/event-subscriptions.png)

## <a name="test-your-bot-with-adapter-in-slack"></a>Slack のアダプターを使用してボットをテストする

Slack アプリの構成が完了したので、自分のアプリのインストール先である Slack ワークスペースにログインできるようになりました (左側のメニューの [Apps]\(アプリ\) セクションに表示されます)。自分のアプリを選択して、メッセージを送信してみてください。 IM ウィンドウにエコーバックされることがわかります。

この機能のテストには、[Slack アダプターのサンプル ボット](https://aka.ms/csharp-60-slack-adapter-sample)を使用することもできます。上記の手順で説明したのと同じ値を appSettings.json ファイルに設定してください。 このサンプルの README ファイルには、リンク共有、添付ファイルの受信、インタラクティブ メッセージの送信の例を紹介する手順も記載されています。

---
