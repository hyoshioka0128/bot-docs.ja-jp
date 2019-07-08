---
title: Bot Framework Emulator を使用してボットをテストし、デバッグする | Microsoft Docs
description: Bot Framework Emulator デスクトップ アプリケーションを利用し、ボットを検査、試験、デバッグする方法について説明します。
keywords: トランスクリプト, msbot ツール, 言語サービス, 音声認識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 0e548700e81fff5029031fd1e349cc75d9d0bc7a
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464642"
---
# <a name="debug-with-the-emulator"></a>エミュレーターを使用したデバッグ

Bot Framework Emulator は、ローカルでもリモートでも、ボットをテストし、デバッグできるデスクトップ アプリケーションです。 このエミュレーターを使用し、ボットとチャットしたり、ボットが送受信するメッセージを検査したりできます。 このエミュレーターでは、Web チャット UI に表示されるときと同じようにメッセージが表示され、ボットとメッセージを交換するときの JSON の要求と応答がログに記録されます。 自分のボットをクラウドにデプロイする前に、エミュレーターを使用してローカルで実行し、テストしてください。 まだ Azure Bot Service で[作成](./bot-service-quickstart.md)していない場合でも、何らかのチャネルで実行するように構成していない場合でも、エミュレーターを使用して自分のボットをテストできます。

