---
title: ボットを Direct Line Speech に接続する (プレビュー)
titleSuffix: Bot Service
description: 音声による入出力で対話するために、既存の Bot Framework ボットと Direct Line Speech チャネルの間に待機時間が短く信頼できる接続を確立する手順、およびその接続の概要。
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: f38caad2a1b09e8f07e5fe8c7ea7bf7e2b2dfd6f
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/03/2019
ms.locfileid: "68756830"
---
# <a name="connect-a-bot-to-direct-line-speech-preview"></a>ボットを Direct Line Speech に接続する (プレビュー)

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

クライアント アプリケーションが Direct Line Speech チャネルを介してボットと通信できるように、そのボットを構成できます。

ご自身のボットを構築した後、Direct Line Speech でそれをオンボードすると、[Speech SDK](https://aka.ms/speech/sdk) を使用して、クライアント アプリケーションとの間に待機時間が短く信頼できる接続を確立できます。 これらの接続は、音声で入出力する会話エクスペリエンス用に最適化されています。 Direct Line Speech、およびクライアント アプリケーションの構築方法の詳細については、[カスタムの音声優先仮想アシスタント](https://aka.ms/bots/speech/va)に関するページをご覧ください。  

## <a name="sign-up-for-direct-line-speech-preview"></a>Direct Line Speech プレビューにサインアップする

Direct Line Speech は現在プレビュー段階です。[Azure portal](https://portal.azure.com) でクイック サインアップが必要です。 詳細は以下を参照してください。 承認されると、チャネルにアクセスできます。

## <a name="add-the-direct-line-speech-channel"></a>Direct Line Speech チャネルを追加する

1. ブラウザーで [Azure portal](https://portal.azure.com) に移動します。 リソースから、**Bot Channel Registration** リソースを選択します。 構成ブレードの *[Bot management]\(ボットの管理\)* セクションで、 **[Channels]\(チャネル\)** をクリックします。

    ![接続先チャネルを選択するために場所が強調表示されている](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "チャネルの選択")

1. チャネルの選択ページで、`Direct Line Speech` を見つけてクリックし、チャネルを選択します。

    ![Direct Line Speech の選択](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "Direct Line Speech への接続")

1. アクセスが承認されていない場合は、アクセスを要求するページが表示されます。 必要な情報を入力し、[要求] をクリックします。 確認ページが表示されます。 要求が承認待ちの間は、このページの先に進むことはできません。   

1. アクセスが承認されると、Direct Line Speech の構成ページが表示されます。 使用条件を確認したら、`Save` をクリックして、チャネルの選択内容を確定します。

    ![Direct Line Speech チャネルの有効化の保存](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "チャネル構成の保存")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Bot Framework Protocol ストリーミング拡張機能を有効にする

Direct Line Speech チャネルがご自身のボットに作成されたら、待機時間が短い最適な対話を実現するために、Bot Framework Protocol ストリーミング拡張機能のサポートを有効にする必要があります。

1. **Bot Channel Registration** リソースの構成ブレードで、 **[Bot Management]\(ボットの管理\)** カテゴリ ( **[Channels]\(チャネル\)** のすぐ下にある) の **[Settings]\(設定\)** をクリックします。 **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** チェック ボックスをオンにします。

    ![ストリーミング プロトコルを有効にする](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "ストリーミング拡張機能のサポートを有効にする")

1. ページの上部にある **[保存]** をクリックします。

1. リソースから、**App Service** リソースを選択します。 表示されたブレードで、 **[設定]** カテゴリの **[構成]** をクリックします。

    ![App Service の設定に移動する](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "App Service を構成する")

1. `General settings` タブをクリックし、`Web socket` サポートを有効にするオプションを選択します。

    ![App Service の WebSocket を有効にする](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "WebSocket を有効にする")

1. 構成ページの上部にある `Save` をクリックします。

1. お使いのボットに対して Bot Framework Protocol ストリーミング拡張機能が有効になりました。 これでお使いのボット コードを更新し、既存のボット プロジェクトに[ストリーミング拡張機能のサポートを統合](https://aka.ms/botframework/addstreamingprotocolsupport)する準備ができました。

## <a name="manage-secret-keys"></a>秘密鍵を管理する

Direct Line Speech チャネルを介してクライアント アプリケーションとお使いのボットを接続するには、クライアント アプリケーションにチャネル シークレットが必要になります。 チャネルの選択を保存したら、次の手順に従ってこれらの秘密鍵を取得できます。

1. リソースから、**Bot Channel Registration** リソースを選択します。 構成ブレードの *[Bot management]\(ボットの管理\)* セクションで、 **[Channels]\(チャネル\)** をクリックします。
1. Direct Line Speech の **[編集]** リンクをクリックします。

    ![Direct Line Speech の秘密鍵の取得](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys1.png "Direct Line Speech の秘密鍵の取得")

    次のウィンドウが表示されます。

    ![Direct Line Speech の秘密鍵の取得](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys.png "Direct Line Speech の秘密鍵の取得")
1. アプリケーションで使用する鍵を表示してコピーします。

## <a name="adding-protocol-support-to-your-bot"></a>プロトコル サポートをお使いのボットに追加する

Direct Line Speech チャネルが接続され、Bot Framework Protocol ストリーミング拡張機能のサポートが有効になったら、あとはお使いのボットに、最適化された通信をサポートするためのコードを追加するだけです。 [お使いのボットへのストリーミング拡張機能サポートの追加](https://aka.ms/botframework/addstreamingprotocolsupport)に関するページに記載されている手順に従って、Direct Line Speech との完全な互換性を確保します。

## <a name="known-issues"></a>既知の問題

このサービスはプレビュー段階なので、変更されることがあり、これがボット開発および全体的なパフォーマンスに影響を及ぼす可能性があることに注意してください。 既知の問題の一覧を次に示します。 

1. 現在サービスは [Azure リージョン](https://azure.microsoft.com/global-infrastructure/regions/)米国西部 2 にデプロイされています。 他のリージョンについてはまもなく展開され、すべてのお客様が、短い待機時間でボットと対話するメリットを得られるようになります。

1. [serviceUrl](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#service-url) など、コントロール フィールドに対する軽微な変更が行われます

1. 会話の開始と終了の通知に使用される [conversationUpdate](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation-update-activity) アクティビティと [endOfCoversation](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#end-of-conversation-activity) アクティビティは、ようこそメッセージの生成によく使用され、他のチャネルとの整合性を確保するために更新されます

1. [SigninCard](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards?view=azure-bot-service-4.0) は、チャネルではまだサポートされていません 
