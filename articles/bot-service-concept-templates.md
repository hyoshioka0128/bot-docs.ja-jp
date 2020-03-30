---
title: Bot Service テンプレート - Bot Service
description: Bot Service でボットを作成するときに使用できるさまざまなテンプレートについて説明します。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 82afb54b7f4a148aee35be15fac761e2f89aed69
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75793083"
---
# <a name="bot-service-templates"></a>Bot Service テンプレート

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Service には、ボットの作成を開始するために役立つ 5 つのテンプレートが含まれています。 これらのテンプレートでは、すばやく開始するのに役立つ、すぐに使用できる完全に機能するボットが提供されます。 [ボットを作成する](bot-service-quickstart.md)ときに、ご自分のボット用のテンプレートと SDK 言語を選択します。

各テンプレートでは、ボットのコア機能に基づく開始点が提供されます。 

## <a name="basic-bot"></a>基本ボット
ユーザー入力に応答するためにダイアログを使用するボットを作成するには、基本テンプレートを選択します。 **基本**テンプレートでは、作業を開始するために必要な最小セットのファイルとコードが用意されているボットが作成されます。 ユーザーが入力するたびに、ボットによってエコーで返されます。 このテンプレートを使用して、ご利用のボットでの会話フローの作成を開始できます。

## <a name="form-bot"></a>フォーム ボット
ガイド付きの会話を通じてユーザーからの入力を収集するボットを作成するには、**フォーム** テンプレートを選択します。 フォーム ボットは、ユーザーから特定の情報セットを収集するように設計されています。 たとえば、ユーザーのサンドウィッチの注文を受けるように設計されているボットでは、パンの種類、トッピングの選択、サンドウィッチのサイズなどの情報を収集する必要があります。

C# 言語のフォーム テンプレートで作成されたボットでは、[FormFlow](dotnet/bot-builder-dotnet-formflow.md) を使用してフォームが管理され、Node.js 言語のフォーム テンプレートで作成されたボットでは、[ウォーターフォール](nodejs/bot-builder-nodejs-dialog-waterfall.md)を使用してフォームが管理されます。

## <a name="language-understanding-bot"></a>Language Understanding ボット
ユーザーの意図を理解するために自然言語モデルを使用するボットを作成するには、**Language Understanding** テンプレートを選択します。 このテンプレートでは <a href="https://www.luis.ai" target="_blank">Language Understanding (LUIS)</a> を活用して、自然言語を理解できます。

ユーザーが "仮想現実会社に関するニュースを入手する" などのメッセージを送信した場合、ご利用のボットでは LUIS を使用して、そのメッセージの意味を解釈できます。 LUIS を使用することで、込められた意図 (ニュースの検索) と存在する主要エンティティ (仮想現実会社) という観点で、ユーザー入力を解釈する HTTP エンドポイントをすばやくデプロイできます。 LUIS では、ご利用のアプリケーションに関連するエンティティと意図のセットを指定することができ、Language Understanding アプリケーションの作成プロセスをガイドします。

Language Understanding テンプレートを使用してボットを作成するときに、Bot Service では、対応する空の (つまり、常に `None` を返す) LUIS アプリケーションが作成されます。 LUIS アプリケーション モデルを更新して、ユーザー入力を解釈できるようにするには、<a href="https://www.luis.ai" target="_blank">LUIS</a> にサインインして、 **[マイ アプリケーション]** をクリックし、サービスで作成されたアプリケーションを選択してから意図を作成し、エンティティを指定してアプリケーションをトレーニングします。

## <a name="question-and-answer-bot"></a>質疑応答ボット
質問と回答のペアなどの半構造化データを個別の役立つ回答に抽出するボットを作成するには、**質疑応答**テンプレートを選択します。 このテンプレートでは <a href="https://qnamaker.ai">QnA Maker</a> サービスを活用して、質問を解析し、回答を提供します。 

質疑応答テンプレートでボットを作成するときは、<a href="https://qnamaker.ai">QnA Maker</a> にサインインして、新しい QnA サービスを作成する必要があります。 この QnA サービスによって、ナレッジ ベース ID とサブスクリプション キーが与えられます。それらを使用して、**QnAKnowldegebasedId** と **QnASubscriptionKey** の[アプリ設定](bot-service-manage-settings.md)の値を更新することができます。 これらの値が設定されたら、ご利用のボットで、QnA サービスのナレッジ ベースにあるすべての質問に回答することができます。

## <a name="proactive-bot"></a>プロアクティブ ボット
ユーザーにプロアクティブ メッセージを送信できるボットを作成するには、**プロアクティブ テンプレート**を選択します。 通常、ボットがユーザーに送信する各メッセージは、ユーザーの前の入力に直接関連しています。 しかし、場合によっては、ユーザーの最新メッセージに直接関連しないユーザー情報をボットから送信する必要があります。 この種のメッセージは、**プロアクティブ メッセージ**と呼ばれます。 プロアクティブ メッセージは、さまざまなシナリオで役立ちます。 たとえば、ボットでタイマーやリマインダーを設定する場合、その時刻になったときにユーザーに通知する必要があります。 あるいは、ボットで外部イベントに関する通知が受信された場合に、その情報をユーザーに伝える必要があります。 

プロアクティブ テンプレートを使用してボットを作成するときは、いくつかの Azure リソースが自動的に作成され、ご利用のリソース グループに追加されます。 既定では、非常にシンプルなプロアクティブ メッセージング シナリオを有効にするように、これらの Azure リソースが既に構成されています。 

| リソース | [説明] |
|----|----|
| Azure Storage | キューを作成するために使用されます。 |
| Azure Function App | キューにメッセージがある場合は常にトリガーされる `queueTrigger` Azure 関数です。 [ダイレクト ライン](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts)を使用して、Bot Service と通信します。 この関数ではボット バインドを使用して、トリガーのペイロードの一部としてメッセージが送信されます。 この関数例では、キューからユーザーのメッセージがそのまま転送されます。
| ボット サービス | ご利用のボットです。 ユーザーからメッセージを受信し、Azure キューにそのメッセージを追加し、Azure 関数からトリガーを受信し、トリガーのペイロードで受信されたメッセージを返送するロジックが含まれます。 |

次の図には、プロアクティブ テンプレートを使用してボットを作成するときに、トリガーされたイベントがどのように動作するかが示されています。

![プロアクティブ ボット例の概要](~/media/bot-proactive-diagram.png)

ユーザーが Bot Framework サーバーを介してボットにメッセージを送信する (手順 1) ときに、プロセスが開始されます。

## <a name="next-steps"></a>次のステップ
これでさまざまなテンプレートについて理解できました。次は、ボット内で使用する Cognitive Services について説明します。

> [!div class="nextstepaction"]
> [ボットの Cognitive Services](bot-service-concept-intelligence.md)
