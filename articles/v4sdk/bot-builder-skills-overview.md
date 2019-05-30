---
title: Bot Framework のスキルの概要 | Microsoft Docs
description: Bot Framework のスキルの詳細について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8ed5841f9bfe874de26f1aecbb0e4460e541668b
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215297"
---
# <a name="virtual-assistant---skills-overview"></a>仮想アシスタント - スキルの概要

> [!NOTE]
> このトピックは、SDK の v4 バージョンに適用されます。 

## <a name="overview"></a>概要

開発者が会話エクスペリエンスを作成するには、"スキル" と呼ばれる再利用可能な会話機能を結合します。

企業内では、これにより 1 つの親ボットが作成され、さまざまなチームが所有する複数のサブボットがまとめられています。または、他の開発者が提供する一般的な機能が幅広く利用されています。 このスキル プレビューを使用すると、開発者が新しいボットを作成し (通常は仮想アシスタント テンプレートを使用)、1 つのコマンドライン操作でスキルを追加/削除して、すべてのディスパッチおよび構成変更を組み込むことができます。     

スキル自体がボットであり、リモートで呼び出されます。また、新しいスキルの作成を容易にするためのスキル開発者向けテンプレート (.NET、TS) もあります。

スキルの主な設計目標は、一貫性のあるアクティビティ プロトコルを維持し、開発エクスペリエンスを通常の V4 SDK ボットにできるだけ近付けることでした。 

![スキルのシナリオ](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Bot Framework のスキル

現在、Microsoft Graph によって次の Bot Framework スキルを複数の言語で利用できます。

![スキルのシナリオ](./media/enterprise-template/skills-at-build.png)

| Name | 説明 |
| ---- | ----------- |
|[カレンダー スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-calendar.md)|カレンダー機能をご自身のアシスタントに追加します。 Microsoft Graph および Google を使用。|
|[メール スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-email.md)|メール機能をご自身のアシスタントに追加します。 Microsoft Graph および Google を使用。|
|[To Do スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-todo.md)|タスク管理機能をご自身のアシスタントに追加します。 Microsoft Graph を使用。|
|[目的地スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-pointofinterest.md)|目的地と方向を検索します。 Azure Maps と FourSquare を使用。|
|[自動車スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/automotive.md)|自動車機能のコントロールを示す業界垂直スキル。|
|[試験段階スキル](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/experimental.md)|ニュース、レストラン予約、および天気。|

## <a name="getting-started"></a>Getting Started (概要)

既存のスキルを利用して独自のスキルを構築する方法については、[作業の開始](https://github.com/Microsoft/AI/tree/master/docs#tutorials)に関するページをご覧ください。
