---
title: Bot Framework Emulator を使用してボットをテストし、デバッグする - Bot Service
description: Bot Framework Emulator デスクトップ アプリケーションを利用し、ボットを検査、試験、デバッグする方法について説明します。
keywords: トランスクリプト, msbot ツール, 言語サービス, 音声認識
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 4828888d2b90cda6854f4195a6bc60f60538820c
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2020
ms.locfileid: "76752834"
---
# <a name="debug-with-the-emulator"></a>エミュレーターを使用したデバッグ

Bot Framework Emulator は、ローカルでもリモートでも、ボットをテストし、デバッグできるデスクトップ アプリケーションです。 このエミュレーターを使用し、ボットとチャットしたり、ボットが送受信するメッセージを検査したりできます。 このエミュレーターでは、Web チャット UI に表示されるときと同じようにメッセージが表示され、ボットとメッセージを交換するときの JSON の要求と応答がログに記録されます。 自分のボットをクラウドにデプロイする前に、エミュレーターを使用してローカルで実行し、テストしてください。 まだ Azure Bot Service で[作成](./bot-service-quickstart.md)していない場合でも、何らかのチャネルで実行するように構成していない場合でも、エミュレーターを使用して自分のボットをテストできます。

## <a name="prerequisites"></a>前提条件
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をインストールします

## <a name="run-a-bot-locally"></a>ボットをローカルで実行する
ボットを Bot Framework Emulator に接続する前に、ボットをローカルで実行する必要があります。 ボットを実行するには、Visual Studio または Visual Studio Code を使用するか、コマンド ラインを使用できます。 コマンド ラインを使用してボットを実行するには、次の操作を行います。


# <a name="c"></a>[C#](#tab/csharp)

* コマンド プロンプトにアクセスし、ボットのプロジェクト ディレクトリに移動します。
* 次のコマンドを実行して、ボットを起動します。 
    ```
    dotnet run
    ```
