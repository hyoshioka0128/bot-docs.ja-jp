---
title: Bot Service についてよく寄せられる質問 | Microsoft Docs
description: Bot Framework の要素と、新しい機能が利用可能になる時期についてよく寄せられる質問を示します。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: e59f9b10686b10ae821b8c4bf259a1fc301ac702
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301226"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework についてよく寄せられる質問

この記事では、Bot Framework についてよく寄せられる質問とその回答を示します。

## <a name="background-and-availability"></a>背景と提供状況
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft が Bot Framework を開発したのはなぜですか?

会話ユーザー インターフェイス (CUI) が登場しましたが、現時点では、新しい会話型エクスペリエンスを構築したり、ユーザーが楽しめる会話型インターフェイスで既存のアプリケーションやサービスを利用できるようにするために必要な専門知識やツールを持っている開発者はほとんどいません。 Microsoft は、開発者が優れたボットを作成し、Microsoft のプレミア チャンネル上など、ユーザーが会話する場所にボットを接続することを容易にするために、Bot Framework を開発しました。

### <a name="what-is-the-v4-sdk"></a>v4 SDK とは何ですか?
Bot Builder v4 SDK は、以前の Bot Builder SDK から得たフィードバックと知識を基に構築されています。 ボットの構成要素の豊富なコンポーネント化を可能にしながら、適切なレベルの抽象化が導入されます。 簡単なボットから始め、モジュール式の拡張可能なフレームワークを使用してボットを高度化していくことができます。 GitHub で SDK の [FAQ](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ) をご覧ください。


## <a name="channels"></a>チャンネル
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>Bot Framework にさらに多くの会話エクスペリエンスが追加されるのはいつですか?

Microsoft では、チャンネルの追加など、Bot Framework を継続的に改善していく予定ですが、現時点でスケジュールを提供することはできません。  
フレームワークに特定のチャンネルを追加することをご希望の場合は[お知らせください][Support]。

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>Bot Framework で構成可能にしたいコミュニケーション チャンネルがあります。 Microsoft と協力してこれを行うことはできますか?

開発者が Bot Framework に新しいチャンネルを追加する一般的なメカニズムは提供していませんが、[Direct Line API][DirectLineAPI] を使用してボットをアプリに接続できます。 お客様がコミュニケーション チャンネルの開発者であり、Microsoft と協力して Bot Framework でチャンネルを 有効にすることをご希望であれば、[ぜひお知らせください][Support]。

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>Skype 用のボットを作成する場合、どのようなツールやサービスを使用すればよいですか?

Bot Framework は、Skype や他の多くのチャンネル向けに、高品質でパフォーマンスと応答性に優れたスケーラブルなボットを作成、接続、展開できるように設計されています。 SDK を使用して、(会話エクスペリエンスで今日のボットの対話の大半を占める) テキスト/SMS、画像、ボタン、およびカード対応のボットや、豊富なオーディオ/動画エクスペリエンスなどの Skype 固有のボットの対話を作成できます。

優れたボットが既にあり、Skype の対象ユーザーを獲得したい場合は、Bot Builder for REST API を使用して Skype (またはサポートされている任意のチャンネル) に簡単に接続できます (インターネットにアクセス可能な REST エンドポイントがある場合)。

## <a name="security-and-privacy"></a>セキュリティとプライバシー
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>Bot Framework に登録されているボットは個人情報を収集するのでしょうか。 収集する場合、データの安全とセキュリティを確保するにはどうすればよいですか?  プライバシーはどうですか?

各ボットは独自のサービスであり、これらのサービスの開発者には、開発者倫理規定に従ってサービス利用規約とプライバシーに関する声明を提供することが義務付けられています。  この情報には、ボット ディレクトリのボットのカードからアクセスできます。

I/O サービスを提供するために、Bot Framework では、お客様が使用したチャット サービスからボットに、メッセージとメッセージのコンテンツ (お客様の ID を含む) を送信します。

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>Microsoft はどのようにしてボットを禁止したり、サービスからボットを削除したりするのですか?

ユーザーは、ディレクトリ内のボットの連絡先カードを使用して、不正な動作をするボットを報告できます。 開発者がサービスに参加するには、Microsoft サービス利用規約に従う必要があります。

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-services"></a>ボット サービスにアクセスするために、会社のファイアウォールでホワイトリストに登録する必要がある URL を教えてください。

会社のファイアウォールでは、次の URL をホワイトリストに登録する必要があります。
- login.botframework.com (Bot 認証)
- login.microsoftonline.com (Bot 認証)
- westus.api.cognitive.microsoft.com (Luis.ai の NLP 統合用)
- state.botframework.com (プロトタイプ作成用のボット状態ストレージ)
- cortanabfchanneleastus.azurewebsites.net (Cortana チャンネル)
- cortanabfchannelwestus.azurewebsites.net (Cortana チャンネル)
- *.botFramework.com (チャンネル)

