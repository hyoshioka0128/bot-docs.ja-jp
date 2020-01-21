---
title: Bot Service シナリオの概要 - Bot Service
description: Bot Service で構築した強力なボットをうまく活用する主なシナリオについて説明します。
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6786a25d56bbf37262760ca5285e32aa860c5959
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75793449"
---
# <a name="bot-scenarios"></a>ボット シナリオ

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

このトピックでは、Bot Service で構築した強力なボットをうまく活用する主なシナリオについて説明します。

すべてのシナリオのボット サンプルのソース コードは、[Bot Framework の一般的なシナリオのサンプル](https://aka.ms/abs-scenarios)からダウンロードまたは複製できます。

## <a name="commerce-bot-scenario"></a>コマース ボットのシナリオ
[コマース ボット](bot-service-scenario-commerce.md)のシナリオでは、ホテルのコンシェルジュ サービスなど、従来の電子メールおよび電話によるインタラクションを置換するボットについて説明します。 ボットは、Cognitive Services を活用し、バックエンド サービスとの統合から収集したコンテキストを踏まえて、テキストや音声による顧客の要求に対する処理を改善します。

コマース ボットのシナリオでは、顧客がホテルのコンシェルジュ サービスに対して要求を行うことができます。 顧客は、Azure Active Directory v2 認証エンドポイントを使用して認証されます。 ボットは、顧客の予約を検索でき、さまざまなサービスのオプションを提供することができます。 たとえば、プール付きの部屋を予約することができます。 ボットは、Language Understanding Intelligent Services (LUIS) を使用して要求を解析し、既存の予約にプール付きの部屋を追加するプロセスを進めます。

## <a name="cortana-skill-bot-scenario"></a>Cortana スキル ボットのシナリオ
[Cortana スキル ボット](bot-service-scenario-cortana-skill.md)のシナリオは、Cortana を利用します。 ユーザーの声とカスタム Cortana スキル ボットの自然なインターフェイスを使用して、現在の場所と時間に基づいて移動自動車整備店などの組織に連絡して予約を入れるように Cortana に頼むことができます。 このボットでは、サービス、利用可能な時間帯、および時間の長さの一覧を指定できます。

## <a name="enterprise-productivity-bot-scenario"></a>Enterprise Productivity ボットのシナリオ
[Enterprise Productivity ボット](bot-service-scenario-enterprise-productivity.md)のシナリオは、ボットと Office 365 カレンダーなどのサービスを統合して生産性を向上させる方法を示します。

ボットは Office 365 と統合されているので、迅速かつ簡単に他のユーザーに会議への出席を依頼することができます。 その課程で、Dynamics CRM などの他のサービスにアクセスすることもできます。 このサンプルでは、Azure Active Directory による認証を使用して Office 365 と統合するために必要なコードを提供します。 また、読者向けの演習として、外部サービス用のモック エントリ ポイントを提供しています。

## <a name="information-bot-scenario"></a>Information ボットのシナリオ
この [Information ボット](bot-service-scenario-informational.md) は、Cognitive Services QnA Maker を使用してナレッジ セットや FAQ で定義された質問に回答したり、Azure Search を使用してより結論が出にくい質問に回答したりできます。

情報は多くの場合、検索によって容易に取り出せる SQL Server のような構造化データ ストアに格納されています。 単純な会話コマンドによって顧客の注文状況を調べることを考えます。 Cognitive Services QnA Maker を使用して、顧客の検索、顧客の最新注文の確認といった、一連の有効な検索オプションがユーザーに提示されます。QnA 形式を定義することで、Azure Search によって強化され、SQL Database に格納されたデータを参照できる質問をユーザーが容易に作成できます。

## <a name="iot-bot-scenario"></a>IoT ボットのシナリオ
この[モノのインターネット (IoT) ボット](bot-service-scenario-internet-things.md)によって、対話型チャット コマンドを使用して、Philips Hue の照明などの自宅周辺のデバイスを簡単に制御できます。

この単純なボットと無料の If This Then That (IFTTT) サービスを組み合わせると、Philips Hue の照明を制御することができます。 IoT デバイスである Philips Hue は、公開されている API を使ってローカルで制御できます。 ただし、この API は広くアクセスできるようにローカル ネットワークの外部に公開されてはいません。 しかし、IFTTT は "[Friend of Hue](http://www2.meethue.com/friends-of-hue/ifttt/)" であり、照明のオンやオフ、照明の色や光の強度を変更など、たくさんの制御コマンドが公開されています。

## <a name="next-steps"></a>次のステップ
これでシナリオの概要が理解できたので、それぞれのシナリオを詳しく確認してください。

> [!div class="nextstepaction"]
> [コマース ボット](bot-service-scenario-commerce.md)
