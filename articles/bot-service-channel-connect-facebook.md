---
title: Facebook Messenger にボットを接続する | Microsoft Docs
description: Facebook Messenger へのボットの接続を構成する方法について説明します。
keywords: Facebook Messenger, ボット チャンネル, Facebook アプリ, アプリ ID, アプリ シークレット, Facebook ボット, 資格情報
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/12/2018
ms.openlocfilehash: 36d98c6eeb368399ee11ef9a048bb42922103f16
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693614"
---
# <a name="connect-a-bot-to-facebook"></a>Facebook にボットを接続する

ボットは Facebook Messenger と Facebook Workplace の両方に接続できるため、両方のプラットフォームのユーザーと通信できます。 次のチュートリアルでは、これらの 2 つのチャネルにボットを接続する方法を詳しく説明します。

> [!NOTE]
> Facebook の UI は、使用しているバージョンによって多少異なる場合があります。

## <a name="connect-a-bot-to-facebook-messenger"></a>Facebook Messenger にボットを接続する

Facebook Messenger 向けの開発の詳細については、[Messenger プラットフォームのドキュメント](https://developers.facebook.com/docs/messenger-platform)を参照してください。 Facebook の[プレリリースのガイドライン](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[クイック スタート](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)、および[設定ガイド](https://developers.facebook.com/docs/messenger-platform/guides/setup)を参照してください。

Facebook Messenger を使用して通信するようにボットを構成するには、Facebook ページで Facebook Messenger を有効にしてから、ボットをアプリに接続します。

### <a name="copy-the-page-id"></a>ページ ID をコピーする

ボットは Facebook ページからアクセスします。 [新しい Facebook ページを作成する](https://www.facebook.com/bookmarks/pages)か、既存のページに移動します。

* Facebook ページの **[About]\(概要\)** ページを開き、 **[Page ID]\(ページ ID\)** をコピーして保存します。

### <a name="create-a-facebook-app"></a>Facebook アプリを作成する

ページで [[Create a new Facebook App]\(新しい Facebook アプリの作成\)](https://developers.facebook.com/quickstarts/?platform=web) を選択肢、アプリのアプリ ID とアプリ シークレットを生成します。

![アプリ ID を作成する](~/media/channels/FB-CreateAppId.png)

* **[App ID]\(アプリ ID\)** と **[App Secret]\(アプリ シークレット\)** をコピーして保存します。

![アプリ ID とシークレットを保存する](~/media/channels/FB-get-appid.png)

[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\) スライダーを [Yes]\(はい\) に設定します。

![アプリケーション設定](~/media/bot-service-channel-connect-facebook/api_settings.png)

### <a name="enable-messenger"></a>Messenger を有効にする

新しい Facebook App で Facebook Messenger を有効にします。

![Messenger を有効にする](~/media/channels/FB-AddMessaging1.png)

### <a name="generate-a-page-access-token"></a>ページ アクセス トークンを生成する

Messenger セクションの **[Token Generation]\(トークンの生成\)** パネルで、ターゲット ページを選択します。 ページ アクセス トークンが生成されます。

* **[Page Access Token]\(ページ アクセス トークン\)** をコピーして保存します。

![トークンを生成する](~/media/channels/FB-generateToken.png)

### <a name="enable-webhooks"></a>Webhook を有効にする

**[Set up Webhooks]\(Webhook の設定\)** をクリックして、Facebook Messenger からボットにメッセージング イベントを転送します。

![Webhook を有効にする](~/media/channels/FB-webhook.png)

### <a name="provide-webhook-callback-url-and-verify-token"></a>Webhook のコールバック URL を指定し、トークンを確認する

[Azure portal](https://portal.azure.com/) で、ボットを開いて、 **[チャンネル]** タブをクリックし、 **[Facebook Messenger]** をクリックします。

* ポータルの **[コールバックの URL]** と **[トークンの確認]** の値をコピーします。

![値をコピーする](~/media/channels/fb-callbackVerify.png)

1. Facebook メッセンジャーに戻り、 **[コールバックの URL]** と **[トークンの確認]** の値を貼り付けます。

2. **[サブスクリプション フィールド]** で、 *[message\_deliveries]* 、 *[messages]* 、 *[messaging\_optins]* 、 *[messaging\_postbacks]* を選択します。

3. **[Verify and Save]\(確認して保存\)** をクリックします。

![Webhook を構成する](~/media/channels/FB-webhookConfig.png)

4. Webhook を Facebook ページにサブスクライブします。

![Webhook をサブスクライブする](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


### <a name="provide-facebook-credentials"></a>Facebook の資格情報を入力する

Azure portal で、前に Facebook Messenger からコピーした **Facebook アプリ ID**、**Facebook アプリ シークレット**、**ページ ID**、および**ページ アクセス トークン**の各値を貼り付けます。 複数の Facebook ページで同じボットを使用するには、追加のページ ID とアクセス トークンを追加します。

![資格情報を入力する](~/media/channels/fb-credentials2.png)

### <a name="submit-for-review"></a>確認用に送信する

Facebook には、基本アプリ設定ページの [Privacy Policy URL]\(プライバシー ポリシーの URL\) と [ Terms of Service URL]\(利用規約の URL\) が必要です。 [[Code of Conduct]\(行動規範\)](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) ページには、プライバシー ポリシーを作成するためのサード パーティのリソース リンクがあります。 [[Terms of Use]\(利用規約\)](https://www.facebook.com/terms.php) ページには、適切な利用規約を作成するために役立つサンプルの規約が掲載されています。

ボットが完了した後、Facebook には、Messenger に発行されたアプリの独自の[レビュー処理](https://developers.facebook.com/docs/messenger-platform/app-review)があります。 ボットは Facebook の[プラットフォーム ポリシー](https://developers.facebook.com/docs/messenger-platform/policy-overview)に準拠していることを確認するためにテストされます。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する

> [!NOTE]
> アプリが発行されるまでは、[開発モード](https://developers.facebook.com/docs/apps/managing-development-cycle)段階です。 プラグインと API の機能は、管理者、開発者、およびテスターのみが使用できます。

レビューが正常に完了したら、[App Dashboard]\(アプリ ダッシュボード\) の [App Review]\(アプリ レビュー\) でアプリを [Public]\(パブリック\) に設定します。
このボットに関連付けられた Facebook ページが発行されていることを確認します。 [Pages settings]\(ページ設定\) に状態が表示されます。

## <a name="connect-a-bot-to-facebook-workplace"></a>Facebook Workplace にボットを接続する

Facebook Workplace については、[Workplace ヘルプ センター](https://workplace.facebook.com/help/work/)を参照してください。Facebook Workplace の開発に関するガイドラインについては、[Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

Facebook Workplace を使用して通信するようにボットを構成するには、カスタム統合を作成してそれにボットを接続します。

### <a name="create-a-facebook-workplace-premium-account"></a>Facebook Workplace プレミアム アカウントを作成する

[こちら](https://www.facebook.com/workplace)の手順に従って、Facebook Workplace プレミアム アカウントを作成し、自分自身をシステム管理者として設定します。 カスタム統合を作成できるのは Workplace のシステム管理者だけであることに注意してください。

### <a name="create-a-custom-integration"></a>カスタム統合を作成する

カスタム統合を作成すると、アクセス許可が定義されたアプリと、Workplace コミュニティ内でのみ表示される "ボット" 型のページが作成されます。

次の手順に従って、Workplace の[カスタム統合](https://developers.facebook.com/docs/workplace/custom-integrations-new)を作成します。

- **[Admin Panel]\(管理パネル\)** で、 **[Integrations]\(統合\)** タブを開きます。
- **[Create your own custom App]\(カスタム アプリの作成\)** ボタンをクリックします。

![Workplace の統合](~/media/channels/fb-integration.png)

- アプリの表示名とプロファイルの写真を選択します。 これらの情報は、"ボット" 型のページと共有されます。
- **[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\)** を [Yes]\(はい\) に設定します。
- 表示されたアプリ ID、アプリ シークレット、およびアプリ トークンをコピーし、安全に保管します。

![Workplace のキー](~/media/channels/fb-keys.png)

これで、カスタム統合の作成が完了しました。 次に示すように、Workplace コミュニティ内で "ボット" 型のページを見つけることができます。

![Workplace のページ](~/media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Facebook の資格情報を入力する

Azure portal で、前に Facebook Workplace からコピーした **Facebook アプリ ID**、**Facebook アプリ シークレット**、および**ページ アクセス トークン**の各値を貼り付けます。 従来のページ ID ではなく、その **[About]\(詳細\)** ページの統合名の後に続く番号を使用します。 ボットを Facebook Messenger に接続するのと同様に、Webhook には、Azure に示されている資格情報を使用して接続できます。

### <a name="submit-for-review"></a>確認用に送信する
詳細については、「**Facebook Messenger にボットを接続する**」セクションと [Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する
詳細については、「**Facebook Messenger にボットを接続する**」セクションを参照してください。

## <a name="setting-the-api-version"></a>API バージョンを設定する

特定のバージョンの Graph API の廃止に関する Facebook からの通知を受け取った場合、[Facebook 開発者向けページ](https://developers.facebook.com)に移動します。 お使いのボットの **[アプリ設定]** に移動し、 **[設定] > [詳細] > [Upgrade API version]\(API バージョンのアップグレード\)** に移動し、 **[Upgrade All Calls]\(すべての呼び出しをアップグレード\)** を 3.0 に切り替えます。

![API バージョンのアップグレード](~/media/channels/facebook-version-upgrade.png)

## <a name="sample-code"></a>サンプル コード

また、参考として、<a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> のサンプル ボットを使用して、Facebook Messenger とのボット通信を探索することができます。

## <a name="also-available-as-an-adapter"></a>アダプターとしても使用可能

このチャネルは[アダプターとしても使用できます](https://botkit.ai/docs/v4/platforms/facebook.html)。 アダプターとチャネルのどちらを選択するかについては、「[現在使用できるアダプター](bot-service-channel-additional-channels.md#currently-available-adapters)」を参照してください。
