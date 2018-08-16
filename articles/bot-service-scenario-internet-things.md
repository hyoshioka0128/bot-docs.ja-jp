---
title: モノのインターネット ボットのシナリオ | Microsoft Docs
description: Bot Framework によるモノのインターネット ボットのシナリオについて説明します。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3583c4382a9262b959b31d5a9a1b7b1b97fc8a04
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301828"
---
# <a name="internet-of-things-iot-bot-scenario"></a>モノのインターネット (IoT) ボットのシナリオ
このモノのインターネット (IoT) ボットによって、音声または対話型チャット コマンドを使用して、Philips Hue の照明などの自宅周辺のデバイスを簡単に制御できます。

人は自分のものに話しかけるのを好みます。 最初のテレビ リモコンが出現した日以来、移動せずに環境に影響を与えることが人々のお気に入りになっています。 この IoT ボットを使用すると、簡単なチャット コマンドまたは音声によって Philips Hue を管理できます。 さらに、チャットを使用した場合、ユーザーは選択する色に関連した視覚的な選択肢が与えられます。

![モノのインターネット ボットのダイアグラム](~/media/scenarios/bot-service-scenario-iot-bot.png)

次に、IoT ボットのロジック フローを示します。

1. ユーザーは Skype にログインし、IoT ボットにアクセスします。
2. ユーザーが IoT デバイスを介して音声でボットに照明をつけるよう要求します。
3. この要求が、IoT デバイス ネットワークへのアクセスが許可されているサードパーティ サービスに伝達されます。
4. コマンドの実行結果がユーザーに返されます。
5. Application Insights が、ボットのパフォーマンスと使用法について開発に役立つランタイム テレメトリを収集します。

## <a name="sample-bot"></a>サンプル ボット
IoT ボットを使用すると、Skype や Slack のようなチャネルから手軽にチャット コマンドを使用して Hue を制御できるようになります。 リモート アクセスを容易にするには、Hue と連携するために事前定義された IFTTT アプレットを呼び出します。

このサンプル ボットのソース コードは、[Bot Framework の一般的なシナリオのサンプル](https://aka.ms/bot/scenarios)からダウンロードまたは複製できます。

## <a name="components-youll-use"></a>使用するコンポーネント
モノのインターネット (IoT) ボットでは、次のコンポーネントを使用します。
-   Philips Hue
-   If This Then That (IFTTT)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
Philips Hue に接続されている電球とブリッジによって、照明を完全に制御できます。 照明に望むあらゆることを Hue が実行できます。 Hue には、ご自身のローカル ネットワークから使用できる API があります。 しかし、ボットの使いやすいインターフェイスを使用して、Hue が制御するデバイスと照明にどこからでもアクセスできることをユーザーは望みます。 そこで、IFTTT を介して Hue にアクセスします。

### <a name="ifttt"></a>IFTTT
IFTTT は無料の Web ベース サービスで、ユーザーはこれを使ってアプレットと呼ばれる単純な条件付きステートメントのチェーンを作成できます。 ボットからアプレットをトリガーし、自分の代わりに作業させることができます。 照明のオンとオフ、シーンの変更などに使用可能な事前定義済みの Hue アプレットが数多くあります。

### <a name="application-insights"></a>Application Insights
Application Insights では、アプリケーションのパフォーマンス管理 (APM) と瞬時の分析によって、行動につながる分析情報の取得が支援されます。 事前設定なしで、ボットが期待通りに動作し可用性が確保されるように、多彩なパフォーマンス監視、強力なアラート機能、使いやすいダッシュボードが用意されています。 問題が発生しているかどうかをすばやく確認し、根本原因分析を行って、問題を検出し、修正できます。
