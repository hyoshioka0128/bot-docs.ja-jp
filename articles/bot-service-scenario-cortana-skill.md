---
title: Cortana スキル ボットのシナリオ | Microsoft Docs
description: Bot Framework による Cortana スキル ボットのシナリオについて説明します。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7676b7bd75a45130b62c1a691499095d6ba07291
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000229"
---
# <a name="cortana-skills-bot-scenario"></a>Cortana スキル ボットのシナリオ

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Cortana スキル ボットによって Cortana が拡張され、出張自動車整備の予約を、音声を使用して予定表のコンテキストから簡単に行うことができるようになります。

Cortana は、ユーザーのパーソナル アシスタントです。 ユーザーの声とカスタム Cortana スキル ボットの自然なインターフェイスを使用して、自動車整備店などの組織に連絡して予約を入れるように Cortana に頼むことができます。 このサービスでは、サービス、利用可能な時間帯、および時間の長さの一覧を指定できます。 Cortana は、ユーザーの予定表を見て競合する予定があるかどうかを確認し、ない場合は、予定を作成して予定表に追加することができます。

![Cortana スキル ボットのダイアグラム](~/media/scenarios/bot-service-scenario-cortana-skill.png)

次に、自動車整備店用の Cortana スキル ボットのロジック フローを示します。

1. ユーザーは自分の PC またはモバイル デバイスから Cortana にアクセスします。
2. ユーザーは、テキストまたは音声のコマンドを使用して、自動車整備店の予約を要求します。
3. このボットは Cortana と統合されているため、ユーザーの予定表にアクセスし、ロジックをこの要求に適用します。
4. ボットは、その情報を使用して自動車整備店に対して、有効な予定を照会できます。
5. コンテキストの選択肢が表示されたら、ユーザーは予定を予約できます。
6. Application Insights が、ボットのパフォーマンスと使用法について開発に役立つランタイム テレメトリを収集します。

## <a name="sample-bot"></a>サンプル ボット
Cortana スキル ボットは、個々人のコンテキストで動作します。 Cortana を使用すると、ご自分の位置情報を基に、ご自分の音声を使って "ボブの出張整備" に車のメンテナンスに来るように依頼できます。 ボットは、Cortana を介して公開される個人情報を使用して、ユーザーがボットに話しかけているときの場所を基に位置情報を確認できます。

このサンプル ボットのソース コードは、[Bot Framework の一般的なシナリオのサンプル](https://aka.ms/bot/scenarios)からダウンロードまたは複製できます。

## <a name="components-youll-use"></a>使用するコンポーネント
Cortana ボットでは、次のコンポーネントを使用します。
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
これで Cortana スキルを作成して、ボットにサポートを追加できるようになりました。 Cortana の新機能 (スキルと呼ばれます) を構築するには、Cortana スキル キットを使用します。 スキルとは、Cortana がより多くのことを実行できるようにするコンストラクトです。 ボットと統合するスキルを構築して、Cortana がタスクを完了したり、作業を実行したりできるようにします。 Cortana は、呼び出しプロセスの一環として、ユーザーに関する情報を (ユーザーの同意を得て) 実行時にスキルに渡します。スキルは、それに応じてそのエクスペリエンスをカスタマイズできます。 Cortana のコンテキストの知識によって、ボットは便利になり、その分さらに賢くなります。 呼び出された特定の種類のスキルによって Cortana のインターフェイスが操作され、スキルとエンドユーザーの間で会話ができます。 公開されると、ユーザーは、Windows 10 Anniversary Update 以降 (デスクトップおよびモバイル)、iOS、および Android 用の Cortana でスキルを表示して使用できます。

### <a name="application-insights"></a>Application Insights
Application Insights では、アプリケーション パフォーマンス管理 (APM) と瞬時の分析によって、行動につながる分析情報を得ることができます。 すぐに使用できる、多機能なパフォーマンス監視、強力なアラート機能、使いやすいダッシュボードによって、ボットの可用性が保たれ、期待どおりに動作していることを確認できます。 問題の発生をすばやく確認し、根本原因を分析して、問題個所を特定し、修正することができます。

## <a name="next-steps"></a>次の手順
次に、Enterprise Productivity ボットのシナリオについて説明します。

> [!div class="nextstepaction"]
> [Enterprise Productivity ボットのシナリオ](bot-service-scenario-enterprise-productivity.md)
