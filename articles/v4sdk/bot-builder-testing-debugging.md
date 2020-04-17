---
title: デバッグのガイドライン - Bot Service
description: ご利用のボットをデバッグする方法について説明します。
keywords: ボットのデバッグ, botframework デバッグ
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d10f60bb4291cbd53eb526423e5bdcd711afa8f2
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791184"
---
# <a name="debugging-guidelines"></a>デバッグのガイドライン

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットは様々なパーツが連携する複雑なアプリです。 そのため、他の複雑なアプリの場合と同様、興味深いバグが発生したり、期待どおりにボットが動作しなかったりすることがあります。

ボットのデバッグは、場合によっては難しい作業になります。 開発者それぞれが独自のやり方でそのタスクを実行しています。以下のガイドラインでは、大部分のボットに適用できる推奨事項を紹介します。

お使いのボットが意図したとおりに動作するように見えるのを確認したら、次はそのボットをチャネルに接続します。 これを行うには、ご自身のボットをステージング サーバーに展開し、そのボットの接続先となる独自のダイレクト ライン クライアントを作成します。
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

独自のクライアントを作成すると、チャネルの内部動作を定義できるほか、ご自身のボットが特定のアクティビティの交換に対してどのように応答するかを具体的にテストできます。 そのクライアントに接続されたら、テストを実行してボットの状態を設定し、お使いの機能を確認します。 お使いのボットで音声などの機能が使用されている場合は、これらのチャネルを使用して、その機能を確認する手段を提供できます。

ここで Azure portal からエミュレーターと Web チャットの両方を使用すると、さまざまなチャネルと対話しているときの、ご利用のボットの動作に関するより詳細な分析情報が得られます。

お使いのボットのデバッグは、他のマルチスレッド アプリと同様に動作し、ブレークポイントを設定する機能が用意されています。また、イミディエイト ウィンドウなどの機能を使用することもできます。 

ボットはイベント ドリブン方式のプログラミング パラダイムに従っています。このパラダイムに詳しくない場合、これを合理的に説明するのは困難です。 お使いのボットがステートレスかつマルチ スレッドであり、非同期/待機の呼び出しを処理するという考え方は、予期しないバグにつながることあります。 こうしたボットのデバッグは他のマルチスレッド アプリと同様に動作しますが、ここでは役に立つ提案、ツール、およびリソースをいくつか紹介します。

## <a name="understanding-bot-activities-with-the-emulator"></a>エミュレーターを使用したボット アクティビティの理解