* 次の記述の前の行のポート番号をコピーします: *Application started.Press Ctrl+C to shut down.* (アプリケーションが開始しました。シャットダウンするには Ctrl + C キーを押してください。)

    ![C# ポート番号](media/bot-service-debug-emulator/csharp_port_number.png)


# <a name="javascript"></a>[JavaScript](#tab/javascript)

* コマンド プロンプトにアクセスし、ボットのプロジェクト ディレクトリに移動します。
* 次のコマンドを実行して、ボットを起動します。
    ```
    node index.js
    ```
* Restify がリッスンするポート番号をコピーします。

    ![JS ポート番号](media/bot-service-debug-emulator/js_port_number.png)

# <a name="python"></a>[Python](#tab/python)
* コマンド プロンプトにアクセスし、ボットのプロジェクト ディレクトリに移動します。
* 次のコマンドを実行して、ボットを起動します。
    ```
   python app.py
    ```
* Restify がリッスンするポート番号をコピーします。

    ![JS ポート番号](media/bot-service-debug-emulator/js_port_number.png)

---

この時点では、ボットはローカルで実行されています。 


## <a name="connect-to-a-bot-running-on-localhost"></a>localhost で実行されているボットに接続する

<!-- auth config steps -->
### <a name="configure-the-emulator-for-authentication"></a>認証用にエミュレーターを構成する

ボットが認証を必要とし、ログイン ダイアログを表示するには、エミュレーターを次のように構成する必要があります。

#### <a name="using-sign-in-verification-code"></a>サインイン確認コードを使用する

1. エミュレーターを起動します。
1. エミュレーターで、左下の歯車アイコンをクリックするか、右上の **[Emulator Settings]** タブをクリックします。
1. **[Use a sign-in verification code for OAuthCards]\(OAuthCard のサインイン確認コードを使用する\)** ボックスをオンにします。
1. **[Bypass ngrok for local address]\(ローカル アドレスに対して ngrok をバイパスする\)** ボックスをオンにします。
1. **[Save]\(保存\)** ボタンをクリックします。

ボットによって表示されるログイン ボタンをクリックすると、確認コードが生成されます。
ボット入力チャット ボックスにコードを入力して、認証を実行します。
その後、許可されている操作を実行できます。

または、以下で説明されている手順を実行することもできます。

#### <a name="using-authentication-tokens"></a>認証トークンを使用する

1. エミュレーターを起動します。
1. エミュレーターで、左下の歯車アイコンをクリックするか、右上の **[Emulator Settings]** タブをクリックします。
1. **[Use version1.0 authentication tokens]\(バージョン1.0 認証トークンを使用する\)** ボックスをオンにします。
1. **ngrok** ツールへのローカル パスを入力します。 ツールの詳細については、「[ngrok](https://ngrok.com/)」を参照してください。
1. **[Run ngrok when the Emulator starts up]\(エミュレーターの起動時に ngrok を実行する\)** ボックスをオンに ます。
1. **[Save]\(保存\)** ボタンをクリックします。

ボットによって表示されるログイン ボタンをクリックすると、資格情報を入力するように求められます。 認証トークンが生成されます。 その後、許可されている操作を実行できます。


![エミュレーター UI](media/emulator-v4/emulator-welcome.png)

ローカルで実行されているボットに接続するには、 **[Open bot]\(ボットを開く\)** をクリックします。 先ほどコピーしたポート番号を次の URL に追加し、その更新した URL を [Bot URL]\(ボットの URL\) バーに貼り付けます。

*http://localhost:**ポート番号**/api/messages*

![エミュレーター UI](media/bot-service-debug-emulator/open_bot_emulator.png)

お使いのボットが [Microsoft アカウント (MSA) の資格情報](#use-bot-credentials)で実行されている場合は、その資格情報も入力します。

### <a name="use-bot-credentials"></a>ボットの資格情報を使用

ボットを開いたときに、ボットが資格情報を使用して実行されている場合は、**Microsoft アプリ ID** および **Microsoft アプリ パスワード**を設定します。 Azure Bot Service でボットを作成した場合、資格情報はそのボットの App Service ( **[設定] -> [構成]** セクションの下) で使用できます。 値がわからない場合は、ローカルで実行されているボットの構成ファイルからそれを削除してから、ボットをエミュレーターで実行できます。 ボットがこれらの設定で実行されていない場合は、その設定でエミュレーターを実行する必要もありません。 

AD ID プロバイダー アプリケーションを作成する場合は次の点に注意してください。

- サポートされるアカウントの種類がシングル テナントに設定されている場合に、Microsoft アカウントではなく個人用サブスクリプションを使用すると、エミュレーターに次のエラーが表示されます。*The bot's Microsoft App ID or Microsoft App Password is incorrect.\(ボットの Microsoft アプリ ID または Microsoft アプリ パスワードが正しくありません。\)* 
- この場合、サポートされているアカウントの種類を *[Accounts in any organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)]\(任意の組織ディレクトリ内のアカウント (任意の Azure AD ディレクトリ - マルチテナント) と個人用の Microsoft アカウント (Skype、Xbox など)\)* に設定する必要があります。

詳細については、「[Azure AD ID プロバイダー アプリケーションを作成する](bot-builder-tutorial-authentication.md#create-an-azure-ad-identity-provider-application)」を参照してください。

## <a name="view-detailed-message-activity-with-the-inspector"></a>インスペクターを利用してメッセージ アクティビティの詳細を表示する

ボットにメッセージを送信すると、ボットが応答を返すはずです。 会話ウィンドウ内でメッセージ吹き出しをクリックし、ウィンドウの右側にある**インスペクター**機能を使用して、未加工の JSON アクティビティを検査できます。 メッセージ吹き出しは選択すると黄色になり、アクティビティ JSON オブジェクトがチャット ウィンドウの左に表示されます。 この JSON 情報には、チャネル ID、アクティビティの種類、会話 ID、テキスト メッセージ、エンドポイント URL など、重要なメタデータが含まれます。 ユーザーから送信されたアクティビティやボットの応答アクティビティを検査できます。

![エミュレーターのメッセージ アクティビティ](media/emulator-v4/emulator-view-message-activity-03.png)

> [!TIP]
> [検査ミドルウェア](bot-service-debug-inspection-middleware.md)をボットに追加することで、チャネルに接続されているボットの状態の変化をデバッグできます。

<!--
## Save and load conversations with bot transcripts

Activities in the emulator can be saved as transcripts. From an open live chat window, select **Save Transcript As** to the transcript file. The **Start Over** button can be used any time to clear a conversation and restart a connection to the bot.  

![Emulator save transcripts](media/emulator-v4/emulator-save-transcript.png)

To load transcripts, simply select **File > Open Transcript File** and select the transcript. A new Transcript window will open and render the message activity to the output window. 

![Emulator load transcripts](media/emulator-v4/emulator-load-transcript.png)
--->
<!---
## Add services 

You can easily add a LUIS app, QnA knowledge base, or dispatch model to your bot directly from the emulator. When the bot is loaded, select the services button on the far left of the emulator window. You will see options under the **Services** menu to add LUIS, QnA Maker, and Dispatch. 

To add a service app, simply click on the **+** button and select the service you want to add. You will be prompted to sign in to the Azure portal to add the service to the bot file, and connect the service to your bot application. 

> [!IMPORTANT]
> Adding services only works if you're using a `.bot` configuration file. Services will need to be added independently. For details on that, see [Manage bot resources](v4sdk/bot-file-basics.md) or the individual how to articles for the service you're trying to add.
>
> If you are not using a `.bot` file, the left pane won't have your services listed (even if your bot uses services) and will display *Services not available*.

![LUIS connect](media/emulator-v4/emulator-connect-luis-btn.png)

When either service is connected, you can go back to a live chat window and verify that your services are connected and working. 

![QnA connected](media/emulator-v4/emulator-view-message-activity.png)

--->

## <a name="inspect-services"></a>サービスの検査

新しい v4 エミュレーターでは、LUIS と QnA から JSON 応答を検査することもできます。 ボットと言語サービスが接続されているとき、右下の LOG ウィンドウにある **[trace]** を選択できます。 この新しいツールには、エミュレーターから直接、言語サービスを更新するための機能もあります。 

![LUIS インスペクター](media/emulator-v4/emulator-luis-inspector.png)

LUIS サービスに接続しているとき、トレース リンクに **Luis Trace** と指定されていることに気付きます。 これが選択されると、意図、エンティティ、指定スコアなど、LUIS サービスからの raw 応答が表示されます。 ユーザー発話の意図を再割り当てすることもできます。 

![QnA インスペクター](media/emulator-v4/emulator-qna-inspector.png)

QnA サービスに接続しているとき、ログに **QnA Trace** と表示されます。これが選択されると、そのアクティビティに関連付けられている質問と回答の組みと信頼度スコアをプレビューできます。 ここから、ある回答に対する質問に別の言い回しを追加できます。

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

<!---## Login to Azure

You can use Emulator to login in to your Azure account. This is particularly helpful for you to add and manage services your bot depends on. 
See [above](#add-services) to learn more about services you can manage using the Emulator.
-->

### <a name="login-to-azure"></a>Azure にログインする
Emulator を使って、ご自身の Azure アカウントにログインできます。 これは、ご自身のボットが依存するサービスを追加および管理するときに特に役に立ちます。 次の手順に従って、Azure にログインします。
- [ファイル]、[Sign in with Azure]\(Azure を使ってサインインする\) の順にクリックします ![Azure ログイン](media/emulator-v4/emulator-azure-login.png)
- ようこそ画面で、[Sign in with your Azure account]\(Azure アカウントを使ってサインインする\) をクリックします。Emulator アプリケーションを再起動しても、その Emulator でのサインイン状態が継続されるようにするように設定することもできます。
![Azure ログイン](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>データ収集の無効化

Emulator による使用状況データの収集を許可する必要がなくなった場合は、次の手順に従って、データ収集を簡単に無効にできます。

1. 左側のナビゲーション バーにある設定ボタン (歯車アイコン) をクリックして、Emulator の設定ページに移動します。

    ![データ収集を無効にする](media/emulator-v4/emulator-disable-data-1.png)

2. **[データ収集]** セクションの *[Help improve the Emulator by allowing us to collect usage data]\(使用状況データの収集を許可して Emulator を改善する\)* チェック ボックスをオフにします。

    ![データ収集を無効にする](media/emulator-v4/emulator-disable-data-2.png)

3. [Save]\(保存\) ボタンをクリックします。

    ![データ収集を無効にする](media/emulator-v4/emulator-disable-data-3.png)
    
気が変わった場合は、いつでもチェック ボックスをオンにして有効にできます。

## <a name="additional-resources"></a>その他のリソース

Bot Framework Emulator はオープン ソースです。 開発に[貢献][EmulatorGithubContribute]したり、[バグや提案を送信][EmulatorGithubBugs]したりできます。

トラブルシューティングについては、[一般的な問題のトラブルシューティング](bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。

## <a name="next-steps"></a>次のステップ

検査ミドルウェアを使用して、チャネルに接続されているボットをデバッグします。

> [!div class="nextstepaction"]
> [トランスクリプト ファイルを使用してお使いのボットをデバッグする](bot-service-debug-inspection-middleware.md)

<!--
Saving a conversation to a transcript file allows you to quickly draft and replay a certain set of interactions for debugging.

> [!div class="nextstepaction"]
> [Debug your bot using transcript files](~/v4sdk/bot-builder-debug-transcript.md)
-->

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
