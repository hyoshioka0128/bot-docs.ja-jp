---
title: Azure Bot Framework コマンド ライン インターフェイス (CLI) の概要 - Bot Service
description: Bot Framework コマンド ライン インターフェイス (CLI) について説明します。
keywords: Bot Framework コマンド ライン インターフェイス, Bot Framework CLI
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8b885b19ed22c4d91163b59abe4e253018531b59
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791429"
---
<!--TODO:
- [?] Add to TOC: Reference/Bot Framework CLI/Reference
- [?] Add other topics to the same node for each of the command groups
-->
# <a name="bot-framework-cli-overview"></a>Bot Framework CLI の概要

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot Framework コマンド ライン インターフェイス (CLI) は、ボットと関連サービスを管理するためのクロスプラットフォーム ツールです。 これにより、古いスタンドアロン CLI ツールのコレクションが、1 つのツールに集約されて置き換えられます。 

## <a name="prerequisites"></a>前提条件

* [Node.js](https://nodejs.org/) のバージョン 10.14.1 以降。

## <a name="installation"></a>インストール

コマンドラインから BF CLI をインストールします。

~~~cmd
npm i -g @microsoft/botframework-cli
~~~

## <a name="available-commands"></a>使用可能なコマンド

現在、次のコマンドを使用できます。

| 古いツール | BF コマンド セット | 説明 |
| :--- | :--- | :--- |
| ChatDown | [`bf chatdown`](bf-cli-reference.md#bf-chatdown) | チャット ダイアログ ( **.chat**) ファイルを操作するためのコマンドです。 |
| NA | [`bf config`](bf-cli-reference.md#bf-config) | CLI 内のさまざまな設定を構成します。 |
| LuDown、LuisGen | [`bf luis`](bf-cli-reference.md#bf-luis) | LUIS リソース ファイルを操作したり、LUIS モデルを管理したりするためのコマンドです。 |
| QnA Maker | [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) | QnA Maker リソース ファイルを操作したり、ナレッジ ベースを管理したりするためのコマンドです。 |

今後のリリースでは、次のツールが移植されます。
- LUIS (API)
- ディスパッチ

以前のツールと新しいツールのマッピングの参照については、[移植マップ](https://github.com/microsoft/botframework-cli/blob/master/PortingMap.md)に関するページを参照してください。

_注意事項: 以前の CLI ツールは今後のリリースでは非推奨となる予定であり、これらのツールのサポートは将来終了します。この領域の新しい投資、バグ修正、新機能はすべて、BF CLI のみを対象とします。_

## <a name="overview"></a>概要

BF CLI はボットと関連サービスを管理します。 エンタープライズ級の会話型 AI エクスペリエンスを構築するための包括的なプラットフォームである Microsoft Bot Framework の一部です。 BF CLI は、ボット関連のリソースを管理するだけでなく、継続的インテグレーションと継続的デプロイ (CI/CD) パイプラインの一部としても使用できます。 ボットを構築する際には、言語を解釈するための LUIS のような AI サービス、ボットが Q&A 形式で簡単な質問に回答できるようにするための QnAMaker を統合する必要も出てくるかもしれません。 ボットに AI サービスを統合するには、以下を使用します。

* [`bf luis`](bf-cli-reference.md#bf-luis) コマンドを使用して LUIS **.lu** リソース ファイルを操作し、LUIS モデルを管理します。 また、対応するソース (C# または JavaScript) コードを生成することもできます。
* [LUIS API ツール](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS/readme.md) を使用して、ローカル ファイルのデプロイ、LUIS サービス内の Language Understanding モデルとしてのトレーニング、テスト、発行を行います。
* [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) コマンドを使用して、QnAMaker ナレッジ ベースを操作します。 QnA Maker の資産をローカルと QnA Maker サービスの両方で作成および管理できます。

* **.lu** および **.qna** ファイル形式を使用する方法については、lu ライブラリの[ドキュメント](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/README.md)を参照してください。

ボットがより高度になると、[ディスパッチ](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) CLI ツールを使用して、複数の LUIS モデルと QnA Maker のナレッジ ベースに対して意図を作成、評価、およびディスパッチできます。

新しい [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) を使用すると、お使いのボットのテストおよび調整を行うことができます。 エミュレーターを使用すると、ローカル コンピューターまたはクラウドでボットをテストおよびデバッグできます。

初期の設計段階では、ボットがサポートする特定のシナリオについて、ユーザーとボットの間にモックの会話を作成する必要が生じる場合があります。 [`bf chatdown`](bf-cli-reference.md#bf-chatdown) コマンドを使用して、会話のモックアップ **.chat** ファイルを作成し、豊富なトランスクリプトに変換し、エミュレーターで会話を表示します。

最後に、[Azure CLI](https://github.com/microsoft/botframework-cli/blob/master/AzureCli.md) (`az bot` コマンド) を使用して、[Azure Bot Service](https://azure.microsoft.com/services/bot-service/) でチャネルの作成、ダウンロード、発行、および構成を行うことができます。 これは、[Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) の機能を拡張して Azure Bot Service の資産を管理するプラグインです。

## <a name="privacy-and-instrumentation"></a>プライバシーおよびインストルメンテーション
BF CLI には、**匿名** の使用パターンに基づいてツールを改善するために設計されたインストルメンテーション オプションが含まれています。 __既定では無効で、オプトアウトされています__。 オプトインを選択した場合、Microsoft はいくつかの使用状況データを収集します。

* コマンド グループの呼び出し
* 特定の値を**除外**した、使用されたフラグ。 たとえば、パラメーター `--folder:name` の場合、`--folder` の使用のみが収集され、フォルダー名は収集されません。

データ コレクションの動作を変更するには、[`bf config`](bf-cli-reference.md#bf-config) コマンドを使用します。

詳細については、「[Microsoft のプライバシーに関する声明](https://privacy.microsoft.com/privacystatement)」を参照してください。  

## <a name="issues-and-feature-requests"></a>イシューと機能に関する要求
- イシューの報告と機能の要求は、[こちら](https://github.com/microsoft/botframework-cli/issues)で行うことができます。
- 既知の問題については[こちら](https://github.com/microsoft/botframework-cli/labels/known-issues)を参照してください。

## <a name="next-steps"></a>次のステップ
- [BF CLI のリファレンス](bf-cli-reference.md)