## <a name="prerequisites"></a>前提条件
- [エミュレーター](https://aka.ms/Emulator-wiki-getting-started)をインストールする

## <a name="connect-to-a-bot-running-on-localhost"></a>localhost で実行されているボットに接続する

![エミュレーター UI](media/emulator-v4/emulator-welcome.png)

ローカルで実行されているボットに接続するには、 **[Open bot]\(ボットを開く\)** をクリックするか、事前構成済みファイル (.bot ファイル) を選択します。 ご自身のボットへの接続に構成ファイルは必要はありませんが、ボットに構成ファイルがある場合、エミュレーターは引き続きそのファイルで動作します。 お使いのボットが [Microsoft アカウント (MSA) の資格情報](#use-bot-credentials)で実行されている場合は、その資格情報も入力します。

![エミュレーター UI](media/emulator-v4/emulator-open-bot.png)

### <a name="use-bot-credentials"></a>ボットの資格情報を使用

ボットを開いたときに、ボットが資格情報を使用して実行されている場合は、**Microsoft アプリ ID** および **Microsoft アプリ パスワード**を設定します。 Azure Bot Service でボットを作成した場合、資格情報はそのボットの App Service ( **[設定] -> [構成]** セクションの下) で使用できます。 値がわからない場合は、ローカルで実行されているボットの構成ファイルからそれを削除してから、ボットをエミュレーターで実行できます。 ボットがこれらの設定で実行されていない場合は、その設定でエミュレーターを実行する必要もありません。 

## <a name="view-detailed-message-activity-with-the-inspector"></a>インスペクターを利用してメッセージ アクティビティの詳細を表示する

ボットにメッセージを送信すると、ボットが応答するはずです。 会話ウィンドウ内でメッセージ吹き出しをクリックし、ウィンドウの右にある **INSPECTOR** 機能を使用して raw JSON アクティビティを検査できます。 メッセージ吹き出しは選択すると黄色になり、アクティビティ JSON オブジェクトがチャット ウィンドウの左に表示されます。 JSON 情報には、チャネル ID、アクティビティの種類、会話 ID、テキスト メッセージ、エンドポイント URL など、重要なメタデータが含まれます。ユーザーから送信されたアクティビティやボットの応答アクティビティを検査できます。 

![エミュレーターのメッセージ アクティビティ](media/emulator-v4/emulator-view-message-activity-03.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>ボット トランスクリプトと共に会話を保存し、読み込む

エミュレーターのアクティビティはトランスクリプトとして保存できます。 開いているライブ チャット ウィンドウで、 **[Save Transcript As]\(トランスクリプトを名前を付けて保存\)** を選択してトランスクリプト ファイルを保存します。 **[やり直す]** ボタンを使用し、いつでも会話を消去し、ボットへの接続を再起動できます。  

![エミュレーターでトランスクリプトを保存する](media/emulator-v4/emulator-save-transcript.png)

トランスクリプトを読み込むには、 **[ファイル] > [Open Transcript File]\(トランスクリプト ファイルを開く\)** の順に選択し、トランスクリプトを選択します。 新しいトランスクリプト ウィンドウが開き、出力ウィンドウにメッセージ アクティビティが表示されます。 

![エミュレーターでトランスクリプトを読み込む](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>サービスの追加 

エミュレーターからボットに直接 LUIS アプリ、QnA ナレッジベース、ディスパッチ モデルを簡単に追加できます。 ボットが読み込まれたら、エミュレーター ウィンドウの左端にあるサービス ボタンを選択します。 **[サービス]** メニューの下に LUIS、QnA Maker、ディスパッチ を追加するためのオプションが表示されます。 

サービス アプリを追加するには、 **+** ボタンをクリックして、追加するサービスを選択します。 Azure portal にサインインしてボット ファイルにサービスを追加し、そのサービスをボット アプリケーションに接続するよう求められます。 

> [!IMPORTANT]
> サービスの追加は、`.bot` 構成ファイルを使用している場合にのみ機能します。 サービスは個別に追加する必要があります。 詳細については、「[ボット リソースの管理](v4sdk/bot-file-basics.md)」または追加しようとしているサービスに関する個別のハウツー記事をご覧ください。
>
> `.bot` ファイルを使用していない場合、(ご自身のボットでサービスが使用されていても) 左側のウィンドウにはサービスは表示されず、"*サービスが使用できない*" ことを示すメッセージが表示されます。

![LUIS 接続](media/emulator-v4/emulator-connect-luis-btn.png)

いずれかのサービスが接続されているとき、ライブ チャット ウィンドウに戻り、サービスが接続され、動作していることを確認できます。 

![QnA 接続完了](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>サービスの検査

新しい v4 エミュレーターでは、LUIS と QnA から JSON 応答を検査することもできます。 ボットと言語サービスが接続されているとき、右下の LOG ウィンドウにある **[トレース]** を選択できます。 この新しいツールには、エミュレーターから直接、言語サービスを更新するための機能もあります。 

![LUIS インスペクター](media/emulator-v4/emulator-luis-inspector.png)

LUIS サービスに接続しているとき、トレース リンクに **Luis Trace** と指定されていることに気付きます。 これが選択されると、意図、エンティティ、指定スコアなど、LUIS サービスからの raw 応答が表示されます。 ユーザー発話の意図を再割り当てすることもできます。 

![QnA インスペクター](media/emulator-v4/emulator-qna-inspector.png)

QnA サービスに接続しているとき、ログに **QnA Trace** と表示されます。これが選択されると、そのアクティビティに関連付けられている質問と回答の組みと信頼度スコアをプレビューできます。 ここから、ある回答に対する質問に別の言い回しを追加できます。

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

## <a name="login-to-azure"></a>Azure にログインする

Emulator を使って、ご自身の Azure アカウントにログインできます。 これは、ご自身のボットが依存するサービスを追加および管理するときに特に役に立ちます。 Emulator を使用して管理できるサービスの詳細については、[上](#add-services)の説明を参照してください。

### <a name="to-login"></a>ログインする

![Azure ログイン](media/emulator-v4/emulator-azure-login.png)

ログインするには
- [ファイル]、[Sign in with Azure]\(Azure を使ってサインインする\) の順にクリックします
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

Bot Framework Emulator はオープン ソースです。 [投稿][EmulatorGithubContribute] to the development and [submit bugs and suggestions][EmulatorGithubBugs]できます。

トラブルシューティングについては、[一般的な問題のトラブルシューティング](bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。

## <a name="next-steps"></a>次の手順

会話をトランスクリプト ファイルに保存すると、デバッグのために一連のやり取りの下書きをすばやく作成し、そのやり取りを再生できます。

> [!div class="nextstepaction"]
> [トランスクリプト ファイルを使用してお使いのボットをデバッグする](~/v4sdk/bot-builder-debug-transcript.md)

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
