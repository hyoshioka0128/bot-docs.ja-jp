---
title: Bot Framework のスキルの概要 | Microsoft Docs
description: Bot Framework のスキルの詳細について説明します
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9bb149773fe1b2cdd09da0165a4fff4e647f419d
ms.sourcegitcommit: 5d81c5b25ea56e04f09d05916ee947d631009172
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/10/2019
ms.locfileid: "72236434"
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

| 名前 | 説明 |
| ---- | ----------- |
|[カレンダー スキル](https://aka.ms/bf-calendar-skill)|カレンダー機能をご自身のアシスタントに追加します。 Microsoft Graph および Google を使用。|
|[メール スキル](https://aka.ms/bf-email-skill)|メール機能をご自身のアシスタントに追加します。 Microsoft Graph および Google を使用。|
|[To Do スキル](https://aka.ms/bf-todo-skill)|タスク管理機能をご自身のアシスタントに追加します。 Microsoft Graph を使用。|
|[目的地スキル](https://aka.ms/bf-poi-skill)|目的地と方向を検索します。 Azure Maps と FourSquare を使用。|
|[自動車スキル](https://aka.ms/bf-auto-skill)|自動車機能のコントロールを示す業界垂直スキル。|
|[試験段階スキル](https://aka.ms/bf-experimental-skills)|ニュース、レストラン予約、および天気。|

## <a name="getting-started"></a>Getting Started (概要)

既存のスキルを利用して独自のスキルを構築する方法については、[チュートリアル](https://aka.ms/bfs-tutorials)をご覧ください。
