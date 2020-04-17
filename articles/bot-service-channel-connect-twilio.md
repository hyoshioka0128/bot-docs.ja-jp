---
title: ボットを Twilio に接続する - Bot Service
description: ボットの Twilio への接続を構成する方法について説明します。
keywords: Twilio, ボット チャネル, SMS, アプリ, 電話, Twilio の構成, クラウド通信, テキスト
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/9/2018
ms.openlocfilehash: 42e79bbf79418c30ff0b4b6c584ab4791cd246c0
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75793093"
---
# <a name="connect-a-bot-to-twilio"></a>ボットを Twilio に接続する

Twilio クラウド通信プラットフォームを使用してユーザーと通信するようにボットを構成できます。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>SMS メッセージを送受信するための Twilio アカウントにログインするかアカウントを作成する

Twilio アカウントを持っていない場合は、<a href="https://www.twilio.com/try-twilio" target="_blank">新しいアカウントを作成します</a>。

## <a name="create-a-twiml-application"></a>TwiML アプリケーションを作成する

手順に従って、<a href="https://support.twilio.com/hc/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">TwiML アプリケーションを作成</a>します。

![アプリを作成する](~/media/channels/twi-StepTwiml.png)

**[Properties]\(プロパティ\)** の下で、**フレンドリ名**を入力します。 このチュートリアルでは、例として "My TwiML app" を使用します。 [Voice]\(音声\) の下の **[REQUEST URL]\(要求 URL\)** は空のままにすることができます。 **[Messaging]\(メッセージング\)** の下の **[Request URL]\(要求 URL\)** は `https://sms.botframework.com/api/sms` にする必要があります。

## <a name="select-or-add-a-phone-number"></a>電話番号を選択または追加する

<a href = "https://support.twilio.com/hc/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">こちら</a>の手順に従って、コンソール サイト経由で検証済みの呼び出し元の ID を追加します。 終了すると、 **[Manage Numbers]\(番号の管理\)** の下の **[Active Numbers]\(アクティブな番号\)** に検証済みの番号が表示されます。

