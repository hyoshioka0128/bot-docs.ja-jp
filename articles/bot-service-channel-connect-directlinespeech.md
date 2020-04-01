---
title: ボットを Direct Line Speech に接続する
titleSuffix: Bot Service
description: 音声による入出力で対話するために、既存の Bot Framework ボットと Direct Line Speech チャネルの間に待機時間が短く信頼できる接続を確立する手順、およびその接続の概要。
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: travisw
ms.openlocfilehash: 3202c8271ffbdb09be10087861449c36f61cfac5
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250131"
---
# <a name="connect-a-bot-to-direct-line-speech"></a>ボットを Direct Line Speech に接続する

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

クライアント アプリケーションが Direct Line Speech チャネルを介してボットと通信できるように、そのボットを構成できます。

ご自身のボットを構築した後、Direct Line Speech でそれをオンボードすると、[Speech SDK](https://aka.ms/speech-service-docs) を使用して、クライアント アプリケーションとの間に待機時間が短く信頼できる接続を確立できます。 これらの接続は、音声で入出力する会話エクスペリエンス用に最適化されています。 Direct Line Speech、およびクライアント アプリケーションの構築方法の詳細については、[カスタム音声優先仮想アシスタント](https://aka.ms/cognitive-services-voice-assistants)に関するページをご覧ください。

## <a name="add-the-direct-line-speech-channel"></a>Direct Line Speech チャネルを追加する

1. Direct Line Speech チャネルを追加するには、まず、[Azure portal](https://portal.azure.com) でボットを開き、リソースから、 **[Bot Channel Registration]\(ボット チャネル登録\)** リソースを選択します。 構成ブレードの **[チャネル]** をクリックします。

    ![接続先チャネルを選択するための場所の強調表示](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "チャネルの選択")

1. チャネルの選択ページで、`Direct Line Speech` を見つけてクリックし、チャネルを選択します。

    ![Direct Line Speech チャネルの選択](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "Direct Line Speech の接続")

1. 次の図に示すように、Direct Line Speech を構成します。

    ![Direct Line Speech チャネルの選択](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-cognitivesericesaccount-selection.png "Cognitive Services リソースの選択")

    Direct Line Speech チャネルには、Cognitive Services リソース (具体的には、**音声**コグニティブ サービス リソース) が必要です。 既存のリソースを使用することも、新しいものを作成することもできます。 新しい音声リソースを作成するには、次の手順に従います。

    - [Azure portal の[リソースの作成]](https://ms.portal.azure.com/#create/hub) にアクセスします。
    - *Speech* を検索し、ドロップダウン リストから選択します。 次のように表示されます。

        ![音声コグニティブ リソースを作成する](media/voice-first-virtual-assistants/create-speech-cognitive-resource.PNG "音声コグニティブ リソースを作成する")

    - ウィザードの手順に従います。

    詳細については、[Cognitive Services リソースの作成](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)に関する記事を参照してください。

1. 使用条件を確認したら、`Save` をクリックして、チャネルの選択内容を確定します。

    ![Direct Line Speech チャネルの有効化の保存](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "チャネル構成の保存")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Bot Framework Protocol ストリーミング拡張機能を有効にする

Direct Line Speech チャネルがご自身のボットに作成されたら、待機時間が短い最適な対話を実現するために、Bot Framework Protocol ストリーミング拡張機能のサポートを有効にする必要があります。

1. [Azure portal](https://portal.azure.com) でご自身のボットのブレードを開きます (まだ開いていない場合)。

1. ( **[チャネル]** のすぐ下にある) **[Bot Management]\(ボット管理\)** カテゴリの **[設定]** をクリックします。 **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** チェック ボックスをオンにします。

    ![ストリーミング プロトコルの有効化](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "ストリーミング拡張機能のサポートの有効化")

1. ページの上部にある **[保存]** をクリックします。

1. 同じブレードで、 **[App Service の設定]** カテゴリの **[構成]** をクリックします。

    ![App Service の設定への移動](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "App Service の構成")

1. `General settings` をクリックし、`Web socket` サポートを有効にするオプションを選択します。

    ![App Service の WebSocket の有効化](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "WebSocket の有効化")

1. 構成ページの上部にある `Save` をクリックします。

1. お使いのボットに対して Bot Framework Protocol ストリーミング拡張機能が有効になりました。 これでお使いのボット コードを更新し、既存のボット プロジェクトに[ストリーミング拡張機能のサポートを統合](https://aka.ms/botframework/addstreamingprotocolsupport)する準備ができました。

## <a name="adding-protocol-support-to-your-bot"></a>プロトコル サポートをお使いのボットに追加する

Direct Line Speech チャネルが接続され、Bot Framework Protocol ストリーミング拡張機能のサポートが有効になったら、あとはお使いのボットに、最適化された通信をサポートするためのコードを追加するだけです。 [お使いのボットへのストリーミング拡張機能サポートの追加](https://aka.ms/botframework/addstreamingprotocolsupport)に関するページに記載されている手順に従って、Direct Line Speech との完全な互換性を確保します。


