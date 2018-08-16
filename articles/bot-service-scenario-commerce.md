---
title: コマース ボットのシナリオ | Microsoft Docs
description: Bot Framework によるコマース ボットのシナリオについて説明します。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 15745bc25013df2fd18b0a2045ae2314d6c361e2
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574858"
---
# <a name="commerce-bot-scenario"></a>コマース ボットのシナリオ

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

[コマース ボット](bot-service-scenario-commerce.md)のシナリオでは、ホテルのコンシェルジュ サービスなど、従来の電子メールおよび電話によるインタラクションを置換するボットについて説明します。 ボットは、Cognitive Services を活用し、バックエンド サービスとの統合から収集したコンテキストを踏まえて、テキストや音声による顧客の要求に対する処理を改善します。

![Application Bot のダイアグラム](~/media/scenarios/bot-service-scenario-commerce-bot.png)

次に、ホテルのコンシェルジュ機能を持つコマース ボットのロジック フローを示します。

1. 顧客は、ホテルのモバイル アプリを使用します。
2. Azure AD B2C を使用してユーザー認証を行います。
3. カスタム Application Bot を使用して、ユーザーが情報を要求します。 
4. Cognitive Services が自然言語の要求を処理します。
5. 自然な会話を使用して質問を改善できるユーザーが応答をレビューします。
6. ユーザーが結果に満足すれば、Application Bot がユーザーの予約を更新します。
7. Application Insights が、ランタイム テレメトリを収集して、ボットのパフォーマンスと使用方法により開発をサポートします。

## <a name="sample-bot"></a>サンプル ボット
サンプルのコマース ボットは、架空のホテルのコンシェルジュ サービスに基づいて設計されています。 このボットは C# で記述されています。顧客は、チェーンの会員サービス モバイル アプリからホテルの Azure AD B2C を認証した後、ボットにアクセスします。 チェーンの予約情報は、SQL Database に格納されます。 顧客は、「プール付きの部屋に泊まるには、いくらかかりますか」というような自然な言葉で質問できます。 次にボットは、どのホテルか、ゲストの宿泊期間はどのくらいかといったコンテキストを尋ねます。 さらに、Language Understanding (LUIS) サービスにより、「プール付きの部屋」のような単純な語句からも簡単にコンテキストを取得できます。 ボットは、ゲストに回答を返すとともに、日数や部屋の種類を提示して、プール付きの部屋を予約することを提案します。 必要なデータがすべてそろうと、ボットは予約を行います。 ゲストは、音声で同じ要求を行うこともできます。

このサンプル ボットは、[Bot Framework の一般的なシナリオのサンプル](https://aka.ms/bot/scenarios)からダウンロードするか、ソース コードを複製できます。

## <a name="components-youll-use"></a>使用するコンポーネント
コマース ボットでは、次のコンポーネントを使用します。
-   認証用の Azure AD
-   Cognitive Services: LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) は、マイクロソフトが提供する、マルチテナントに対応したクラウド ベースのディレクトリと ID の管理サービスです。 また Azure AD は、世界中の数多くの組織が使用する卓越した ID 管理ソリューションとすばやく簡単に統合できるため、ボット開発者がボットの構築に集中することを可能にします。 Azure AD では、Google、Facebook、Microsoft アカウントなどの外部 ID を使用して個人を識別できる B2C コネクタがサポートされています。 Azure AD を使うと、ユーザーの資格情報管理が不要になり、ボットのソリューションに集中できるようになります。ボットのユーザーとアプリケーションによって公開されている正しいデータを関連付けることができるのは皆さんであることを把握しているからです。

### <a name="cognitive-services-luis"></a>Cognitive Services: LUIS
Cognitive Services ファミリ テクノロジのメンバーである Language Understanding (LUIS) を使うと、アプリに機械学習の力を追加することができます。 現時点で、LUIS はいくつかの言語をサポートしており、それを使って人がやりたいことをボットに解釈させることができます。 LUIS を統合する際は、意図を明示し、ボットが解釈するエンティティを定義します。 続いて、発話の例を使ってトレーニングを行い、ボットがこれらの意図やエンティティを解釈できるようにします。 この統合は、ボットができるだけ柔軟に特定の会話ニーズに対応できるように、フレーズ リストや正規表現機能を使用して調整することができます。

### <a name="application-insights"></a>Application Insights
Application Insights では、アプリケーション パフォーマンス管理 (APM) と瞬時の分析によって、行動につながる知見を得ることができます。 事前の設定なしで、さまざまな機能を持つパフォーマンス監視、強力なアラート機能、使いやすいダッシュボードによって、ボットの可用性が保たれ、期待通りに動作していることを確認できます。 問題が発生していることをすばやく確認し、根本原因を分析して、問題を検出し、修正できます。

## <a name="next-steps"></a>次の手順
次に、Cortana スキル ボットのシナリオについて説明します。

> [!div class="nextstepaction"]
> [Cortana スキル ボットのシナリオ](bot-service-scenario-cortana-skill.md)