![電話番号を設定する](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>音声とメッセージングに使用するアプリケーションを指定する

番号をクリックし、 **[Configure]\(構成\)** に移動します。 [Voice]\(音声\) と [Messaging]\(メッセージング\) の両方で、 **[CONFIGURE WITH]\(構成の対象\)** を [TwiML App]\(TwiML アプリ\) に設定し、 **[TWIML APP]\(TwiML アプリ\)** を "My TwiML app" に設定します。 終了したら、 **[保存]** をクリックします。

![アプリケーションを指定する](~/media/channels/twi-StepPhone2.png)

**[Manage Numbers]\(番号の管理\)** に戻ります。[Voice]\(音声\) と [Messaging]\(メッセージング\) の両方の構成が [TwiML App]\(TwiML アプリ\) に変更されています。

![指定された番号](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>資格情報を収集する

[コンソール ホームページ](https://www.twilio.com/console/)に戻ります。次のように、アカウント SID と認証トークンがプロジェクト ダッシュボードに表示されます。

![アプリの資格情報を収集する](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>資格情報を送信する

別のウィンドウで、Bot Framework サイト (https://dev.botframework.com/ ) に戻ります。 

- **[My bots]\(マイ ボット\)** を選択し、Twilio に接続するボットを選択します。 これにより Azure portal に移動します。
- **[Bot Management]\(ボット管理\)** の下の **[チャネル]** を選択します。 Twilio (SMS) アイコンをクリックします。
- 前に記録した電話番号、アカウント SID、および認証トークンを入力します。 終了したら、 **[保存]** をクリックします。

![資格情報を送信する](~/media/channels/twi-StepSubmit.png)

これらの手順を完了すると、ボットは、Twilio を使ってユーザーと通信するように正しく構成されます。

## <a name="connect-a-bot-to-twilio-using-the-twilio-adapter"></a>Twilio アダプターを使用してボットを Twilio に接続する

Twilio にボットを接続するには、Azure Bot Service で用意されているチャネルを使用するほかに、Twilio アダプターを使用する方法があります。 この記事では、アダプターを使用してボットを Twilio に接続する方法について説明します。  この記事では、Twilio に接続するよう EchoBot サンプルに変更を加える手順を解説します。

> [!NOTE]
> 以下の手順では、Twilio アダプターの C# 実装について説明しています。 JS アダプター (BotKit ライブラリに含まれます) の使用手順については、[BotKit Twilio のドキュメント](https://botkit.ai/docs/v4/platforms/twilio.html)を参照してください。

### <a name="prerequisites"></a>前提条件

* [EchoBot サンプル コード](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot)

* Twilio アカウント。 Twilio アカウントをお持ちでない場合は、[こちらで作成](https://www.twilio.com/try-twilio)できます。

### <a name="get-a-twilio-number-and-gather-account-credentials"></a>Twilio 番号を取得してアカウントの資格情報を収集する

1. [Twilio](https://twilio.com/console) にログインします。 ご利用のアカウントの **[ACCOUNT SID]\(アカウント SID\)** と **[AUTH TOKEN]\(認証トークン\)** がページの右側に表示されるので、それらを書き留めておいてください。後で自分のボット アプリケーションを構成する際に必要になります。

2. **[Get Started with Twilio]\(Twilio の開始\)** のオプションから **[Programmable Voice]\(プログラミング可能な音声\)** を選択します。

![プログラミング可能な音声で開始する](~/media/bot-service-channel-connect-twilio/get-started-voice.png)

3. 次のページで、 **[Get your first Twilio number]\(最初の Twilio 番号を取得\)** ボタンをクリックします。  ポップアップ ウィンドウに新しい番号が表示されます。これは、 **[Choose this number]\(この番号を選択する\)** をクリックすることで受け入れられます (画面の指示に従って別の番号を検索することもできます)。

4. 目的の番号を選択したら、それを書き留めておいてください。後の手順でボット アプリケーションを構成する際に必要になります。

### <a name="wiring-up-the-twilio-adapter-in-your-bot"></a>ボットに Twilio アダプターを接続する

Twilio 番号とアカウントの資格情報が揃ったら、自分のボット アプリケーションを構成する必要があります。

#### <a name="install-the-twilio-adapter-nuget-package"></a>Twilio アダプターの NuGet パッケージをインストールする

[Microsoft.Bot.Builder.Adapters.Twilio](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Twilio/) NuGet パッケージを追加します。 NuGet の使用方法について詳しくは、[Visual Studio でパッケージをインストールして管理する方法](https://aka.ms/install-manage-packages-vs)に関するページを参照してください。

#### <a name="create-a-twilio-adapter-class"></a>Twilio アダプター クラスを作成する

***TwilioAdapter*** クラスを継承する新しいクラスを作成します。 Twilio チャネルのアダプターとして機能するこのクラスには、エラー処理機能が含まれています (これは Azure Bot Service からの他の要求を処理するために使用される、サンプル内の ***BotFrameworkAdapterWithErrorHandler*** クラスに類似しています)。

```csharp
public class TwilioAdapterWithErrorHandler : TwilioAdapter
{
    public TwilioAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
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

#### <a name="create-a-new-controller-for-handling-twilio-requests"></a>Twilio の要求を処理するための新しいコントローラーを作成する

Azure Bot Service チャネルからの要求に使用される既定の "api/messages" ではなく、新しいエンドポイント "api/twilio" に、Twilio からの要求を処理する新しいコントローラーを作成します。  自分のボットに新しくエンドポイントを追加することによって、Bot Service チャネルからの要求だけでなく、Twilio からの要求も、同じボットを使用して受け入れることができます。

```csharp
[Route("api/twilio")]
[ApiController]
public class TwilioController : ControllerBase
{
    private readonly TwilioAdapter _adapter;
    private readonly IBot _bot;

    public TwilioController(TwilioAdapter adapter, IBot bot)
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

#### <a name="inject-the-twilio-adapter-in-your-bot-startupcs"></a>ボットの startup.cs に Twilio アダプターを挿入する

自分の startup.cs ファイル内の ***ConfigureServices*** メソッドに次の行を追加します。 これによって Twilio アダプターが登録され、新しいコントローラー クラスで利用できるようになります。  前の手順で追加した構成設定が、アダプターによって自動的に使用されます。

```csharp
services.AddSingleton<TwilioAdapter, TwilioAdapterWithErrorHandler>();
```

追加後の ***ConfigureServices*** メソッドは次のようになります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Twilio Adapter
    services.AddSingleton<TwilioAdapter, TwilioAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

#### <a name="obtain-a-url-for-your-bot"></a>ボットの URL を取得する

自分のボット プロジェクトにアダプターを接続したら、自分のボットが確実にメッセージを受信できるよう、Twilio に提供する適切なエンドポイントを指定する必要があります。 また、自分のボット アプリケーションの構成を完了するためにも、この URL が必要です。

この手順を実行するために、[ボットを Azure にデプロイ](https://aka.ms/bot-builder-deploy-az-cli)し、そのデプロイしたボットの URL を書き留めてください。

> [!NOTE]
> 自分のボットを Azure にデプロイする準備が整っていない場合や、Twilio アダプターを使用している状態でボットをデバッグしたい場合は、[ngrok](https://www.ngrok.com) (Bot Framework エミュレーターを過去に使用したことがある場合は、既にインストールされている可能性があります) などのツールを使用して、ローカルで実行されているボットにトンネル接続し、そのパブリック アクセス用の URL を取得してください。 
> 
> ngrok トンネルを作成して自分のボットへの URL を取得したい場合は、ターミナル ウィンドウで次のコマンドを使用します (これは、ローカル ボットがポート 3978 で実行されていることを想定しています。それとは異なるポートでボットが実行されている場合は、コマンド内のポート番号を変更してください)。
> 
> ```
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

#### <a name="add-twilio-app-settings-to-your-bots-configuration-file"></a>ボットの構成ファイルに Twilio アプリの設定を追加する

次に示す設定を自分のボット プロジェクトの appSettings.json ファイルに追加します。 **TwilioNumber**、**TwilioAccountSid**、**TwilioAuthToken** は、Twilio 番号を作成する際に収集した値を使用して設定してください。 **TwilioValidationUrl** には、自分のボットの URL と、新しく作成したコントローラーで指定した `api/twilio` エンドポイントを入力する必要があります。 たとえば、「 `https://yourboturl.com/api/twilio` 」のように入力します。


```json
  "TwilioNumber": "",
  "TwilioAccountSid": "",
  "TwilioAuthToken": "",
  "TwilioValidationUrl", ""
```

上記の設定が済んだら、自分のボットを再デプロイ (ngrok を使用してローカルで実行している場合は再起動) する必要があります。

### <a name="complete-configuration-of-your-twilio-number"></a>Twilio 番号の構成を完了する

最後の手順では、自分のボットがメッセージを確実に受信できるように、新しい Twilio 番号のメッセージング エンドポイントを構成します。

1. Twilio の [[Active Numbers]\(アクティブ番号\) ページ](https://www.twilio.com/console/phone-numbers/incoming)に移動します。

2. 前の手順で作成した電話番号をクリックします。

3. **[Messaging]\(メッセージング\)** セクションの **[A MESSAGE COMES IN]\(受信されるメッセージの形式\)** セクションに必要事項を入力します。ドロップダウンから **[Webhook]** を選択し、前の手順で **TwilioValidationUrl** を設定する際に使用したボットのエンドポイントをテキスト ボックスに入力してください (例: `https://yourboturl.com/api/twilio`)。

![Twilio 番号の Webhook を構成する](~/media/bot-service-channel-connect-twilio/twilio-number-messaging-settings.png)

4. **[保存]** ボタンをクリックします。

### <a name="test-your-bot-with-adapter-in-twilio"></a>Twilio のアダプターを使用してボットをテストする

SMS メッセージを自分の Twilio 番号に送信して、自分のボットが Twilio に正しく接続されているかどうかをテストしてみましょう。  ボットはメッセージを受信すると、そのメッセージのテキストをエコーする形でメッセージを返します。

この機能のテストには、[Twilio アダプターのサンプル ボット](https://aka.ms/csharp-63-twilio-adapter-sample)を使用することもできます。上記の手順で説明したのと同じ値を appSettings.json に設定してください。
