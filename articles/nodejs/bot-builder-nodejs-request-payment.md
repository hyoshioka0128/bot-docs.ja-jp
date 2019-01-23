---
title: 支払いを要求する | Microsoft Docs
description: Bot Framework SDK for Node.js を使用して支払い要求を送信する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 296004c654cfd59de6c245bf9702a80024526140
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225747"
---
# <a name="request-payment"></a>支払いの要求

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

ボットでユーザーがアイテムを購入できるようになっている場合、[リッチ カード](bot-builder-nodejs-send-rich-cards.md)に特殊な種類のボタンを含めて、支払いを要求することができます。 この記事では、Bot Framework SDK for Node.js を使用して支払い要求を送信する方法について説明します。

## <a name="prerequisites"></a>前提条件

Bot Framework SDK for Node.js を使用して支払い要求を送信するには、次の前提条件のタスクを完了している必要があります。

### <a name="register-and-configure-your-bot"></a>ボットを登録して構成する

`MicrosoftAppId` と `MicrosoftAppPassword` のボットの環境変数を、[登録](~/bot-service-quickstart-registration.md)プロセス中にボットのために生成されたアプリ ID とパスワードの値に更新します。 

> [!NOTE]
> ボットの **AppID** と **AppPassword** を見つけるには、「[MicrosoftAppID と MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」を参照してください。

### <a name="create-and-configure-merchant-account"></a>マーチャント アカウントを作成して構成する

1. <a href="https://dashboard.stripe.com/register" target="_blank">まだお持ちでない場合は、Stripe アカウントを作成してアクティブにします。</a>

2. <a href="https://seller.microsoft.com/en-us/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Microsoft アカウントを使用して Seller Center にサインインします。</a>

3. Seller Center 内で、そのアカウントを Stripe に接続します。

4. Seller Center 内で、Dashboard に移動し、**MerchantID** の値をコピーします。

5. `PAYMENTS_MERCHANT_ID` 環境変数を Seller Center Dashboard からコピーした値に更新します。 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>支払いボット サンプル

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支払いボット</a> サンプルは、Node.js を使用して支払い要求を送信するボットの例を提供します。 動作中のこのサンプル ボットを確認するには、<a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">それを Web チャットで試し</a>、<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">Skype 連絡先として追加するか</a>、または支払いボット サンプルをダウンロードし、Bot Framework エミュレーターを使用してそれをローカルで実行できます。 

> [!NOTE]
> Web チャットまたは Skype で**支払いボット** サンプルを使用してエンド ツー エンドの支払いプロセスを完了するには、Microsoft アカウント内で有効なクレジット カードまたはデビット カード (つまり、米国のカード発行者からの有効なカード) を指定する必要があります。 **支払いボット** サンプルはテスト モードで実行される (つまり、**.env** で `PAYMENTS_LIVEMODE` が `false` に設定されている) ため、そのカードは課金されず、カードの CVV も検証されません。

この記事の次のいくつかのセクションでは、**支払いボット** サンプルのコンテキストでの支払いプロセスの 3 つの部分について説明します。

## <a id="request-payment"></a> 支払いの要求

ボットは、"支払い" の `type` を指定するボタンが含まれた[リッチ カード](bot-builder-nodejs-send-rich-cards.md)を含むメッセージを送信することによって、ユーザーに支払いを要求できます。 **支払いボット** サンプルからのこのコード スニペットは、ユーザーが支払いプロセスを開始するためにクリック (またはタップ) できる **[購入]** ボタンが含まれた Hero カードを含むメッセージを作成します。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#requestPayment)]

この例では、ボタンの種類は、アプリが "支払い" として定義する `payments.PaymentActionType` として指定されています。 このボタンの値は、サポートされる支払い方法、詳細、およびオプションに関する情報を含む `PaymentRequest` オブジェクトを返す `createPaymentRequest` 関数によって入力されます。 実装の詳細については、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支払いボット</a> サンプル内の **app.js** を参照してください。

このスクリーンショットは、上のコード スニペットによって生成される (**[購入]** ボタンが含まれた) Hero カードを示しています。 
 
![支払いのサンプル ボット](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> **[Buy]** ボタンにアクセスできるユーザーはすべて、そのボタンを使用して支払いプロセスを開始できます。 グループ会話のコンテキスト内では、特定のユーザーだけが使用するボタンを指定することはできません。 

## <a id="user-experience"></a> ユーザー エクスペリエンス

ユーザーは **[購入]** ボタンをクリックすると、その Microsoft アカウントを使用して必要なすべての支払い、配送、および連絡先情報を指定するための支払いの Web エクスペリエンスに移動します。 

![Microsoft での支払い](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP コールバック

ボットが特定の操作を実行すべきであることを示すために、HTTP コールバックがボットに送信されます。 各コールバックは、次のプロパティ値を含むイベントになります。 

| プロパティ | 値 |
|----|----|
| `type` | invoke | 
| `name` | ボットが実行すべき操作の種類 (配送先住所の更新、配送オプションの更新、支払いの完了など) を示します。 | 
| `value` | JSON 形式の要求ペイロード。 | 
| `relatesTo` |  支払い要求に関連付けられているチャネルとユーザーを説明します。 | 

> [!NOTE]
> `invoke` は、Microsoft Bot Framework で使用するために予約されている特殊なイベントの種類です。 `invoke` イベントの送信者は、ボットが HTTP 応答を送信することによってコールバックを確認することを想定します。

## <a id="process-callbacks"></a> コールバックの処理

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>[配送先住所の更新] および [配送オプションの更新] コールバック

[配送先住所の更新] または [配送オプションの更新] コールバックの受信時、ボットには、クライアントからの支払い詳細の現在の状態がイベントの `value` プロパティで提供されます。
マーチャントは、これらのコールバックを静的として処理する必要があります。入力の支払い詳細が与えられ、何らかの出力の支払い詳細を計算しますが、クライアントによって提供された入力状態が何らかの理由で無効である場合は失敗します。 
ボットは、提供された情報をそのままで有効であると判定した場合は、単純に HTTP 状態コード `200 OK` を変更されていない支払い詳細と共に送信します。 ボットが HTTP 状態コード `200 OK` を、その注文を処理する前に適用する必要がある更新された支払い詳細と共に送信する場合があります。 場合によっては、ボットが更新された情報を無効であると判定することがあり、そのままでは注文を処理できません。 たとえば、ユーザーの配送先住所として、製品のサプライヤーが配送先にしていない国が指定されている場合があります。 その場合、ボットは HTTP 状態コード `200 OK` と、支払い詳細オブジェクトのエラー プロパティを含むメッセージを送信できます。 `400` または `500` の範囲にあるいずれかの HTTP 状態コードを送信すると、顧客にとっては一般的なエラーになります。

### <a name="payment-complete-callbacks"></a>[支払いの完了] コールバック

[支払いの完了] コールバックの受信時、ボットには、最初の変更されていない支払い要求および支払い応答オブジェクトのコピーがイベントの `value` プロパティで提供されます。 支払い応答オブジェクトには、顧客が行った最後の選択が、支払いトークンと共に含まれます。 ボットはこの機会を利用して、最初の支払い要求と顧客の最後の選択とに基づき、最後の支払い要求を再計算する必要があります。 顧客の選択が有効であると判定された場合、ボットは支払いトークン ヘッダー内の金額と通貨を確認して、それらが最後の支払い要求に一致することを確認する必要があります。  ボットは、顧客に課金することを決定した場合、顧客が確認した価格に等しい支払いトークン ヘッダー内の金額のみを課金する必要があります。 ボットが想定する値と支払いの完了コールバックで受信した値が一致しない場合、HTTP 状態コード `200 OK` を送信し、結果フィールドを `failure` に設定すれば、支払い要求を失敗させることができます。   

支払い詳細の確認に加えて、ボットは支払い処理を開始する前に、注文を実行できることも確認する必要があります。 たとえば、購入される項目が引き続き在庫にあることを確認することもできます。 値が正しく、支払いプロセッサが正常に支払いトークンを課金した場合、ボットは HTTP 状態コード `200 OK` で応答し、結果フィールドを `success` に設定することによって、支払いの Web エクスペリエンスに支払い確認が表示されるようにする必要があります。 ボットが受信する支払いトークンは、それを要求したマーチャントが 1 回だけ使用でき、Stripe (Bot Framework が現在サポートしている唯一の支払いプロセッサ) に送信される必要があります。 `400` または `500` の範囲にあるいずれかの HTTP 状態コードを送信すると、顧客にとっては一般的なエラーになります。

**支払いボット** サンプルの次のコード スニペットは、ボットが受信するコールバックを処理します。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#processCallback)]

この例では、ボットは受信イベントの `name` プロパティを検証して、実行する必要のある操作の種類を識別してから、適切なメソッドを呼び出してコールバックを処理します。 実装の詳細については、<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支払いボット</a> サンプル内の **app.js** を参照してください。

## <a name="testing-a-payment-bot"></a>支払いボットのテスト

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支払いボット</a> サンプルでは、**.env** 内の `PAYMENTS_LIVEMODE` 環境変数によって、[支払いの完了] コールバックにエミュレートされた支払いトークンまたは実際の支払いトークンのどちらが含まれるかが決定されます。 `PAYMENTS_LIVEMODE` が `false` に設定されている場合は、ボットがテスト モードにあることを示すヘッダーがボットの送信支払い要求に追加され、[支払いの完了] コールバックには、課金できないエミュレートされた支払いトークンが含まれます。 `PAYMENTS_LIVEMODE` が `true` に設定されている場合、ボットがテスト モードであることを示すヘッダーはボットの支払い送信要求から省略され、支払いの完了コールバックには、ボットが支払い処理のために Stripe に送信する実際の支払いトークンが含まれます。 これは、指定された支払い方法への課金が発生する実際のトランザクションになります。 

## <a name="additional-resources"></a>その他のリソース

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支払いボット サンプル</a>
- [メッセージにリッチ カード添付ファイルを追加する](bot-builder-nodejs-send-rich-cards.md)
- <a href="http://www.w3.org/Payments/" target="_blank">W3C での Web 支払い</a> 
