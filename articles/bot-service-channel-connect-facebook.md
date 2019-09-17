---
title: Facebook Messenger にボットを接続する | Microsoft Docs
description: Facebook Messenger へのボットの接続を構成する方法について説明します。
keywords: Facebook Messenger, ボット チャンネル, Facebook アプリ, アプリ ID, アプリ シークレット, Facebook ボット, 資格情報
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 08/03/2019
ms.openlocfilehash: a856e3cc578b8c73583126df9f670bfde68ec9dc
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386106"
---
# <a name="connect-a-bot-to-facebook"></a>Facebook にボットを接続する

ボットは Facebook Messenger と Facebook Workplace の両方に接続できるため、両方のプラットフォームのユーザーと通信できます。 次のチュートリアルでは、これらの 2 つのチャネルにボットを接続する方法を説明します。

> [!NOTE]
> Facebook の UI は、使用しているバージョンによって多少異なる場合があります。

## <a name="connect-a-bot-to-facebook-messenger"></a>Facebook Messenger にボットを接続する

Facebook Messenger 向けの開発の詳細については、[Messenger プラットフォームのドキュメント](https://developers.facebook.com/docs/messenger-platform)を参照してください。 Facebook の[プレリリースのガイドライン](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[クイック スタート](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)、および[設定ガイド](https://developers.facebook.com/docs/messenger-platform/guides/setup)を参照してください。

Facebook Messenger を使用して通信するようにボットを構成するには、Facebook ページで Facebook Messenger を有効にしてから、ボットを接続します。

### <a name="copy-the-page-id"></a>ページ ID をコピーする

ボットは Facebook ページからアクセスします。

1. [新しい Facebook ページを作成する](https://www.facebook.com/bookmarks/pages)か、既存のページに移動します。

1. Facebook ページの **[About]\(概要\)** ページを開き、 **[Page ID]\(ページ ID\)** をコピーして保存します。

### <a name="create-a-facebook-app"></a>Facebook アプリを作成する

1. ブラウザーで、[[Create a new Facebook App]\(新しい Facebook アプリの作成\)](https://developers.facebook.com/quickstarts/?platform=web) に移動します。
1. アプリの名前を入力し、 **[Create New Facebook App ID]\(新しい Facebook アプリ ID の作成\)** ボタンをクリックします。

    ![アプリを作成する](media/channels/fb-create-messenger-bot-app.png)

1. 表示されたダイアログ ボックスで、電子メール アドレスを入力し、 **[Create App ID]\(アプリ ID の作成\)** ボタンをクリックします。

    ![アプリ ID を作成する](media/channels/fb-create-messenger-bot-app-id.png)

1. ウィザードの手順を実行します。

1. 必要なチェック情報を入力し、右上にある **[Skip Quick Start]\(クイック スタートをスキップ\)** ボタンをクリックします。

1. 次に表示されるウィンドウの左側のウィンドウで、 *[設定]* を展開し、 **[基本]** をクリックします。

1. 右側のウィンドウで、 **[アプリ ID]** および **[アプリ シークレット]** をコピーして保存します。

    ![アプリ ID とアプリ シークレットをコピー](media/channels/fb-messenger-bot-get-appid-secret.png)

1. 左側のウィンドウの *[設定]* で、 **[詳細]** をクリックします。

1. 右側のウィンドウで、 **[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\)** スライダーを **[はい]** に設定します。

    ![アプリ ID とアプリ シークレットをコピー](media/channels/fb-messenger-bot-api-settings.png)

1. 右下のページで、 **[変更の保存]** ボタンをクリックします。

### <a name="enable-messenger"></a>Messenger を有効にする

1. 左側のウィンドウで、 **[ダッシュボード]** をクリックします。
1. 右側のウィンドウで下にスクロールし、 **[Messenger]** ボックスの **[Set Up]\(設定\)** ボタンをクリックします。 Messenger のエントリは、左側のウィンドウの *[PRODUCTS]\(製品\)* セクションに表示されます。  

    ![Messenger を有効にする](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>ページ アクセス トークンを生成する

1. 左側のウィンドウで、Messenger のエントリの下にある **[設定]** をクリックします。
1. 右側のウィンドウで下にスクロールし、 **[Token Generation]\(トークンの生成\)** セクションでターゲット ページを選択します。

    ![Messenger を有効にする](media/channels/fb-messenger-bot-select-messenger-page.png)

1. アクセス トークンを生成するために、 **[アクセス許可の編集]** ボタンをクリックして、pages_messaging をアプリに許可します。
1. ウィザードの手順に従います。 最後の手順では、既定の設定をそのまま使用し、 **[完了]** ボタンをクリックします。 最後に、**ページ アクセス トークン**が生成されます。

    ![Messenger のアクセス許可](media/channels/fb-messenger-bot-permissions.png)

1. **[Page Access Token]\(ページ アクセス トークン\)** をコピーして保存します。

### <a name="enable-webhooks"></a>Webhook を有効にする

ボットから Facebook Messenger にメッセージやその他のイベントを送信するには、Webhook 統合を有効にする必要があります。 この時点では、Facebook 設定の手順は保留のままにしておき、後で戻ってきます。

1. ブラウザーで新しいウィンドウを開き、[Azure portal](https://portal.azure.com/) に移動します。 

1. リソースの一覧でボット リソースの登録をクリックし、関連するブレードで **[チャネル]** をクリックします。

1. 右側のウィンドウで、 **[Facebook]** アイコンをクリックします。

1. ウィザードで、前の手順で保存した Facebook の情報を入力します。 情報が正しい場合、ウィザードの下部に**コールバック URL** と**トークンの確認**が表示されます。 それらをコピーして保存します。  

    ![FB Messenger チャネル構成](media/channels/fb-messenger-bot-config-channel.PNG)

1. **[保存]** ボタンをクリックします。


1. Facebook の設定に戻りましょう。 右側のウィンドウで下にスクロールし、 **[Webhook]** セクションで **[Subscribe To Events]\(イベントのサブスクライブ\)** ボタンをクリックします。 これは、メッセージング イベントを Facebook Messenger からボットに転送するためです。

    ![Webhook を有効にする](media/channels/fb-messenger-bot-webhooks.PNG)

1. 表示されたダイアログ ボックスで、前に保存した **[コールバック URL]** および **[トークンの確認]** の値を入力します。 **[サブスクリプション フィールド]** で、 *[message\_deliveries]* 、 *[messages]* 、 *[messaging\_optins]* 、 *[messaging\_postbacks]* を選択します。

    ![Webhook を構成する](media/channels/fb-messenger-bot-config-webhooks.png)

1. **[確認して保存]** ボタンをクリックします。
1. Webhook をサブスクライブする Facebook ページを選択します。 **[サブスクライブ]** ボタンをクリックします。

    ![Webhook ページを構成する](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>確認用に送信する

Facebook には、基本アプリ設定ページの [Privacy Policy URL]\(プライバシー ポリシーの URL\) と [ Terms of Service URL]\(利用規約の URL\) が必要です。 [[Code of Conduct]\(行動規範\)](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) ページには、プライバシー ポリシーを作成するためのサード パーティのリソース リンクがあります。 [[Terms of Use]\(利用規約\)](https://www.facebook.com/terms.php) ページには、適切な利用規約を作成するために役立つサンプルの規約が掲載されています。

ボットが完了した後、Facebook には、Messenger に発行されたアプリの独自の[レビュー処理](https://developers.facebook.com/docs/messenger-platform/app-review)があります。 ボットは Facebook の[プラットフォーム ポリシー](https://developers.facebook.com/docs/messenger-platform/policy-overview)に準拠していることを確認するためにテストされます。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する

> [!NOTE]
> アプリが発行されるまでは、[開発モード](https://developers.facebook.com/docs/apps/managing-development-cycle)段階です。 プラグインと API の機能は、管理者、開発者、およびテスターのみが使用できます。

レビューが正常に完了したら、[App Dashboard]\(アプリ ダッシュボード\) の [App Review]\(アプリ レビュー\) でアプリを [Public]\(パブリック\) に設定します。
このボットに関連付けられた Facebook ページが発行されていることを確認します。 [Pages settings]\(ページ設定\) に状態が表示されます。

## <a name="connect-a-bot-to-facebook-workplace"></a>Facebook Workplace にボットを接続する

Facebook Workplace は、従業員が簡単に接続して共同作業できる Facebook のビジネス指向バージョンです。 これには、ライブ動画、ニュース フィード、グループ、Messenger、リアクション、検索、およびトレンドの投稿が含まれています。 また、以下もサポートします。

- 分析と統合。 企業が Workplace を既存の IT システムと統合するために使用する分析、統合、シングルサインオン、および ID プロバイダーを備えたダッシュボード。
- 複数の企業のグループ。 さまざまな組織の従業員が連携して共同作業を行うことができる共有スペース。

Facebook Workplace については、[Workplace ヘルプ センター](https://workplace.facebook.com/help/work/)を参照してください。Facebook Workplace の開発に関するガイドラインについては、[Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

ボットと共に Facebook Workplace を使用するには、ボットを接続するために、Workplace アカウントとカスタム統合を作成する必要があります。

### <a name="create-a-workplace-premium-account"></a>Workplace プレミアム アカウントを作成する

1. 会社を代表して [Workplace](https://www.facebook.com/workplace) に申請を送信します。
1. 申請が承認されると、参加するよう招待するメールが届きます。 応答にはしばらく時間がかかる場合があります。
1. 招待メールの **[Get Started]\(作業の開始\)** をクリックします。
1. プロファイル情報を入力します。
    > [!TIP]
    > 自分自身をシステム管理者として設定します。 カスタム統合を作成できるのはシステム管理者のみであることに注意してください。
1. **[Preview Profile]\(プロファイルのプレビュー\)** をクリックし、情報が正しいことを確認します。
1. *[Free Trial]\(無料試用版\)* にアクセスします。
1. **パスワード**を作成します。
1. **[Invite Coworkers]\(同僚を招待する\)** をクリックして、サインインするように従業員を招待します。 招待した従業員は、サインインするとすぐにメンバーになります。 彼らは、これらの手順で説明した内容と同様のサインイン プロセスを行います。

### <a name="create-a-custom-integration"></a>カスタム統合を作成する

以下で説明する手順に従って、Workplace の[カスタム統合](https://developers.facebook.com/docs/workplace/custom-integrations-new)を作成します。 カスタム統合を作成すると、アクセス許可が定義されたアプリと、Workplace コミュニティ内でのみ表示される "ボット" 型のページが作成されます。

1. **[Admin Panel]\(管理パネル\)** で、 **[Integrations]\(統合\)** タブを開きます。
1. **[Create your own custom App]\(カスタム アプリの作成\)** ボタンをクリックします。

    ![Workplace の統合](media/channels/fb-integration.png)

1. アプリの表示名とプロファイルの写真を選択します。 これらの情報は、"ボット" 型のページと共有されます。
1. **[Allow API Access to App Settings]\(アプリ設定への API のアクセスを許可する\)** を [Yes]\(はい\) に設定します。
1. 表示されたアプリ ID、アプリ シークレット、およびアプリ トークンをコピーし、安全に保管します。

    ![Workplace のキー](media/channels/fb-keys.png)

1. これで、カスタム統合の作成が完了しました。 次に示すように、Workplace コミュニティ内で "ボット" 型のページを見つけることができます。

    ![Workplace のページ](media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Facebook の資格情報を入力する

Azure portal で、前に Facebook Workplace からコピーした **Facebook アプリ ID**、**Facebook アプリ シークレット**、および**ページ アクセス トークン**の各値を貼り付けます。 従来のページ ID ではなく、その **[About]\(詳細\)** ページの統合名の後に続く番号を使用します。 ボットを Facebook Messenger に接続するのと同様に、Webhook には、Azure に示されている資格情報を使用して接続できます。

### <a name="submit-for-review"></a>確認用に送信する
詳細については、「**Facebook Messenger にボットを接続する**」セクションと [Workplace の開発者向けドキュメント](https://developers.facebook.com/docs/workplace)を参照してください。

### <a name="make-the-app-public-and-publish-the-page"></a>アプリを発行してページを発行する
詳細については、「**Facebook Messenger にボットを接続する**」セクションを参照してください。

## <a name="setting-the-api-version"></a>API バージョンを設定する

特定のバージョンの Graph API の廃止に関する Facebook からの通知を受け取った場合、[Facebook 開発者向けページ](https://developers.facebook.com)に移動します。 お使いのボットの **[アプリ設定]** に移動し、 **[設定] > [詳細] > [Upgrade API version]\(API バージョンのアップグレード\)** に移動し、 **[Upgrade All Calls]\(すべての呼び出しをアップグレード\)** を 3.0 に切り替えます。

![API バージョンのアップグレード](media/channels/fb-version-upgrade.png)

## <a name="see-also"></a>関連項目

- **サンプル コード**。 <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> のサンプル ボットを使用して、Facebook Messenger とのボット通信を探索します。

- **アダプターとして使用可能**。 このチャネルは[アダプターとしても使用できます](https://botkit.ai/docs/v4/platforms/facebook.html)。 アダプターとチャネルのどちらを選択するかについては、「[現在使用できるアダプター](bot-service-channel-additional-channels.md#currently-available-adapters)」を参照してください。
