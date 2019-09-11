---
title: 音声プライミングを構成する | Microsoft Docs
description: Azure portal を使用して、ボット サービス用の音声プライミングを構成する方法について説明します。
keywords: 音声プライミング, 音声認識, LUIS
author: v-royhar
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 5cb993392c57db63074fd5354ff85616d5004133
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298032"
---
# <a name="configure-speech-priming"></a>音声プライミングを構成する

音声プライミングで、ボットでよく使用される話し言葉やフレーズの認識が改善されます。 Web チャットと Cortana チャネルを使用する音声対応ボットの場合、音声プライミングでは、Language Understanding (LUIS) アプリで指定された例を使用して、重要な単語の音声認識精度を向上します。

ボットは既に LUIS アプリと統合されている場合があります。また、音声プライミングのためにボットを関連付ける LUIS アプリを作成することもできます。 LUIS アプリには、ユーザーがボットに話しかけると予想される言葉の例が含まれています。 ボットに認識させる重要な単語には、エンティティとラベルを付ける必要があります。 たとえば、チェスのボットでは、ユーザーが「Move knight」(ナイトを動かして) と言った場合に、「Move night」(夜を動かして) と解釈されないようにします。 LUIS アプリには、"knight" (ナイト) がエンティティとしてラベル付けした例を含める必要があります。

> [!NOTE]
> Web チャット チャネルで音声プライミングを使用するには、Bing Speech サービスを使用する必要があります。 Bing Speech サービスを使用する方法については、[Web チャット チャネルで音声を有効にする方法](~/bot-service-channel-connect-webchat-speech.md)に関するページを参照してください。

> [!IMPORTANT]
> 音声プライミングは、Cortana チャネルまたは Web チャット チャネル用に構成されたボットにのみ適用されます。

> [!IMPORTANT]
> プライミングは米国リージョン以外の LUIS アプリ (eu.luis.ai、au.luis.ai など) ではサポートされていません。

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>ボットが使用する LUIS アプリの一覧を変更する

ボットで Bing Speech に使用する LUIS アプリの一覧を変更するには、次の手順を実行します。

1. ボット サービス ブレードの **[Speech priming]\(音声プライミング\)** をクリックします。 使用できる LUIS アプリの一覧が表示されます。
2. Bing Speech で使用する LUIS アプリを選択します。
 
    a. 一覧から LUIS アプリを選択するには、チェックボックスが表示されるまで LUIS モデルにマウス カーソルを移動し、チェックボックスをオンにします。
     
    b. 一覧にない LUIS アプリを選択するには、一番下までスクロールし、テキスト ボックスに LUIS アプリケーション ID の GUID を入力します。
     
3. **[保存]** をクリックして、ボットの Bing Speech に関連付けられている LUIS アプリの一覧を保存します。

![[Speech priming]\(音声プライミング\) パネル](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>その他のリソース

- Web チャットで音声を有効にする方法については、「[Enable speech in the Web Chat channel](~/bot-service-channel-connect-webchat-speech.md)」(Web チャット チャネルで音声を有効にする) を参照してください。
- 音声プライミングの詳細については、「[Speech Support in Bot Framework – Web Chat to Directline, to Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/)」(Bot Framework での音声のサポート - Web チャットから Direct Line、そして Cortana へ) を参照してください。
- LUIS アプリの詳細については、「[Language Understanding Intelligent Service](https://www.luis.ai)」を参照してください。
