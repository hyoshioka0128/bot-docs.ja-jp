---
title: ボットを Web サイトに埋め込む - Bot Service
description: Web サイトに埋め込まれるボットを設計する方法について説明します。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 200ad4fc9528a24390edff8ee54b26d344547e04
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80647486"
---
# <a name="embed-a-bot-in-a-website"></a>ボットを Web サイトに埋め込む

ボットは一般的には Web サイト外に存在しますが、Web サイト内に埋め込むことができます。 たとえば、複雑な Web サイト構造で検索が大変な情報をユーザーがすばやく見つけられるように、[ナレッジ ボット](~/bot-service-design-pattern-knowledge-base.md)を Web サイトに埋め込むことができます。 また、受信ユーザー要求に対する最初のレスポンダーとして機能するように、ヘルプ デスク Web サイトにボットを埋め込むこともあります。 シンプルな問題についてはボットが自力で解決し、複雑な問題は、人のエージェントに[ハンドオフ](~/bot-service-design-pattern-handoff-human.md)されます。 

この記事では、Web サイトへのボットの統合と、Web ページとボットの間のプライベート通信を容易にする "*バックチャネル*" メカニズムを使用するプロセスについて説明します。 

Microsoft では、Skype Web コントロールとオープン ソース Web コントロールの 2 つの異なる方法で、ボットを Web サイトに統合できます。

## <a name="skype-web-control"></a>Skype Web コントロール

>[!NOTE]
> 2019 年 10 月 31 日をもって、Skype チャネルは新しいボット公開要求を受け付けなくなりました。 つまり、Skype チャンネルを使ってボットの開発を続けることはできますが、ボットのユーザー数は 100 人に制限されます。 ボットをより多くのユーザーに公開することはできません。 現在の Skype ボットは引き続き中断なく実行されます。 [Skype で一部の機能が利用できなくなった理由](https://support.skype.com/faq/fa12091/why-are-some-features-not-available-in-skype-anymore)についてご確認ください。


[Skype Web コントロール](https://aka.ms/bot-skype-web-control)は、基本的には、Web 対応コントロールの Skype クライアントです。 組み込みの Skype 認証により、ボットでユーザーを認証および認識できます。開発者がカスタム コードを記述する必要はありません。 Skype では、Web クライアントで使用されている Microsoft アカウントが自動的に認識されます。 

Skype Web コントロールは、単に Skype のフロントエンドとして機能するので、ユーザーの Skype クライアントでは、Web コントロールで容易になるすべての会話の完全なコンテキストに、自動的にアクセスすることができます。 Web ブラウザーを閉じた後も、ユーザーは Skype クライアントを使用して引き続きボットを操作できます。 

## <a name="open-source-web-control"></a>オープン ソース Web コントロール

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">オープン ソース Web チャット コントロール</a>は ReactJS に基づいており、Bot Framework との通信には [Direct Line API][directLineAPI] が使用されています。 Web チャット コントロールには、Web チャットを実装するための空白のキャンバスが用意されており、そのチャットの動作と、チャットのユーザー エクスペリエンスをフル コントロールできます。 

"*バックチャネル*" メカニズムにより、コントロールをホストしている Web ページが、ユーザーにはまったく見えないようにボットと直接通信できます。 この機能により、複数の便利なシナリオに対応できます。 

- 関連データを Web ページからボットに送信できます (例: GPS の位置情報)。
- ユーザー アクションに関するアドバイスを Web ページからボットに提供できます (例: "ユーザーがドロップダウンからオプション A を選択した")。
- ログインしたユーザー用の認証トークンを Web ページからボットに送信できます。
- 関連データをボットから Web ページに送信できます (例: ユーザーのポートフォリオの現在の値)。
- "コマンド" をボットから Web ページに送信できます (例: 背景色の変更)。

## <a name="using-the-backchannel-mechanism"></a>バックチャネル メカニズムの使用

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>サンプル コード

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">オープン ソース Web チャット コントロール</a>は GitHub で入手できます。 オープン ソース Web チャット コントロールと Bot Framework SDK for Node.js を使用してバックチャネル メカニズムを実装する方法の詳細については、「[バックチャネル メカニズムの使用](~/nodejs/bot-builder-nodejs-backchannel.md)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [Direct Line API][directLineAPI]
- [オープン ソース Web チャット コントロール](https://github.com/Microsoft/BotFramework-WebChat)
- [バックチャネル メカニズムの使用](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle
