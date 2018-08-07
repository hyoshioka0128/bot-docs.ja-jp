---
title: Conversation Designer | Microsoft Docs
description: Conversation Designer について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 42e9e8137619fff2ca86f82fea1fe81aa348449b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39111559"
---
# <a name="conversation-designer"></a>Conversation Designer
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

Conversation Designer は、ビジュアル ボット ビルダー、ボットを定義するための宣言型モデル、そしてボットの作成を簡単にするランタイムを提供する強力なフレームワークです。

便利な会話型ボットをビルドするには、会話型コントロール、言語の理解、ダイアログ、言語の生成の統合についてだけでなく、同時にユーザー エクスペリエンスと、既存のビジネス ロジックとの統合へのフォーカスの維持についての統合されたアプローチも必要です。 Conversation Designer を使用すると、オーサリング、デバッグ、LUIS、分析、音声認識、および既存の Bot Framework SDK ボットとの互換性にまたがる統合されたパスを使用してボットをビルドできます。

Conversation Designer では、利用可能な入力/出力モダリティ簡単に適応するボットをビルドできます。活用できる機能は、次のとおりです。 

- 状態管理のオーバーヘッドを最小限に抑える強力なランタイム
- 会話の簡単なビジュアル モデリングを可能にする組み込みの会話の状態
- コードとポット UI の分離
- AI フレームワーク (<a href="https://luis.ai" target="_blank">LUIS</a>、<a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">音声認識</a>など) の組み込みサポート
- シンプルな条件付きの応答のテンプレートを使用した言語生成のための組み込みのサポート
- [アダプティブ カード](conversation-designer-adaptive-cards.md)のサポート

## <a name="prerequisites"></a>前提条件

- Conversation Designer には、Azure サブスクリプションが必要です。 <a href="https://azure.microsoft.com/en-us/" target="_blank">ここ</a>から開始できます。
- まだ実行していない場合は、[LUIS ポータル](https://luis.ai)にアカウントを作成した後に少なくとも 1 回はサインインしていることを確認してください。
- JavaScript のプログラミングに関する知識。 カスタム スクリプト関数は JavaScript で記述します。
- Microsoft Edge または Google Chrome

## <a name="next-steps"></a>次の手順
> [!div class="nextstepaction"]
> [ボットの作成](conversation-designer-create-bot.md)

## <a name="additional-resources"></a>その他のリソース
Conversation Designer を使用したボットのビルドについて説明します。
- [言語の理解](conversation-designer-luis.md): ボットの言語の理解の設定
- [ダイアログ](conversation-designer-dialogues.md): ボットの Conversation ダイアログの設定
- [応答のテンプレート](conversation-designer-response-templates.md): ボットの応答テンプレートの設定に関するクイックヒント
- [アダプティブ カード](conversation-designer-adaptive-cards.md): アダプティブ カードを使用した、ボットの豊富な視覚的相互作用を設定
