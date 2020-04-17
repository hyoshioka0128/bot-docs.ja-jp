---
title: Cortana スキルのテスト - Bot Service
description: Cortana スキルを呼び出すことで Cortana ボットをテストする方法について説明します。
keywords: Bot Framework SDK, ボットを登録する, Cortana
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 62ed0fb7cb05072024617d65266cf457c554d4c7
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75792771"
---
# <a name="test-a-cortana-skill"></a>Cortana スキルのテスト

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
Bot Framework SDK を使用して Cortana スキルを構築したら、Cortana からそのスキルを呼び出してテストできます。 次の手順では、Cortana スキルをテストするために必要な手順について説明します。

## <a name="register-your-bot"></a>ボットの登録
Azure で Bot Service を使用して[ボットを作成](~/bot-service-quickstart.md)した場合、ボットは既に登録されているため、この手順をスキップできます。

ボットを他の場所にデプロイした場合や、ボットをローカルでテストする必要がある場合は、ボットを Cortana に接続できるように、[登録](bot-service-quickstart-registration.md)する必要があります。 登録プロセス中に、ボットの**メッセージング エンドポイント**を提供する必要があります。 ボットをローカルでテストする場合は、ローカルのボットのエンドポイントを取得するために、[ngrok](http://ngrok.com) などのトンネリング ソフトウェアを実行する必要があります。

## <a name="get-messaging-endpoint-using-ngrok"></a>ngrok を使用したメッセージング エンドポイントの取得

ボットをローカルで実行している場合は、[ngrok](https://ngrok.com) などのトンネリング ソフトウェアを実行して、テストに用に使用するエンドポイントを取得できます。 ngrok を使用してエンドポイントを取得するには、コンソール ウィンドウから次のように入力します。 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

これによって、ポート 3978 で実行される、ご使用のボットに要求を転送する ngrok 転送リンクが構成および表示されます。 転送リンクの URL は、`https://0d6c4024.ngrok.io` のようになります。  このリンクの末尾に `/api/messages` を追加して、`https://0d6c4024.ngrok.io/api/messages` の形式でメッセージング エンドポイントの URL を形成します。 

このエンドポイントの URL をボットの [[設定]](~/bot-service-manage-settings.md) ブレードの **[構成]** セクションに入力します。

## <a name="enable-speech-recognition-priming"></a>音声認識の準備の有効化
ボットで Language Understanding (LUIS) アプリを使用している場合、LUIS アプリケーション ID が、登録済みボット サービスと関連付けられていることを確認します。 これによって、LUIS モデルで定義されている発話をボットが認識できます。 詳細については、[音声認識の準備](~/bot-service-manage-speech-priming.md)に関するページを参照してください。

## <a name="add-the-cortana-channel"></a>Cortana チャネルの追加
Cortana をチャネルとして追加するには、「[Connect a bot to Cortana (Cortana へのボットの接続)](bot-service-channel-connect-cortana.md)」に記載された手順に従います。

## <a name="test-using-web-chat-control"></a>Web チャット コントロールを使用したテスト

ボット サービスで統合 Web チャット コントロールを使用してボットをテストするには、 **[Web チャットでのテスト]** をクリックし、ボットが動作していることを確認するためのメッセージを入力します。

## <a name="test-using-emulator"></a>エミュレーターを使用したテスト

[エミュレーター](~/bot-service-debug-emulator.md)を使用してボットをテストするには、次の操作を行います。

1. ボットを実行します。
2. エミュレーターを開き、必要な情報を入力します。 自分のボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID and MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。 
3. **[接続]** をクリックして、エミュレーターをボットに接続します。
4. メッセージを入力し、ボットが動作していることを確認します。

## <a name="test-using-cortana"></a>Cortana を使用したテスト
Cortana に、呼び出しフレーズを話しかけることで、Cortana スキルを呼び出すことができます。 
1. Cortana を開きます。
2. Cortana 内で Notebook を開き、 **[自己紹介]** をクリックして、Cortana にどのアカウントを使用しているかを確認します。 ボットを登録するときに使用したのと同じ Microsoft アカウントでサインインしていることを確認します。 
   ![Cortana のノートブックにサインインする](~/media/cortana/cortana-notebook.png)
2. Cortana アプリまたは Windows の [Ask me anything]\(何でも聞いて\) 検索ボックスにあるマイク ボタンをクリックし、ボットの[呼び出しフレーズ][InvocationNameGuidelines]を話しかけます。 呼び出しフレーズには "*呼び出し名*" が含まれており、これによって、呼び出すスキルが一意に識別されます。 たとえば、スキルの呼び出し名が "Northwind Photo" の場合は、適切な呼び出しフレーズには、"Northwind Photo に依頼..." または "Northwind Photo に通知…" が含まれます。

   ボットの*呼び出し名*は、Cortana に対してボットを構成するときに指定します。
   ![Cortana チャネルを構成するときに、呼び出し名を入力する](~/media/cortana/cortana-invocation-name-callout.png)

3. Cortana が呼び出しフレーズを認識できた場合、ボットによって Cortana のキャンバスが起動されます。 

## <a name="troubleshoot"></a>トラブルシューティング

Cortana スキルの起動に失敗した場合は、次のことを確認してください。
* Bot Framework ポータルでボットを登録するときに使用したのと同じ Microsoft アカウントで Cortana にサインインしていることを確認します。
* **[Web チャットでのテスト]** をクリックして **[チャット]** ウィンドウを開き、メッセージを入力して、ボットが機能しているかどうかを確認します。
* 呼び出し名が[ガイドライン][InvocationNameGuidelines]を満たしているかどうかを確認してください。 呼び出し名が 4 つ以上の単語で構成されている、発音が難しい、または他の単語と音が似ている場合は、Cortana で認識することが困難なことがあります。
* スキルが LUIS モデルを使用している場合、[音声プライミングを構成する](~/bot-service-manage-speech-priming.md)を確認してください。

追加のトラブルシューティングのヒントと、Cortana のダッシュボードでスキルのデバッグを有効にする方法については、[Cortana スキルのデバッグの有効化][Cortana-TestBestPractice]に関するページを参照してください。 


## <a name="next-steps"></a>次のステップ

Cortana スキルをテストして期待どおりに動作することを確認したら、ベータ テスト担当者のグループにデプロイするか、一般に公開することができます。 詳細については、「[Publishing Cortana Skills][Cortana-Publish]」 (Cortana スキルの公開) を参照してください。

## <a name="additional-resources"></a>その他のリソース
* [Cortana スキル キット][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/cortana

[CortanaSpecificEntities]: https://aka.ms/cortana-channel-data
[CortanaAuth]: https://aka.ms/add-auth-cortana-skill

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill
