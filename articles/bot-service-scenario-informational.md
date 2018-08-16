---
title: Information ボットのシナリオ | Microsoft Docs
description: Bot Framework による Information ボットのシナリオについて説明します。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43eb8e25e2a17e1d6b1d30e767dd15569fcad78b
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574868"
---
# <a name="information-bot-scenario"></a>Information ボットのシナリオ

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

この Information ボットは、Cognitive Services QnA Maker を使用してナレッジ セットや FAQ で定義された質問に回答したり、Azure Search を使用してより結論が出にくい質問に回答したりできます。

情報は多くの場合、検索によって容易に取り出せる SQL Server のような構造化データ ストアに格納されています。 単純な会話コマンドによって顧客の注文状況を調べることを考えます。 Cognitive Services QnA Maker を使用して、顧客の検索、顧客の最新注文の確認といった、一連の有効な検索オプションがユーザーに提示されます。QnA 形式を定義することで、Azure Search によって強化され、SQL Database に格納されたデータを参照できる質問をユーザーが容易に作成できます。

![Information ボットの図](~/media/scenarios/bot-service-scenario-informational-bot.png)

次に、Information ボットのロジック フローを示します。

1. 従業員が Information ボットを開始します。
2. Azure Active Directory が従業員の ID を確認します。
3. 従業員はサポートされているクエリの種類をボットに質問できます。
4. Cognitive Services が、QnA Maker で構築された FAQ ボットを返します。
5. 従業員が有効なクエリを定義します。
6. ボットが、アプリケーション データに関する情報を返す Azure Search にクエリを送信します。
7. Application Insights が、ランタイム テレメトリを収集して、ボットのパフォーマンスと使用方法により開発をサポートします。

## <a name="sample-bot"></a>サンプル ボット
C# で記述されたサンプル ボットは Microsoft Azure で動作し、Azure Search によってインデックス付けされた SQL Database インスタンス内のデータを操作します。 ボットは、尋ねることができる質問のリストを、Cognitive Services: QnA Maker を使用して質問 (回答) をフレーズ化する方法に関する情報と共に公開します。 ボットのユーザーはその後、インデックス付けされているデータベースの広い領域または特定の領域で Azure Search によってデータを参照するクエリを入力できます。 このサンプルは、顧客と注文情報の単純なデータベースを提供します。 Application Insights はボットの使用状況を追跡し、ボットの例外の監視に役立ちます。 誰が情報にアクセスできるかを制限できるよう、このボットは Azure AD アプリとして発行されます。

このサンプル ボットは、[Bot Framework の一般的なシナリオのサンプル](https://aka.ms/bot/scenarios)からダウンロードするか、ソース コードを複製できます。

## <a name="components-youll-use"></a>使用するコンポーネント
Information ボットでは、次のコンポーネントを使用します。
-   認証用の Azure AD
-   Cognitive Services: QnA Maker
-   Azure Search
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) は、マイクロソフトが提供する、マルチテナントに対応したクラウド ベースのディレクトリと ID の管理サービスです。 また Azure AD は、世界中の数多くの組織が使用する卓越した ID 管理ソリューションとすばやく簡単に統合できるため、ボット開発者がボットの構築に集中することを可能にします。 独自の複雑な認証および認可システムを実装しなくても、Azure AD アプリを定義することによって、ボットおよびボットが公開するデータに誰がアクセスできるかを制御できます。

### <a name="cognitive-services-qna-maker"></a>Cognitive Services: QnA Maker
Cognitive Services QnA Maker は、ユーザーがボットからクエリできる FAQ データ ソースを提供するために役立ちます。 さまざまなシステムに格納された膨大な量の情報にアプローチする際、情報のソースとセットをユーザーが絞り込めるようにすることは有用です。 単一の SQL データベースに膨大な量の情報が含まれていて、自由形式の検索を適用するとあまりにも多くの情報が返される可能性があります。 最初に QnA Maker を使用してボット ユーザーのためのロードマップを定義し、Azure Search によって取得できるインテリジェントな質問の作成方法を習得させることができます。

### <a name="azure-search"></a>Azure Search
Azure Search はアプリ向けのクラウド検索サービスであり、迅速に検索インデックスを構築して稼働させることができます。 Microsoft Azure で稼働するため、使用量の需要に応じてスケールアップとスケールダウンを簡単に行えます。 検索ランキングの高度な制御によって検索結果をビジネス目標に結び付け、データベースに隠れたデータを表出させることができます。

### <a name="application-insights"></a>Application Insights
Application Insights では、アプリケーション パフォーマンス管理 (APM) と瞬時の分析によって、行動につながる知見を得ることができます。 事前の設定なしで、さまざまな機能を持つパフォーマンス監視、強力なアラート機能、使いやすいダッシュボードによって、ボットの可用性が保たれ、期待通りに動作していることを確認できます。 問題が発生していることをすばやく確認し、根本原因を分析して、問題を検出し、修正できます。

## <a name="next-steps"></a>次の手順
次に、モノのインターネット (IoT) ボットのシナリオを理解します。

> [!div class="nextstepaction"]
> [モノのインターネット (IoT) ボットのシナリオ](bot-service-scenario-internet-things.md)
