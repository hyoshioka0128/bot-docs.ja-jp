---
title: ボットを Web サイトに埋め込む | Microsoft Docs
description: Web サイトに埋め込まれるボットを設計する方法について説明します。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 6aae4e9f6cb6a4e892b8036eafb9489dfaedbb36
ms.sourcegitcommit: ee63d9dc1944a6843368bdabf5878950229f61d0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/23/2018
ms.locfileid: "42795098"
---
# <a name="embed-a-bot-in-a-website"></a>ボットを Web サイトに埋め込む

ボットは一般的には Web サイト外に存在しますが、Web サイト内に埋め込むことができます。 たとえば、複雑な Web サイト構造で検索が大変な情報をユーザーがすばやく見つけられるように、[ナレッジ ボット](~/bot-service-design-pattern-knowledge-base.md)を Web サイトに埋め込むことができます。 また、受信ユーザー要求に対する最初のレスポンダーとして機能するように、ヘルプ デスク Web サイトにボットを埋め込むこともあります。 シンプルな問題についてはボットが自力で解決し、複雑な問題は、人のエージェントに[ハンドオフ](~/bot-service-design-pattern-handoff-human.md)されます。 

この記事では、Web サイトへのボットの統合と、Web ページとボットの間のプライベート通信を容易にする "*バックチャネル*" メカニズムを使用するプロセスについて説明します。 

Microsoft では、Skype Web コントロールとオープン ソース Web コントロールの 2 つの異なる方法で、ボットを Web サイトに統合できます。

## <a name="skype-web-control"></a>Skype Web コントロール

Skype Web コントロールは、基本的には、Web 対応コントロールの Skype クライアントです。 組み込みの Skype 認証により、ボットでユーザーを認証および認識できます。開発者がカスタム コードを記述する必要はありません。 Skype では、Web クライアントで使用されている Microsoft アカウントが自動的に認識されます。 

Skype Web コントロールは、単に Skype のフロントエンドとして機能するので、ユーザーの Skype クライアントでは、Web コントロールで容易になるすべての会話の完全なコンテキストに、自動的にアクセスすることができます。 Web ブラウザーを閉じた後も、ユーザーは Skype クライアントを使用して引き続きボットを操作できます。 

## <a name="open-source-web-control"></a>オープン ソース Web コントロール

<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">オープン ソース Web チャット コントロール</a>は ReactJS に基づいており、Bot Framework との通信には [Direct Line API][directLineAPI] が使用されています。 Web チャット コントロールには、Web チャットを実装するための空白のキャンバスが用意されており、そのチャットの動作と、チャットのユーザー エクスペリエンスをフル コントロールできます。 

"*バックチャネル*" メカニズムにより、コントロールをホストしている Web ページが、ユーザーにはまったく見えないようにボットと直接通信できます。 この機能により、複数の便利なシナリオに対応できます。 

- 関連データを Web ページからボットに送信できます (例: GPS の位置情報)。
- ユーザー アクションに関するアドバイスを Web ページからボットに提供できます (例: "ユーザーがドロップダウンからオプション A を選択した")。
- ログインしたユーザー用の認証トークンを Web ページからボットに送信できます。
- 関連データをボットから Web ページに送信できます (例: ユーザーのポートフォリオの現在の値)。
- "コマンド" をボットから Web ページに送信できます (例: 背景色の変更)。

## <a name="using-the-backchannel-mechanism"></a>バックチャネル メカニズムの使用

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>サンプル コード

<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">オープン ソース Web チャット コントロール</a>は GitHub で入手できます。 オープン ソース Web チャット コントロールと Bot Builder SDK for Node.js を使用してバックチャネル メカニズムを実装する方法の詳細については、「[Use the backchannel mechanism (バックチャネル メカニズムの使用)](~/nodejs/bot-builder-nodejs-backchannel.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [Direct Line API][directLineAPI]
- [オープン ソース Web チャット コントロール](https://github.com/Microsoft/BotFramework-WebChat)
- [バックチャネル メカニズムの使用](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
