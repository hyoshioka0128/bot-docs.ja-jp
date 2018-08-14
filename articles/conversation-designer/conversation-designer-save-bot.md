---
title: Conversation Designer ボットの保存 | Microsoft Docs
description: Conversation Designer ボットで、言語解釈モデルを保存してトレーニングし、音声認識に学習させる方法を説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304121"
---
# <a name="saving-your-conversation-designer-bot"></a>Conversation Designer ボットの保存
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の使用可能性については、今年中に詳細をお知らせする予定です。

Conversation Designer で作業中は、すべての作業内容はブラウザーのメモリにキャッシュされます。 変更をコミットするには、左側のナビゲーション メニューの左上隅にある **[保存]** ボタンをクリックします。 作業内容が失われないようにするには、作業内容を頻繁に保存することをお勧めします。 **[保存]** ボタンをクリックする以外に、**Ctrl + S** のキーボード ショートカットを使用して作業内容を保存することもできます。

## <a name="trains-luis-and-primes-speech-recognition"></a>LUIS のトレーニングと音声認識の学習

**[保存]** ボタンをクリックすると、ボットの変更が保存され、いくつかの追加タスクが実行されます。 キーボード ショートカットとは異なり、**[保存]** ボタンをクリックすると、Conversation Designer に次のタスクも実行するように指示されます。

- ボットで新しい LUIS の意図とエンティティをトレーニングし、Bot Service の LUIS モデルをローカルに公開します (必要な場合)。 これらの意図は、Conversation Designer またはボットの対応する [LUIS](https://luis.ai) アプリに追加できます。
- 会話ランタイムを更新して、新しい LUIS モデルを使用します。
- サンプルの発話を準備して Microsoft Cognitive Services に送信することにより、音声認識に学習させると、このボットの音声認識精度が大幅に向上します。

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [ボットのテスト](conversation-designer-debug-bot.md)
