---
title: Bot Builder の基礎 | Microsoft Docs
description: Bot Builder SDK の基本構造。
keywords: ターン コンテキスト, ボット構造, 入力の受信
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 34564b411f911ae82197d5a34cb954a103abe70b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905416"
---
# <a name="basic-bot-structure"></a>基本的なボットの構造

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot Service および Bot Builder SDK は、ボットの構築とデバッグに役立つライブラリ、サンプル、およびツールを提供します。 しかし、それらの詳細を論じる前に、ボットの基本構造と、各要素の連携のしくみを理解することが重要です。 これらの概念は、どのプログラミング言語を選択するかにかかわらず適用されます。 より詳細なコンテンツへのリンクも提供していますが、この記事では、ボットの動作方法に関する最初の心構えの枠組みを提供します。

まっさらの状態からボットの基本構造を見ていきましょう。

## <a name="creation-of-your-bot"></a>ボットの作成

ボットは、[Azure portal](~/bot-service-quickstart.md) 内で、[Visual Studio](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) で、あるいは [JavaScript](~/javascript/bot-builder-javascript-quickstart.md)、[Java](~/java/bot-builder-java-quickstart.md)、[Python](~/python/bot-builder-python-quickstart.md) のコマンドライン ツールを使用するなど、さまざまな方法で作成できます。 作成したボットはローカル コンピューター、Azure、または別のクラウド サービスで実行できます。 実行される場所や構築方法にかかわらず、すべてのボットは非常に似通った形で機能します。

## <a name="interaction-with-your-bot"></a>ボットとの対話

ボットは本質的に、Web サイトやアプリのような目に見える UI を持たない代わりに、ユーザーが[会話](~/v4sdk/bot-concepts.md#activities-and-conversations)を通じてボットと対話します。 ボットとの接続に使用されるアプリケーション ([チャネル](~/v4sdk/bot-concepts.md)と呼ばれますが、ここでは説明しません) に応じて、ユーザーとボットの間で特定の情報がやり取りされます。

情報の各単位はボット内で**アクティビティ**と呼ばれ、アクティビティはさまざまな形態をとる可能性があります。 アクティビティには、**メッセージ**と呼ばれるユーザーからのコミュニケーションと、他のいくつかの[アクティビティ タイプ](~/bot-service-activities-entities.md)にまとめられた追加情報の両方が含まれます。 この追加情報には、新しい参加者が会話に参加または退出した時刻、会話が終了した時刻などが含まれます。ユーザーの接続から発生するそれらの種類の通信は基盤となるシステムによって送信され、ユーザーは何もする必要はありません。

ボットは通信を受け取り、適切なタイプのアクティビティ オブジェクトにラップして、ボットのコードに渡します。 他のすべてのアクティビティ タイプも有用な情報を提供しますが、最も興味深くて最も一般的なアクティビティはユーザーからの**メッセージ** アクティビティです。

ボットが受け取る各アクティビティはターンを開始します。ターンについては後で詳しく説明します。

## <a name="receiving-user-input"></a>ユーザー入力の受信

ユーザーからメッセージ アクティビティを受け取ったら、ユーザーから送信されてきた内容を理解する必要があります。 最も簡単な方法は、受信メッセージのテキストを単純に文字列とマッチングすることです。 どのような文字列であるかに基づいて、またボットが達成しようとしている目標に応じて、何らかの処理を行うことを選択できます。 これにはユーザーへの応答、変数またはリソースの更新、[ストレージ](~/v4sdk/bot-builder-storage-concept.md)への保存、または同様の処理が含まれます。

[LUIS](~/v4sdk/bot-builder-concept-luis.md) または [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) の使用など、ユーザー入力を認識するためのより複雑な方法もありますが、文字列のマッチングは最も単純です。

## <a name="defining-a-turn"></a>ターンの定義

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

アクティビティの処理は、「[アクティビティの処理](~/v4sdk/bot-builder-concept-activity-processing.md)」で詳しく説明する**アダプター**によって管理されます。 アクティビティを受け取ると、アダプターはアクティビティについての情報を提供する**ターン コンテキスト**を作成し、処理中の現在のターンにコンテキストを付与します。 ターン コンテキストはターンの存続期間中は存在し、その後破棄され、ターンの終わりを示します。

[ターン コンテキスト](~/v4sdk/bot-builder-concept-activity-processing.md#turn-context)にはいくつかの情報が含まれており、ボットのすべてのレイヤーを通して同じオブジェクトが使用されます。 これが便利な理由は、このターン コンテキスト オブジェクトを使用して、ターンで後から必要になる情報を保存しておくことができ、またそうすることが推奨されているためです。

> [!IMPORTANT]
> すべての**ターン**は互いに独立しており、単独で実行され、重複する可能性があります。 ボットは、さまざまなチャネルのさまざまなユーザーからの複数のターンを同時に処理する場合があります。 各ターンには独自のターン コンテキストがありますが、いくつかの状況で発生する複雑さを考慮する価値はあります。

## <a name="where-to-go-from-here"></a>次にすること

この記事では、[アクティビティの処理](~/v4sdk/bot-builder-concept-activity-processing.md)方法、さまざまな[会話の種類](~/v4sdk/bot-builder-conversations.md)、よりインテリジェントな会話のための[会話状態](~/v4sdk/bot-builder-storage-concept.md)の追跡など、多くの詳細には触れていません。 残りの概念トピックは基礎に立脚しており、ボットと Azure Bot Service の両方を理解するために必要なその他のアイデアを扱っています。 次の手順のセクションから知識を強化する、興味のある項目にジャンプする、[クイック スタート](~/bot-service-quickstart.md)で最初のボットを構築してみる、あるいはボットの[開発](~/v4sdk/bot-builder-howto-send-messages.md)方法を学ぶことができます。

## <a name="next-steps"></a>次の手順

Bot Connector Service により、さまざまなプラットフォームでボットを使用してユーザーとコミュニケーションができます。

> [!div class="nextstepaction"]
> [チャネルと Bot Connector サービス](~/v4sdk/bot-concepts.md)
