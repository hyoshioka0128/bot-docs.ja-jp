---
title: ボットへのトレース アクティビティの追加 - Bot Service
description: Bot Framework SDK のトレース アクティビティとその使用方法について説明します。
keywords: トレース, アクティビティ, ボット, Bot Framework SDK
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5cfb4e8f2d16c868bcbc7d3d11c7cb0b2455f7cb
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "76895820"
---
# <a name="add-trace-activities-to-your-bot"></a>ボットへのトレース アクティビティの追加

[!INCLUDE[applies-to](../includes/applies-to.md)]

<!-- What is it and why use it -->

_トレース アクティビティ_ は、ボットが Bot Framework Emulator に送信できるアクティビティです。
トレース アクティビティを使用すると、ボットをローカルで実行している間に情報を表示することができるため、ボットをインタラクティブにデバッグできます。

<!-- Details -->

トレース アクティビティはエミュレーターにのみ送信され、他のクライアントまたはチャネルには送信されません。
エミュレーターでは、メインのチャット パネルではなく、ログに表示されます。

- ターン コンテキストを介して送信されたトレース アクティビティは、ターン コンテキストに登録された_送信アクティビティ ハンドラー_を介して送信されます。
- ターン コンテキストを介して送信されたトレース アクティビティは、会話リファレンスがあった場合にそれを適用し、受信アクティビティに関連付けられています。
  プロアクティブ メッセージの場合、 _[reply to ID]\(返信先 ID)_ は新しい GUID になります。
- トレース アクティビティでは、送信方法に関係なく、 _[responded]\(返信済み\)_ フラグが設定されることはありません。

## <a name="to-use-a-trace-activity"></a>トレース アクティビティを使用するには

エミュレーターでトレース アクティビティを表示するには、例外をスローし、アダプターのオン ターン エラー ハンドラーからトレース アクティビティを送信するなど、ボットがトレース アクティビティを送信するシナリオが必要です。

ボットからトレース アクティビティを送信するには:

1. 新しいアクティビティを作成します。
   - _type_ プロパティを "trace" に設定します。 これは必須です。
   - 必要に応じて、トレースに対応する、_name_、_label_、_value_、および _value type_ プロパティを設定します。
1. _turn context_ オブジェクトの _send activity_ メソッドを使用して、トレース アクティビティを送信します。
   - このメソッドは、受信アクティビティに基づいて、アクティビティの残りの必須プロパティの値を追加します。
     これには、_channel ID_、_service URL_、_from_、および _recipient_ プロパティが含まれます。

エミュレーターでトレース アクティビティを表示するには:

1. お使いのマシン上でローカルでボットを実行します。
1. エミュレーターを使ってテストします。
   - ボットと対話し、シナリオのステップを使用してトレース アクティビティを生成します。
   - ボットがトレース アクティビティを出力すると、トレース アクティビティがエミュレーター ログに表示されます。

ここでは、ボットが依存している QnAMaker ナレッジ ベースを最初に設定することなく、コアボットを実行した場合に表示されるトレース アクティビティを示します。

![エミュレーターのスクリーン ショット](./media/using-trace-activities.png)

## <a name="add-a-trace-activity-to-the-adapters-on-error-handler"></a>アダプターのオンエラー ハンドラーにトレース アクティビティを追加する

アダプターの _オン ターン エラー_ ハンドラーは、ターン中にボットからスローされ、その他の方法ではキャッチされない例外をキャッチします。
これは、わかりやすいメッセージをユーザーに送信し、例外に関するデバッグ情報をエミュレーターに送信できるため、トレース アクティビティに適しています。

このコード例は、**コア ボット** サンプルからのものです。 [**C#** ](https://aka.ms/cs-core-sample)、[**JavaScript**](https://aka.ms/js-core-sample)、または [**Python**](https://aka.ms/py-core-sample) の完全なサンプルを参照してください。

# <a name="c"></a>[C#](#tab/csharp)

アダプターの **OnTurnError** ハンドラーは、例外情報を含むトレース アクティビティを作成して、エミュレーターに送信します。

**AdapterWithErrorHandler.cs**

[!code-csharp[OnTurnError](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=16-51&highlight=33-34)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

アダプターの **onTurnError** ハンドラーは、例外情報を含むトレース アクティビティを作成して、エミュレーターに送信します。

**index.js**

[!code-javascript[onTurnError](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/index.js?range=35-57&highlight=8-14)]

# <a name="python"></a>[Python](#tab/python)

アダプターの **on_error** ハンドラーは、例外情報を含むトレース アクティビティを作成して、エミュレーターに送信します。

**adapter_with_error_handler.py**

[!code-python[on_error](~/../BotBuilder-Samples/samples/python/13.core-bot/adapter_with_error_handler.py?range=26-50&highlight=24-25)]

---

## <a name="additional-resources"></a>その他のリソース

- [検査ミドルウェアを使用してボットをデバッグする](../bot-service-debug-inspection-middleware.md)方法では、トレース アクティビティを出力するミドルウェアを追加する方法について説明します。
- デプロイされたボットをデバッグする場合は、Application Insights を使用できます。 詳細については、「[ボットへのテレメトリの追加](bot-builder-telemetry.md)」を参照してください。
- アクティビティの種類の詳細については、[ボットのフレームワークのアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)に関するページをご覧ください。