お使いのボットでは、通常の "_メッセージ_" アクティビティだけでなく、さまざまな種類の[アクティビティ](bot-builder-basics.md#the-activity-processing-stack)が処理されます。 [エミュレーター](../bot-service-debug-emulator.md)を使用すると、それがどのようなアクティビティで、いつ発生し、どのような情報が含まれるかを確認できます。 これらのアクティビティを理解すると、ボットのコードを効率的に書いたり、ボットが送受信しているアクティビティが期待どおりのものかを確認したりするうえで役立ちます。

## <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>ユーザーとの対話のトランスクリプトの保存と取得

Azure BLOB のトランスクリプト ストレージは、ユーザーとボットの間の対話が含まれる[トランスクリプトを格納して保存](bot-builder-howto-v4-storage.md)できる、特殊化されたリソースを提供します。  

さらに、ユーザー入力の対話が格納された後は、Azure の "_ストレージ エクスプローラー"_ を使用して、BLOB のトランスクリプト ストア内に格納されたトランスクリプトに含まれるデータを手動で確認できます。 次の例では、"_mynewtestblobstorage_" の設定から "_ストレージ エクスプローラー_" を開きます。 保存されたユーザー入力を開くには、  [BLOB コンテナー] > [ChannelId] > [TranscriptId] > [ConversationId] の順に選択します。

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

これにより、JSON 形式で格納された会話のユーザー入力が開きます。 ユーザー入力はキーとなる "_text_" と共に保持されます。

## <a name="how-middleware-works"></a>ミドルウェアのしくみ

特に実行の継続やショートサーキットに関して言えば、初めて使う[ミドルウェア](bot-builder-concept-middleware.md)は直感的とは言えないでしょう。 ミドルウェアは、ボット ロジックに実行が渡されるタイミングをディクテーションする `next()` デリゲートへの呼び出しを使用して、ターンの立ち上がりまたは立ち下がりで実行できます。 

複数のミドルウェアを使用している場合は、お使いのパイプラインの方向付けに従って、デリゲートによって別のミドルウェアに実行が渡されることがあります。 詳細については、[ボットのミドルウェア パイプライン](bot-builder-concept-middleware.md#the-bot-middleware-pipeline)に関する記事をご覧ください。この考え方を、さらに明確に理解するうえで役立ちます。

`next()` デリゲートが呼び出されない場合、それは[ショート サーキット ルーティング](bot-builder-concept-middleware.md#short-circuiting)と呼ばれます。 ミドルウェアが現在のアクティビティを満たしている場合、そして実行を渡す必要ないと判断した場合に、これが発生します。 

ミドルウェアでショートサーキットが発生するタイミングと理由を理解すると、どのミドルウェアをパイプラインの先頭にするかを指定するときに役立ちます。 また、予期される結果を理解することは、SDK または他の開発者によって提供される組み込みのミドルウェアには特に重要です。 組み込みのミドルウェアを使用する前に、まず独自のミドルウェアを作成して少し実験してみると役立つ場合もあります。

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

## <a name="understanding-state"></a>状態の理解

特に複雑なタスクの場合、状態を追跡することが、お使いのボットにとっては重要です。 一般的に、ベスト プラクティスは、アクティビティをできるだけ迅速に処理し、状態が永続化されるようにその処理を完了させることです。 複数のアクティビティがほぼ同時にご使用のボットに送信されるため、非同期アーキテクチャが原因で、非常に紛らわしいバグが発生することがあります。

重要なのは、自分が意図したとおりに状態が永続化されているかどうかを確認することです。 永続化の状態が存在する場所によっては、[Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/local-emulator) および [Azure Table Storage](https://docs.microsoft.com/azure/storage/common/storage-use-emulator) 用のストレージ エミュレーターが、運用環境のストレージを使用する前にその状態を確認するうえで役立つ場合があります。

## <a name="how-to-use-activity-handlers"></a>アクティビティ ハンドラーを使用する方法

アクティビティ ハンドラーにより、別の複雑さが生じることがあります。具体的には、各アクティビティが独立したスレッドで実行されます (お使いの言語によっては Web worker で実行されます)。 ご利用のハンドラーが行っている処理によっては、現在の状態が、期待どおりのものではないという問題が発生することがあります。

組み込みの状態はターンの最後に書き込まれますが、そのターンによって生成されたアクティビティはすべて、ターン パイプラインとは無関係に実行されています。 多くの場合、この影響を受けることはありませんが、アクティビティ ハンドラーによって状態が変更された場合は、書き込まれたその状態に、その変更を含める必要があります。 この場合、ターン パイプラインは、アクティビティ上で処理が完了するのを待つことができます。これにより、完了前に、そのターンについて適切な状態が確実に記録されます。

_send activity_ メソッドとそのハンドラーにより、独自の問題が発生します。 _on send activities_ ハンドラー内から _send activity_ を単に呼び出すと、スレッドの無限フォークが発生します。 この問題を回避する方法はいくつかあります。たとえば、メッセージを送信情報に追加します。また、コンソールやファイルなどの別の場所に書き出してボットのクラッシュを回避します。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [ボットの単体テストを実行する方法](unit-test-bots.md)

## <a name="additional-resources"></a>その他のリソース

* [Visual Studio でのデバッグ](https://docs.microsoft.com/visualstudio/debugger/index)
* Bot Framework のための[デバッグ、トレース、およびプロファイリング](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/)
* 運用環境のコードに含めたくないメソッドに [ConditionalAttribute](https://docs.microsoft.com/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) を使用する
* [Fiddler](https://www.telerik.com/fiddler) などのツールを使用してネットワーク トラフィックを確認する
* [一般的な問題のトラブルシューティング](../bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。
