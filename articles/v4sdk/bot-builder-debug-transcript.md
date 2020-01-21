---
title: トランスクリプト ファイルを使用してお使いのボットをデバッグする - Bot Service
description: トランスクリプト ファイルを使用して、ボットのデバッグに役立てる方法について説明します。
keywords: デバッグ, FAQ, トランスクリプト ファイル, エミュレーター
author: DanDev33
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6d1f3aafb805eba6ebf09fc7b15f29df4437d11c
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798602"
---
# <a name="debug-your-bot-using-transcript-files"></a>トランスクリプト ファイルを使用してボットをデバッグする

[!INCLUDE[applies-to](../includes/applies-to.md)]

ボットのテストとデバッグを成功させるうえで重要なポイントの 1 つが、ご自身のボットを実行するときに発生する一連の状況を記録し、調査できることです。 この記事では、ボット トランスクリプト ファイルを作成して使用し、テストおよびデバッグのために、一連の詳細なユーザー操作とボット応答を提供する方法について説明します。

## <a name="the-bot-transcript-file"></a>ボット トランスクリプト ファイル
ボット トランスクリプト ファイルは、ユーザーとご自身のボットの間のやり取りが保持される特殊な JSON ファイルです。 トランスクリプト ファイルには、メッセージの内容だけでなく、ユーザー ID、チャネル ID、チャネルの種類、チャネルの機能、やり取りが行われた時間など、やり取りの詳細が保持されます。この情報はすべて、後でご自身のボットをテストまたはデバッグするときに使用して、問題の特定および解決に役立てることができます。 

## <a name="creatingstoring-a-bot-transcript-file"></a>ボット トランスクリプト ファイルを作成/格納する
この記事では、Microsoft の [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) を使用して、ボット トランスクリプト ファイルを作成する方法について説明します。 トランスクリプト ファイルはプログラムで作成することもできます。その方法の詳細については、[こちら](./bot-builder-howto-v4-storage.md#blob-transcript-storage)をお読みください。 この記事では、ユーザーの移動手段、名前、年齢を要求する[マルチ ターン プロンプト ボット](https://aka.ms/cs-multi-prompts-sample)用の Bot Framework のサンプル コードを使用しますが、Microsoft の Bot Framework Emulator を使用してアクセスできるすべてのコードを、トランスクリプト ファイルの作成に使用することができます。

このプロセスを開始するには、テストするボット コードがご自身の開発環境内で実行されていることを確認します。 Bot Framework Emulator を起動し、 _[Open Bot]\(ボットを開く\)_ を選択して、次の画像に示すように、ブラウザーに表示される _localhost:port_ のアドレスを入力し、その後ろに "/api/messages" を付けます。 ここで、 _[接続]_ をクリックして、エミュレーターをお使いのボットに接続します。

![エミュレーターをコードに接続する](./media/emulator_open_bot_configuration.png)

エミュレーターをご自身の実行中のコードに接続した後、シミュレートされたユーザー操作をボットに送信してコードをテストします。 この例では、ユーザーの移動手段、名前、および年齢が渡されました。 保持したいすべてのユーザー操作を入力し、Bot Framework Emulator を使って、この会話が含まれるトランスクリプト ファイルを作成し、保存します。 

_[ライブ チャット]_ タブ (以下を参照) で、 _[Save transcript]\(トランスクリプトの保存\)_ を選択します。 

![[save transcripot]\(トランスクリプトの保存\) を選択する](./media/emulator_transcript_save.png)

トランスクリプト ファイルの場所と名前を選択し、保存ボタンを選択します。

![トランスクリプトを ursula として保存する](./media/emulator_transcript_saveas_ursula.png)

エミュレーターでご自身のコードをテストするために入力した、すべてのユーザー操作とボット応答がトランスクリプト ファイルに保存されました。これは後で再度読み込むことができるため、該当するユーザーとご自身のボットの間のやり取りをデバッグするときに役立ちます。

## <a name="retrieving-a-bot-transcript-file"></a>ボット トランスクリプト ファイルを取得する
Bot Framework Emulator を使用してボット トランスクリプト ファイルを取得するには、以下に示すように、エミュレーターの左上隅にある _[ファイル]_ 、 _[Open Transcript...]\(トランスクリプトを開く\)_ を順に選択します。 次に、取得するトラン スクリプト ファイルを選択します。 (トランスクリプトには、エミュレーターの _[リソース]_ セクションの _[トランスクリプト]_ リスト コントロール内からアクセスできます) 

この例では、"ursula_user.transcript" という名前のトランスクリプト ファイルを取得します。 トランスクリプト ファイルを選択すると、保持されているやり取り全体が、"_トランスクリプト_" という名前の新しいタブに自動的に読み込まれます。

![保存済みトランスクリプトを取得する](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>トランスクリプト ファイルを使用してデバッグする
ご自身のトランスクリプト ファイルが読み込まれると、取り込んだ、ユーザーとご自身のボットの間のやり取りをデバッグする準備が整います。 これを行うには、エミュレーターの右下の領域に表示される _[ログ]_ セクションで、記録された任意のイベントまたはアクティビティをクリックするだけです。 次の例では、ユーザーが "Hello" というメッセージを送信したときの最初のやり取りが選択されています。 これを行うと、トランスクリプト ファイル内のこの特定のやり取りに関するすべての情報が、エミュレーターの _[インスペクター]_ ウィンドウに JSON 形式で表示されます。 これらの値を下から順番にいくつか見ていくと、次の情報を確認できます。
* やり取りの種類が "_メッセージ_" であったこと。
* メッセージが送信された時間。
* 送信されたプレーンテキストには "Yes" が含まれていたこと。
* メッセージがボットに送信されたこと。
* ユーザー ID と情報。
* チャネルの ID、機能、および情報。

![トランスクリプトを使用してデバッグする](./media/emulator_transcript_debug.png)

このレベルの詳細情報により、ユーザー入力とボットの応答の間のやり取りを段階的に把握できます。これは、意図したとおりにボットが応答しなかったり、ユーザーにまったく応答しなかったりした状況をデバッグする際に役立ちます。 これらの値と、やり取りが失敗するまでの過程の記録の両方によって、ご自身のコードをステップ実行し、ボットが意図したとおりに応答していない場所を特定して、これらの問題を解決することができます。

ご自身のボットのコードとユーザー操作は、さまざまなツールを使ってテストおよびデバッグすることができます。トランスクリプト ファイルと Bot Framework Emulator の併用はその 1 つに過ぎません。 お使いのボットのテストおよびデバッグに使用できる他の方法については、以下に示す他のリソースを参照してください。

## <a name="additional-information"></a>関連情報

その他のテストおよびデバッグ情報については、以下を参照してください。

* [ボットのテストとデバッグのガイドライン](./bot-builder-testing-debugging.md)
* [Bot Framework Emulator でデバッグする](../bot-service-debug-emulator.md)
* [一般的な問題のトラブルシューティング](../bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。
* [Visual Studio でのデバッグ](https://docs.microsoft.com/visualstudio/debugger/index)
