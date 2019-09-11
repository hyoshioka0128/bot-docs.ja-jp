---
title: 支払いを要求する | Microsoft Docs
description: Bot Framework SDK for .NET を使用して支払い要求を送信する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 23dd1c86d2605c50fafc572adcf9ca4369850131
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297854"
---
# <a name="request-payment"></a>支払いの要求

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-request-payment.md)

ボットでユーザーがアイテムを購入できるようになっている場合、[リッチ カード](bot-builder-dotnet-add-rich-card-attachments.md)に特殊な種類のボタンを含めて、支払いを要求することができます。 この記事では、Bot Framework SDK for .NET を使用して支払い要求を送信する方法について説明します。

## <a name="prerequisites"></a>前提条件

Bot Framework SDK for .NET を使用して支払い要求を送信するには、次の前提条件のタスクを完了しておく必要があります。

### <a name="update-webconfig"></a>Web.config の更新

ボットの **Web.config** ファイルを更新して、`MicrosoftAppId` と `MicrosoftAppPassword` を、[登録](~/bot-service-quickstart-registration.md)プロセス中にボットのために生成されたアプリ ID とパスワードの値に設定します。 

> [!NOTE]
> ボットの **AppID** と **AppPassword** を見つけるには、「[MicrosoftAppID と MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」を参照してください。

### <a name="create-and-configure-merchant-account"></a>マーチャント アカウントを作成して構成する

1. <a href="https://dashboard.stripe.com/register" target="_blank">まだお持ちでない場合は、Stripe アカウントを作成してアクティブにします。</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Microsoft アカウントを使用して Seller Center にサインインします。</a>

3. Seller Center 内で、そのアカウントを Stripe に接続します。

4. Seller Center 内で、Dashboard に移動し、**MerchantID** の値をコピーします。

5. **Web.config** ファイルを更新して、`MerchantId` を Seller Center Dashboard からコピーした値に設定します。 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>支払いボット サンプル

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支払いボット</a> サンプルは、.NET を使用して支払い要求を送信するボットの例を提供します。 このサンプル ボットの動作を確認するには、<a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">それを Web チャットで試すか</a>、<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">Skype 連絡先として追加するか</a>、支払いボット サンプルをダウンロードしてから Bot Framework エミュレーターを使用してローカルで実行することができます。 

> [!NOTE]
> Web チャットまたは Skype で**支払いボット** サンプルを使用してエンド ツー エンドの支払いプロセスを完了するには、Microsoft アカウント内で有効なクレジット カードまたはデビット カード (つまり、米国のカード発行者からの有効なカード) を指定する必要があります。 **支払いボット** サンプルはテスト モードで実行される (つまり、**Web.config** で `LiveMode` が `false` に設定されている) ため、カードに課金されたり、カードの CVV が検証されたりすることはありません。

この記事の次のいくつかのセクションでは、**支払いボット** サンプルのコンテキストでの支払いプロセスの 3 つの部分について説明します。

## <a id="request-payment"></a> 支払いの要求

ボットは、"支払い" の `CardAction.Type` を指定するボタンが含まれた[リッチ カード添付ファイル](bot-builder-dotnet-add-rich-card-attachments.md)を含むメッセージを送信して、ユーザーに支払いを要求できます。 **支払いボット** サンプルからのこのコード スニペットは、ユーザーが支払いプロセスを開始するためにクリック (またはタップ) できる **[Buy]** ボタンが含まれた Hero カードを含むメッセージを作成します。 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

この例では、ボタンの種類は、Bot Builder ライブラリが "支払い" として定義する `PaymentRequest.PaymentActionType` として指定されています。 このボタンの値は、`BuildPaymentRequest` メソッドによって入力されます。このメソッドは、サポートされる支払い方法、詳細、オプションに関する情報が入った `PaymentRequest` オブジェクトを返します。 実装の詳細については、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支払いボット</a> サンプル内の **Dialogs/RootDialog.cs** を参照してください。

次のスクリーンショットは、上のコード スニペットによって生成される ( **[Buy]** ボタンが含まれた) Hero カードを示しています。 
 
![支払いのサンプル ボット](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> **[Buy]** ボタンにアクセスできるユーザーはすべて、そのボタンを使用して支払いプロセスを開始できます。 グループ会話のコンテキスト内では、特定のユーザーだけが使用するボタンを指定することはできません。 

## <a id="user-experience"></a> ユーザー エクスペリエンス

**[Buy]** ボタンをクリックすると、ユーザーは支払いの Web エクスペリエンスに移動し、Microsoft アカウントを通して、支払い、配送、連絡先に関する必要な情報すべてを指定できます。 

![Microsoft での支払い](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP コールバック

ボットが特定の操作を実行すべきであることを示すために、HTTP コールバックがボットに送信されます。 各コールバックが、次のプロパティ値を含む[アクティビティ](bot-builder-dotnet-activities.md)になります。 

| プロパティ | 値 |
|----|----|
| `Activity.Type` | invoke | 
| `Activity.Name` | ボットが実行すべき操作の種類 (配送先住所の更新、配送オプションの更新、支払いの完了など) を示します。 | 
| `Activity.Value` | JSON 形式の要求ペイロード。 | 
| `Activity.RelatesTo` |  支払い要求に関連付けられているチャネルとユーザーを記述します。 | 

> [!NOTE]
> `invoke` は、Microsoft Bot Framework で使用するために予約されている特殊なアクティビティの種類です。 `invoke` アクティビティを送信した側は、ボットがコールバックを確認するために HTTP 応答を送信するものと予期します。

## <a id="process-callbacks"></a> コールバックの処理

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>配送先住所の更新コールバックと配送オプションの更新コールバック

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>支払いの完了コールバック

支払いの完了コールバックの受信時、変更されていない最初の支払い要求と支払い応答オブジェクトのコピーが `Activity.Value` プロパティに入れられて、ボットに提供されます。 支払い応答オブジェクトには、顧客が行った最後の選択が、支払いトークンと共に含まれます。 ボットはこの機会を利用して、最初の支払い要求と顧客の最後の選択とに基づき、最後の支払い要求を再計算する必要があります。 顧客の選択が有効であると判定された場合、ボットは支払いトークン ヘッダー内の金額と通貨を確認して、それらが最後の支払い要求に一致することを確認する必要があります。  ボットは、顧客に課金することを決定した場合、顧客が確認した価格に等しい支払いトークン ヘッダー内の金額のみを課金する必要があります。 ボットが想定する値と支払いの完了コールバックで受信した値が一致しない場合、HTTP 状態コード `200 OK` を送信し、結果フィールドを `failure` に設定すれば、支払い要求を失敗させることができます。   

支払い詳細の確認に加えて、ボットは支払い処理を開始する前に、注文を実行できることも確認する必要があります。 たとえば、購入される項目が引き続き在庫にあることを確認することもできます。 値が正しく、支払いプロセッサが正常に支払いトークンを課金した場合、ボットは HTTP 状態コード `200 OK` で応答し、結果フィールドを `success` に設定することによって、支払いの Web エクスペリエンスに支払い確認が表示されるようにする必要があります。 ボットが受信する支払いトークンは、それを要求したマーチャントが 1 回だけ使用でき、Stripe (Bot Framework が現在サポートしている唯一の支払いプロセッサ) に送信される必要があります。 `400` または `500` の範囲にあるいずれかの HTTP 状態コードを送信すると、顧客にとっては一般的なエラーになります。

**支払いボット** サンプルの `OnInvoke` メソッドは、ボットが受信するコールバックを処理します。 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

この例では、ボットは受信アクティビティの `Name` プロパティを調べて、実行する必要のある操作の種類を識別し、適切なメソッドを呼び出してコールバックを処理します。 実装の詳細については、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支払いボット</a> サンプル内の **Controllers/MessagesControllers.cs** を参照してください。

## <a name="testing-a-payment-bot"></a>支払いボットのテスト

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支払いボット</a> サンプルでは、**Web.config** の `LiveMode` 構成設定によって、エミュレートされた支払いトークンと実際の支払いトークンのどちらを支払いの完了コールバックに含めるかが判断されます。 `LiveMode` が `false` に設定されている場合、ボットがテスト モードであることを示すためにボットの支払い送信要求にヘッダーが追加され、[支払いの完了] コールバックに課金できないエミュレートされた支払いトークンが含まれます。 `LiveMode` が `true` に設定されている場合、ボットがテスト モードであることを示すヘッダーはボットの支払い送信要求から省略され、支払いの完了コールバックには、ボットが支払い処理のために Stripe に送信する実際の支払いトークンが含まれます。 これは、指定された支払い方法への課金が発生する実際のトランザクションになります。 

## <a name="additional-resources"></a>その他のリソース

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支払いボット サンプル</a>
- [アクティビティの概要](bot-builder-dotnet-activities.md)
- [メッセージへのリッチ カードの追加](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">W3C での Web 支払い</a> 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>
