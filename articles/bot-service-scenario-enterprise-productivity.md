---
title: Enterprise Productivity ボットのシナリオ | Microsoft Docs
description: Bot Framework による Enterprise Productivity ボットのシナリオについて説明します。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4e0bde9d05ed49f6674b2d721e07235b26c5cea4
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574758"
---
# <a name="enterprise-productivity-bot-scenario"></a>Enterprise Productivity ボットのシナリオ

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Enterprise ボットは、Office 365 カレンダーやその他のサービスとボットを統合することによって生産性を向上させる方法を示しています。

多数のウィンドウを開かずに迅速に顧客情報にアクセスできるようにすることが Enterprise Productivity ボットの目的です。 単純なチャット コマンドを使用して、営業担当者は顧客を検索し、次回の予約を Graph API と Office 365 で確認できます。 そこから、Dynamics CRM に格納されている顧客固有の情報にアクセスし、案件を取得したり、新たに作成したりすることができます。

![Enterprise ボットのダイアグラム](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

次に、Enterprise Productivity ボットのロジック フローを示します。

1. 従業員が Enterprise Productivity ボットにアクセスします。
2. Azure Active Directory が従業員の ID を確認します。
3. Enterprise Productivity ボットは、Azure Graph を介して従業員の Office 365 カレンダーに対してクエリを実行することができます。
4. カレンダーから収集したデータを使用して、ボットが Dynamics CRM 内の案件情報にアクセスします。
5. 従業員に情報が返され、その従業員はボットから離れずにデータを絞り込むことができます。
6. Application Insights がランタイム テレメトリを収集して、ボットのパフォーマンスと使用状況の情報によって開発をサポートします。

このサンプル ボットのソース コードは、「[Samples for Common Bot Framework Scenarios (Bot Framework の一般的なシナリオのサンプル)](https://aka.ms/bot/scenarios)」からダウンロードするか複製することができます。

## <a name="sample-bot"></a>サンプル ボット
ボットにはさまざまなチャネルからアクセスできるため、デスクで会社のポータルから使用したり、外出先で Skype から使用したりすることができます。必要なのは、認証を受けることだけです。 Azure AD の統合により、Enterprise Productivity ボットは、ボットにアクセスできるユーザーであれば Azure AD によって認証されていることを把握しています。 そこから、特定の顧客との次回の予定を確認するようにボットに依頼することができます。 ボットは、Graph API を介して Office 365 に対してクエリを実行することでこの情報を取得します。 その後、今後 7 日以内に予定があれば、ボットは CRM に対してクエリを実行し、顧客の最近の案件を探します。 ボットは、案件が見つからなかったと応答するか、オープン中またはクローズ済みの案件の数を返します。 そこから、種類ごとの案件の一覧を作成し、個別の案件の詳細を表示するようにボットに依頼することができます。

## <a name="components-youll-use"></a>使用するコンポーネント
Enterprise Productivity ボットは、次のコンポーネントを使用します。
-   認証用の Azure AD
-   Office 365 への Graph API
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) は、マイクロソフトが提供する、マルチテナントに対応したクラウド ベースのディレクトリと ID の管理サービスです。 また、Azure AD は世界中の数多くの組織が使用する卓越した ID 管理ソリューションとすばやく簡単に統合できるため、ボット開発者はボットの構築に集中することができます。 独自の複雑な認証および承認システムを実装しなくても、Azure AD アプリを定義することによって、ボットおよびボットが公開するデータにだれがアクセスできるかを制御できます。

### <a name="graph-api-to-office-365"></a>Office 365 への Graph API
Microsoft Graph は、Office 365 およびその他の Microsoft クラウド サービスの複数の API を https://graph.microsoft.com の単一エンドポイント経由で公開しています。 Microsoft Graph を使用すると、開発者とボットは、より簡単にクエリを実行することができます。 API は、Office 365 の一部としての Exchange Online、Azure Active Directory、SharePoint など、複数の Microsoft クラウド サービスのデータを公開します。 API を使用して、エンティティとリレーションシップ間を移動することができます。 SDK または REST エンドポイントを使用してボットの API を使用することも、Android、iOS、Ruby、UWP、Xamarin などのネイティブ サポートを備えた他のアプリの API を使用することもできます。

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM は、顧客エンゲージメント プラットフォームです。 ボットと CRM の API を使用すると、CRM に格納されている豊富なデータにアクセスできる、多機能な対話的ボットを構築できます。 ボットで、案件の作成、状態の確認、ナレッジ マネジメントの検索などのために、Dynamics CRM の機能を利用できます。

### <a name="application-insights"></a>Application Insights
Application Insights では、アプリケーション パフォーマンス管理 (APM) と瞬時の分析によって、行動につながる分析情報を得ることができます。 すぐに使用できる、多機能なパフォーマンス監視、強力なアラート機能、使いやすいダッシュボードによって、ボットの可用性が保たれ、期待どおりに動作していることを確認できます。 問題の発生をすばやく確認し、根本原因を分析して、問題個所を特定し、修正することができます。

## <a name="next-steps"></a>次の手順
次に、Information ボットのシナリオについて学習します。

> [!div class="nextstepaction"]
> [Information ボットのシナリオ](bot-service-scenario-informational.md)