## <a name="rate-limiting"></a>レート制限
### <a name="what-is-rate-limiting"></a>レート制限とは何ですか?
Bot Framework サービスは、どのボットも他のボットのパフォーマンスに悪影響を及ぼすことがないように、不正な呼び出しパターン (サービス拒否攻撃など) からサービス自体と顧客を保護する必要があります。 このような保護を実現するために、すべてのエンドポイントにレート制限 ("調整" とも呼ばれます) が追加されました。 レート制限を適用することで、ボットが特定の呼び出しを行うことができる頻度を制限できます。 たとえば、レート制限を有効にすると、ボットは多数のアクティビティを投稿する場合に、それらを一定期間にわたって分割して投稿する必要があります。 レート制限の目的は、ボットの総容量の上限を設けることではないことに注意してください。 人間の会話パターンに従っていない会話インフラストラクチャの不正使用を防ぐことを目的としています。

### <a name="how-will-i-know-if-im-impacted"></a>影響を受けているかどうかはどうすればわかりますか?
要求が大量であっても、お客様がレート制限を受けることはまずありません。 ほとんどのレート制限は、(ボットまたはクライアントからの) アクティビティの一括送信、極端なロード テスト、またはバグによってのみ発生します。 要求が調整されると、要求の再試行が成功するまでの待機時間 (秒単位) を示す Retry-After ヘッダーと共に、HTTP 429 (Too Many Requests) 応答が返されます。 この情報を収集するには、Azure Application Insights を使用してボットの分析を有効にします。 また、メッセージをログに記録するコードをボットに追加することもできます。 

### <a name="how-does-rate-limiting-occur"></a>レート制限はどのように発生しますか?
次の場合に発生する可能性があります。
-   ボットがメッセージを送信する頻度が高すぎる場合
-   ボットのクライアントがメッセージを送信する頻度が高すぎる場合
-   Direct Line クライアントが新しい Web ソケットを要求する頻度が高すぎる場合

### <a name="what-are-the-rate-limits"></a>レート制限値を教えてください。
サービスとユーザーを保護しながら、できるだけ緩やかなものにするために、レート制限値は継続的に調整されています。 しきい値が変更されることがあるため、現時点では数値を公開していません。 レート制限の影響を受けている場合は、[bf-reports@microsoft.com](mailto://bf-reports@microsoft.com) までお気軽にご連絡ください。

## <a name="related-services"></a>関連サービス
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Bot Framework は Cognitive Services にどのように関連していますか?

Bot Framework と [Cognitive Services](http://www.microsoft.com/cognitive) は、[Microsoft Build 2016](http://build.microsoft.com) で導入された新しい機能であり、一般提供時に Cortana Intelligence Suite にも統合されます。 これらのサービスはいずれも、長年にわたる研究と一般的な Microsoft 製品での使用に基づいて構築されています。 これらの機能を "Cortana Intelligence" と組み合わせることで、あらゆる組織が、データの力、クラウド、インテリジェンスを活用して、新たなチャンスを切り開き、ビジネスを加速させ、顧客にサービスを提供する業界をリードする独自のインテリジェント システムを構築できます。

### <a name="what-is-cortana-intelligence"></a>Cortana Intelligence とは何ですか?

Cortana Intelligence は、データをインテリジェントなアクションに変える、フル マネージドのビッグ データ、高度な分析、およびインテリジェンスのスイートです。  
これは、Microsoft 全体にわたる長年の研究と技術革新に基づいたテクノロジー (高度な分析、機械学習、ビッグ データ ストレージ、クラウドでの処理など) をまとめた包括的なスイートです。

* すべてのデータを収集、管理、保存することができます。データは、スケーラブルかつ安全な方法で、長期的にシームレスにコスト効率よく増加させることができます。
* クラウドによって強化された、アクションにつながる分析を簡単に行うことができます。これらの分析により、最も厳しい問題に対処するための予測、指示、意思決定の自動化が可能になります。
* Cognitive Services とエージェントを使用して、状況に応じた自然な方法で周囲の世界を見る、聴く、解釈する、理解することを可能にするインテリジェント システムを実現します。

Microsoft は、Cortana Intelligence によって、企業のお客様が新たなチャンスを切り開き、ビジネスを加速させ、業界のリーダーになれるよう支援したいと考えています。

### <a name="what-is-the-direct-line-channel"></a>Direct Line チャンネルとは何ですか?

Direct Line は、サービス、モバイル アプリ、または Web ページにボットを追加できる REST API です。

Direct Line API のクライアントは、任意の言語で記述できます。 [Direct Line プロトコル][DirectLineAPI]にコード化し、Direct Line 構成ページでシークレットを生成して、コードが存在する場所からボットに話しかけるだけです。

Direct Line は以下に適しています。

* iOS、Android、Windows Phone などの OS 上のモバイル アプリ
* Windows、OSX などの OS 上のデスクトップ アプリケーション
* [埋め込み可能な Web チャット チャンネル][WebChat]が提供するものよりも多くのカスタマイズが必要な Web ページ
* サービス間アプリケーション

[DirectLineAPI]: http://docs.botframework.com/en-us/restapi/directline/
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

