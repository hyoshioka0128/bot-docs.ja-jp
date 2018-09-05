---
title: Bot Framework Emulator を使用してボットをテストし、デバッグする | Microsoft Docs
description: Bot Framework Emulator デスクトップ アプリケーションを利用し、ボットを検査、試験、デバッグする方法について説明します。
keywords: トランスクリプト, msbot ツール, 言語サービス, 音声認識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 2a92281e9bbee09d8dfb00fbbc67fe6ac6b6c69b
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756690"
---
# <a name="debug-with-the-bot-framework-emulator"></a>Bot Framework Emulator でデバッグする

Bot Framework Emulator は、ローカルでもリモートでも、ボットをテストし、デバッグできるデスクトップ アプリケーションです。 このエミュレーターを使用し、ボットとチャットしたり、ボットが送受信するメッセージを検査したりできます。 このエミュレーターでは、Web チャット UI に表示されるときと同じようにメッセージが表示され、ボットとメッセージを交換するときの JSON の要求と応答がログに記録されます。 

> [!TIP] 
> 自分のボットをクラウドにデプロイする前に、エミュレーターを使用してローカルで実行し、テストしてください。 Bot Framework に[登録](~/bot-service-quickstart-registration.md)していない場合でも、何らかのチャネルで実行するように構成していない場合でも、エミュレーターを使用して自分のボットをテストできます。

![エミュレーター UI](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>Bot Framework Emulator をダウンロードする

Mac、Windows、Linux 向けのダウンロード パッケージは [GitHub リリース ページ](https://github.com/Microsoft/BotFramework-Emulator/releases)から入手できます。

## <a name="connect-to-a-bot-running-on-local-host"></a>ローカル ホストで実行されているボットに接続する

![エミュレーター エンドポイント](media/emulator-v4/emulator-endpoint.png)

ローカルで実行されているボットに接続するには、左上の [Bot Explorer] タブを選択します。 **[エンドポイント]** タブの下にある **+** アイコンをクリックします。ここで、ボットに接続する目的で、ボットがローカルで実行されているポートと同じポートにエンドポイントを指定できます。 **[送信]** をクリックすると、ライブ チャット ウィンドウにリダイレクトされます。このウィンドウで自分のボットと対話できます。

## <a name="view-detailed-message-activity-with-the-inspector"></a>インスペクターを利用してメッセージ アクティビティの詳細を表示する

![エミュレーターのメッセージ アクティビティ](media/emulator-v4/emulator-view-message-activity-02.png)

会話ウィンドウ内でいずれかのメッセージ吹き出しをクリックし、ウィンドウの右にある **INSPECTOR** 機能を使用して raw JSON アクティビティを検査できます。 メッセージ吹き出しは選択すると黄色になり、アクティビティ JSON オブジェクトがチャット ウィンドウの左に表示されます。 この JSON 情報には、チャネル ID、アクティビティの種類、会話 ID、テキスト メッセージ、エンドポイント URL など、重要なメタデータが含まれます。ユーザーから送信されたアクティビティやボットの応答アクティビティを表示し、検査できます。 

[Channel Inspector](bot-service-channel-inspector.md) を利用し、特定のチャネルでサポートされている機能をプレビューすることもできます。


## <a name="save-and-load-conversations-with-bot-transcripts"></a>ボット トランスクリプトと共に会話を保存し、読み込む

エミュレーターのメッセージ アクティビティはトランスクリプトとして保存できます。 開いているライブ チャット ウィンドウから、**[Save Transcript As]\(名前を付けてトランスクリプトを保存\)** を選択し、出力されたトランスクリプト ファイルに名前を付け、保存場所を選択できます。 

>[!TIP]
> **[やり直す]** ボタンを使用し、いつでも会話を消去し、ボットへの接続を再起動できます。  

![エミュレーターでトランスクリプトを保存する](media/emulator-v4/emulator-live-chat.png)

トランスクリプトを読み込むには、**[ファイル]** --> **[Open Transcript File]\(トランスクリプト ファイルを開く\)** の順に選択し、トランスクリプトを選択します。 新しいトランスクリプト ウィンドウが開き、出力ウィンドウにメッセージ アクティビティが表示されます。 

![エミュレーターでトランスクリプトを読み込む](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>Chatdown でトランスクリプトを作成する

[Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) ツールは、[マークダウン](https://daringfireball.net/projects/markdown/syntax) ファイルを利用してアクティビティ トランスクリプトを生成するトランスクリプト ジェネレーターです。 独自のトランスクリプトをマークダウン形式で完全に作成し、**.chat** ファイルとして保存し、トランスクリプトを生成できます。 ボット開発中、模擬会話シナリオを簡単に作成する際に役立ちます。  

### <a name="prerequisites"></a>前提条件

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) v.4 以降 
- [Node.js](https://nodejs.org/en/)
 
Chatdown は、Node.js を必要とする npm モジュールとして利用できます。 Chatdown をインストールするには、お使いのコンピューターにグローバル インストールします。 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>トランスクリプト ファイルを作成し、読み込む ###

次は **.chat** ファイルの作成方法の例です。 これらのファイルは 2 つの部分を含むマークダウンです。
- 会話の参加者 (ユーザー、ボット) を定義するヘッダー
- 参加者間の往復会話

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[ここをクリック](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples)すると、.chat ファイルのサンプルが他にも表示されます。 

トランスクリプト ファイルを生成するには、CLI で **chatdown** コマンドを使用し、.chat ファイルの名前を入力し、'>' と出力トランスクリプト ファイルの名前を続けます。 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>MSBot ツールでボット リソースを管理する

新しい [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) ツールを使用すると、ボットで使用されるさまざまなサービスに関するメタデータをすべて 1 か所に保存する **.bot** ファイルを作成できます。 このファイルによって、ボットを CLI からサービスに接続できます。このツールは npm モジュールとして利用できます。インストールするには、次のコマンドを実行します。

```
npm install -g msbot 
```
![MSBot CLI ウィンドウ](media/emulator-v4/msbot-cli-window.png)


ボット ファイルを作成するには、CLI から「**msbot init**」と入力し、続けてボットの名前とターゲット URL エンドポイントを入力します。次に例を示します。

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![ボットファイル](media/emulator-v4/botfile-generated.png)

>**注:** このガイドで使用されているボットは、Azure CLI ボット拡張から生成される、単純なエコー ボットです。 [ここをクリック](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli)すると、Azure CLI でボットをビルドする詳しい方法をご覧いただけます。 

.bot ファイルが作成されたら、ボットをエミュレーターに簡単に読み込むことができます。 .bot ファイルは、さまざまなエンドポイントと言語コンポーネントをボットに登録するためにも必要です。 

![ボットの [ファイル] のドロップダウン](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>言語サービスを追加する 

エミュレーターから .bot ファイルに直接、LUIS アプリや QnA ナレッジ ベースを簡単に登録できます。 .bot ファイルが読み込まれたら、エミュレーター ウィンドウの左端にあるサービス ボタンを選択します。 **[サービス]** メニューの下に LUIS、QnA Maker、Dispatch、エンドポイント、Azure Bot Service を追加するためのオプションが表示されます。 

LUIS アプリを追加するには、LUIS メニューで **+** ボタンをクリックし、LUIS アプリの資格情報を入力し、**[送信]** をクリックします。 これにより .bot ファイルに LUIS アプリケーションが登録され、サービスがボット アプリケーションに接続されます。 

![LUIS 接続](media/emulator-v4/emulator-connect-luis-btn.png)

同様に、QnA ナレッジ ベースを追加するには、QnA メニューの **+** ボタンをクリックし、QnA Maker ナレッジ ベースの資格情報を入力し、**[送信]** をクリックします。 これでナレッジ ベースが .bot ファイルに登録され、使用する準備が整います。 

![QnA 接続](media/emulator-v4/emulator-connect-qna-btn.png)

いずれかのサービスが接続されているとき、ライブ チャット ウィンドウに戻り、サービスが接続され、動作していることを確認できます。 

![QnA 接続完了](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>言語サービスを検査する

新しい v4 エミュレーターでは、LUIS と QnA から JSON 応答を検査することもできます。 ボットと言語サービスが接続されているとき、右下の LOG ウィンドウにある **[トレース]** を選択できます。 この新しいツールには、エミュレーターから直接、言語サービスを更新するための機能もあります。 

![LUIS インスペクター](media/emulator-v4/emulator-luis-inspector.png)

LUIS サービスに接続しているとき、トレース リンクに **Luis Trace** と指定されていることに気付きます。 これが選択されると、意図、エンティティ、指定スコアなど、LUIS サービスからの raw 応答が表示されます。 ユーザー発話の意図を再割り当てすることもできます。 

![QnA インスペクター](media/emulator-v4/emulator-qna-inspector.png)

QnA サービスに接続しているとき、ログに **QnA Trace** と表示されます。これが選択されると、そのアクティビティに関連付けられている質問と回答の組みと信頼度スコアをプレビューできます。 ここから、ある回答に対する質問に別の言い回しを追加できます。

[!TIP]
> 上記の機能は v4 SDK ボットでのみ利用できます。 


## <a name="speech-recognition"></a>音声認識
Bot Framework Emulator では [Cognitive Services Speech API](/azure/cognitive-services/Speech/home) による音声認識がサポートされています。 これにより、開発中、エミュレーターの音声を介して音声対応ボット (あるいは Cortana スキル) を訓練できます。 Bot Framework Emulator では、ボット 1 つにつき、1 日 3 時間まで無料で音声認識が提供されます。 

## <a id="ngrok"></a> ngrok をインストールして構成する

Windows を使用し、ファイアウォールまたはその他のネットワーク境界の背後で Bot Framework Emulator を実行しているとき、リモートでホストされているボットに接続するには、トンネリング ソフトウェアの **ngrok** をインストールし、構成する必要があります。 Bot Framework Emulator はトンネリング ソフトウェアの [ngrok][ngrokDownload] ([inconshreveable][inconshreveable] 開発) と密結合され、必要なときに自動起動できます。

Windows に **ngrok** をインストールし、それを使用するようにエミュレーターを構成するには、次の手順を完了してください。 

1. お使いのローカル コンピューターに [ngrok][ngrokDownload] 実行可能ファイルをダウンロードします。

2. エミュレーターの [アプリの設定] ダイアログを開き、ngrok のパスを入力し、ローカル アドレスに対して ngrok をバイパスするかどうかを選択し、**[保存]** をクリックします。

![ngrok パス](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>その他のリソース

Bot Framework Emulator はオープン ソースです。 開発に[貢献][EmulatorGithubContribute]したり、[バグや提案を送信][EmulatorGithubBugs]したりできます。


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md
