---
title: Facebook Messenger にボットを接続する | Microsoft Docs
description: Facebook Messenger へのボットの接続を構成する方法について説明します。
keywords: Facebook Messenger, ボット チャンネル, Facebook アプリ, アプリ ID, アプリ シークレット, Facebook ボット, 資格情報
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0a9ad7d51234b417d5d0f27dbcffe4ce839ba94a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301206"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>Facebook Messenger にボットを接続する

Facebook Messenger 向けの開発の詳細については、[Messenger プラットフォームのドキュメント](https://developers.facebook.com/docs/messenger-platform)を参照してください。 Facebook の[プレリリースのガイドライン](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[クイック スタート](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)、および[設定ガイド](https://developers.facebook.com/docs/messenger-platform/guides/setup)を参照してください。

Facebook Messenger を使用して通信するようにボットを構成するには、Facebook ページで Facebook Messenger を有効にしてから、ボットをアプリに接続します。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

> [!NOTE]
> Facebook の UI は、使用しているバージョンによって多少異なる場合があります。

## <a name="copy-the-page-id"></a>ページ ID をコピーする

ボットは Facebook ページからアクセスします。 [新しい Facebook ページを作成する](https://www.facebook.com/bookmarks/pages)か、既存のページに移動します。

* Facebook ページの **[About]\(概要\)** ページを開き、**[Page ID]\(ページ ID\)** をコピーして保存します。

## <a name="create-a-facebook-app"></a>Facebook アプリを作成する

ページで [[Create a new Facebook App]\(新しい Facebook アプリの作成\)](https://developers.facebook.com/quickstarts/?platform=web) を選択肢、アプリのアプリ ID とアプリ シークレットを生成します。

![アプリ ID を作成する](~/media/channels/FB-CreateAppId.png)

* **[App ID]\(アプリ ID\)** と **[App Secret]\(アプリ シークレット\)** をコピーして保存します。

![アプリ ID とシークレットを保存する](~/media/channels/FB-get-appid.png)

[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\) スライダーを [Yes]\(はい\) に設定します。

![アプリケーション設定](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>Messenger を有効にする


新しい Facebook App で Facebook Messenger を有効にします。

* アプリの **[Product Setup]\(製品の設定\)** ページで **[Get Started]\(開始\)** をクリックし、もう一度 **[Get Started]\(開始\)** をクリックします。


![Messenger を有効にする](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>ページ アクセス トークンを生成する

Messenger セクションの **[Token Generation]\(トークンの生成\)** パネルで、ターゲット ページを選択します。 ページ アクセス トークンが生成されます。

* **[Page Access Token]\(ページ アクセス トークン\)** をコピーして保存します。

![トークンを生成する](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>Webhook を有効にする

**[Set up Webhooks]\(Webhook の設定\)** をクリックして、Facebook Messenger からボットにメッセージング イベントを転送します。

![Webhook を有効にする](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>Webhook のコールバック URL を指定し、トークンを確認する

[Bot Framework Portal](https://dev.botframework.com/) に戻ります。 ボットを開き、**[チャンネル]** タブをクリックし、**[Facebook Messenger]** をクリックします。

* ポータルの **[コールバックの URL]** と **[トークンの確認]** の値をコピーします。

![値をコピーする](~/media/channels/fb-callbackVerify.png)

1. Facebook メッセンジャーに戻り、**[コールバックの URL]** と **[トークンの確認]** の値を貼り付けます。

2. **[Subscription Fields]\(サブスクリプション フィールド\)** で、*[message\_deliveries]\(メッセージ配信\)*、*[messages]\(メッセージ\)*、*[messaging]\(メッセージング\)\_ オプション*を選択し、*[messaging\_postbacks]\(メッセージングのポストバック\)* を選択します。

3. **[Verify and Save]\(確認して保存\)** をクリックします。

![Webhook を構成する](~/media/channels/FB-webhookConfig.png)

4. Webhook を Facebook ページにサブスクライブします。

![Webhook をサブスクライブする](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>Facebook の資格情報を入力する

Bot Framework Portal で、Facebook Messenger からコピーした**ページ ID**、**アプリ ID**、**アプリ シークレット**、および**ページ アクセス トークン**を貼り付けます。

![資格情報を入力する](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>確認用に送信する

Facebook には、基本アプリ設定ページの [Privacy Policy URL]\(プライバシー ポリシーの URL\) と [ Terms of Service URL]\(利用規約の URL\) が必要です。 [[Code of Conduct]\(行動規範\)](https://aka.ms/bf-conduct) ページには、プライバシー ポリシーを作成するためのサード パーティのリソース リンクがあります。 [[Terms of Use]\(利用規約\)](https://aka.ms/bf-terms) ページには、適切な利用規約を作成するために役立つサンプルの規約が掲載されています。

ボットが完了した後、Facebook には、Messenger に発行されたアプリの独自の[レビュー処理](https://developers.facebook.com/docs/messenger-platform/app-review)があります。 ボットは Facebook の[プラットフォーム ポリシー](https://developers.facebook.com/docs/messenger-platform/policy-overview)に準拠していることを確認するためにテストされます。

## <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する

> [!NOTE]
> アプリが発行されるまでは、[開発モード](https://developers.facebook.com/docs/apps/managing-development-cycle)段階です。 プラグインと API の機能は、管理者、開発者、およびテスターのみが使用できます。

レビューが正常に完了したら、[App Dashboard]\(アプリ ダッシュボード\) の [App Review]\(アプリ レビュー\) でアプリを [Public]\(パブリック\) に設定します。
このボットに関連付けられた Facebook ページが発行されていることを確認します。 [Pages settings]\(ページ設定\) に状態が表示されます。
