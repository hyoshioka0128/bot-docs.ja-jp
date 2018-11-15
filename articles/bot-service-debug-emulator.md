---
title: Bot Framework Emulator を使用してボットをテストし、デバッグする | Microsoft Docs
description: Bot Framework Emulator デスクトップ アプリケーションを利用し、ボットを検査、試験、デバッグする方法について説明します。
keywords: トランスクリプト, msbot ツール, 言語サービス, 音声認識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/13/2018
ms.openlocfilehash: f3a6a57a5fd01061493e5c216875f0c4210483f6
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645612"
---
# <a name="debug-with-the-emulator"></a>エミュレーターを使用したデバッグ

Bot Framework Emulator は、ローカルでもリモートでも、ボットをテストし、デバッグできるデスクトップ アプリケーションです。 このエミュレーターを使用し、ボットとチャットしたり、ボットが送受信するメッセージを検査したりできます。 このエミュレーターでは、Web チャット UI に表示されるときと同じようにメッセージが表示され、ボットとメッセージを交換するときの JSON の要求と応答がログに記録されます。 自分のボットをクラウドにデプロイする前に、エミュレーターを使用してローカルで実行し、テストしてください。 まだ Azure Bot Service で[作成](./bot-service-quickstart.md)していない場合でも、何らかのチャネルで実行するように構成していない場合でも、エミュレーターを使用して自分のボットをテストできます。

## <a name="prerequisites"></a>前提条件
- [エミュレーター](https://aka.ms/Emulator-wiki-getting-started)をインストールする
- [ngrok][ngrokDownload] トンネリング ソフトウェアをインストールする

## <a name="connect-to-a-bot-running-on-localhost"></a>localhost で実行されているボットに接続する

ローカルで実行されている bot に接続するには、**[Open bot]\(ボットを開く\)** をクリックして .bot ファイルを選択します。 

![エミュレーター UI](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>インスペクターを利用してメッセージ アクティビティの詳細を表示する

ボットにメッセージを送信すると、ボットが応答するはずです。 会話ウィンドウ内でメッセージ吹き出しをクリックし、ウィンドウの右にある **INSPECTOR** 機能を使用して raw JSON アクティビティを検査できます。 メッセージ吹き出しは選択すると黄色になり、アクティビティ JSON オブジェクトがチャット ウィンドウの左に表示されます。 JSON 情報には、チャネル ID、アクティビティの種類、会話 ID、テキスト メッセージ、エンドポイント URL など、重要なメタデータが含まれます。ユーザーから送信されたアクティビティやボットの応答アクティビティを検査できます。 

![エミュレーターのメッセージ アクティビティ](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>ボット トランスクリプトと共に会話を保存し、読み込む

エミュレーターのアクティビティはトランスクリプトとして保存できます。 開いているライブ チャット ウィンドウで、**[Save Transcript As]\(トランスクリプトを名前を付けて保存\)** を選択してトランスクリプト ファイルを保存します。 **[やり直す]** ボタンを使用し、いつでも会話を消去し、ボットへの接続を再起動できます。  

![エミュレーターでトランスクリプトを保存する](media/emulator-v4/emulator-live-chat.png)

トランスクリプトを読み込むには、**[ファイル] > [Open Transcript File]\(トランスクリプト ファイルを開く\)** の順に選択し、トランスクリプトを選択します。 新しいトランスクリプト ウィンドウが開き、出力ウィンドウにメッセージ アクティビティが表示されます。 

![エミュレーターでトランスクリプトを読み込む](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>サービスの追加 

エミュレーターから、.bot ファイルに直接、LUIS アプリ、QnA ナレッジベース、ディスパッチ モデルを簡単に追加できます。 .bot ファイルが読み込まれたら、エミュレーター ウィンドウの左端にあるサービス ボタンを選択します。 **[サービス]** メニューの下に LUIS、QnA Maker、ディスパッチ を追加するためのオプションが表示されます。 

サービス アプリを追加するには、**+** ボタンをクリックして、追加するサービスを選択します。 Azure portal にサインインしてボット ファイルにサービスを追加し、そのサービスをボット アプリケーションに接続するよう求められます。 

![LUIS 接続](media/emulator-v4/emulator-connect-luis-btn.png)

いずれかのサービスが接続されているとき、ライブ チャット ウィンドウに戻り、サービスが接続され、動作していることを確認できます。 

![QnA 接続完了](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>サービスの検査

新しい v4 エミュレーターでは、LUIS と QnA から JSON 応答を検査することもできます。 ボットと言語サービスが接続されているとき、右下の LOG ウィンドウにある **[トレース]** を選択できます。 この新しいツールには、エミュレーターから直接、言語サービスを更新するための機能もあります。 

![LUIS インスペクター](media/emulator-v4/emulator-luis-inspector.png)

LUIS サービスに接続しているとき、トレース リンクに **Luis Trace** と指定されていることに気付きます。 これが選択されると、意図、エンティティ、指定スコアなど、LUIS サービスからの raw 応答が表示されます。 ユーザー発話の意図を再割り当てすることもできます。 

![QnA インスペクター](media/emulator-v4/emulator-qna-inspector.png)

QnA サービスに接続しているとき、ログに **QnA Trace** と表示されます。これが選択されると、そのアクティビティに関連付けられている質問と回答の組みと信頼度スコアをプレビューできます。 ここから、ある回答に対する質問に別の言い回しを追加できます。

## <a name="configure-ngrok"></a>ngrok の構成

Windows を使用し、ファイアウォールまたはその他のネットワーク境界の背後で Bot Framework Emulator を実行しているとき、リモートでホストされているボットに接続するには、トンネリング ソフトウェアの **ngrok** をインストールし、構成する必要があります。 Bot Framework Emulator はトンネリング ソフトウェアの ngrok ([inconshreveable][inconshreveable] 開発) と密結合され、必要なときに自動起動できます。

**[Emulator Settings]\(エミュレーターの設定\)** を開き、ngrok のパスを入力し、ローカル アドレスに対して ngrok をバイパスするかどうかを選択し、**[保存]** をクリックします。

![ngrok パス](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>その他のリソース

Bot Framework Emulator はオープン ソースです。 開発に[貢献][EmulatorGithubContribute]したり、[バグや提案を送信][EmulatorGithubBugs]したりできます。



[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
