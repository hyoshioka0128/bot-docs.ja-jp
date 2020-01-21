---
title: ボットを LINE に接続する - Bot Service
description: ボットから LINE への接続を構成する方法について説明します。
keywords: ボットの接続, ボット チャンネル, LINE ボット, 資格情報, 構成, 携帯電話
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 1/7/2019
ms.openlocfilehash: 092a8327f4a4828642a413201dc0c9483b779345
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791951"
---
# <a name="connect-a-bot-to-line"></a>ボットを LINE に接続する

LINE アプリを介して複数のユーザーと通信できるように、ボットを構成できます。

## <a name="log-into-the-line-console"></a>LINE のコンソールにログインする

*[LINE アカウントでログイン]* を使用して、お使いの LINE アカウントの [LINE Developers Console](https://developers.line.biz/console/register/messaging-api/provider/) にログインします。 

> [!NOTE]
> 登録がまだの場合は、[LINE をダウンロード](https://line.me/)し、設定に移動して電子メール アドレスを登録します。

### <a name="register-as-a-developer"></a>開発者として登録する

初めて LINE Developers Console にアクセスした場合は、名前と電子メール アドレスを入力して、開発者アカウントを作成します。

![LINE のスクリーンショット (開発者の登録)](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>新しいプロバイダーを作成する

まず、ボットのプロバイダーを作成します (まだ設定していない場合)。 プロバイダーは、アプリを提供するエンティティ (個人または会社) です。

![LINE のスクリーンショット (プロバイダーの作成)](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>メッセージング API チャンネルを作成する

次に、新しい Messaging API チャンネルを作成します。 

![LINE のスクリーンショット (チャンネルの種類)](./media/channels/LINE-channel-type-selection.png)

新しい Messaging API チャンネルを作成するには、緑色の四角形をクリックします。

![LINE のスクリーンショット (チャンネルの作成)](./media/channels/LINE-create-channel.png)

名前には、"LINE" またはこれに似た文字列を含めることはできません。 必須フィールドに入力し、チャンネルの設定を確認します。

![LINE のスクリーンショット (チャンネルの設定)](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>チャンネルの設定から必要な値を取得する

チャンネルの設定を確認したら、次のようなページが表示されます。

![LINE のスクリーンショット (チャンネルのページ)](./media/channels/LINE-screenshot-5.png)

作成したチャネルをクリックしてチャネル基本設定にアクセスし、下にスクロールして **[基本情報] > [Channel Secret]** を見つけます。 これを一時的にどこかに保存します。 また、`PUSH_MESSAGE` [利用可能な機能]  が含まれていることを確認します。

![LINE のスクリーンショット (チャンネル シークレット)](./media/channels/LINE-screenshot-6.png)

次に、さらに **[メッセージ送受信設定])** までスクロールします。 **[アクセストークン（ロングターム）]** フィールドと *[再発行]* ボタンがあります。 ボタンをクリックしてアクセストークンを取得し、先ほど同様に一時的に保存します。

![LINE のスクリーンショット (チャンネル トークン)](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>LINE のチャンネルを Azure ボットに接続する

[Azure portal](https://portal.azure.com/) にログインしてボットを見つけ、 **[チャンネル]** をクリックします。 

![LINE のスクリーンショット (Azure の設定)](./media/channels/LINE-channel-setting-2.png)

ここで、LINE のチャンネルを選択し、上記のチャンネル シークレットとアクセス トークンを適切なフィールドに貼り付けます。 必ず変更内容を保存してください。

Azure から提供されるカスタム Webhook URL をコピーします。

![LINE のスクリーンショット (Azure の設定)](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>LINE の Webhook 設定を構成する

次に、LINE Developers Console に戻って、Azure からコピーした Webhook URL を **[メッセージ送受信設定] > [Webhook URL]** に貼り付け、 **[接続確認]** をクリックして接続を確認します。 Azure でチャネルを作成した直後の場合は、有効になるまでに数分かかることがあります。

次に **[メッセージ送受信設定] > [Webhook送信] を [利用する]** にします。

> [!IMPORTANT]
> LINE の Developers Console では、最初に Webhook URL を設定した場合のみ、 **[Use webhooks]\(Webhook の使用\) を[Enabled]\(有効\)** に設定できます。 URL を空にしたままで Webhook を有効にすると、UI では有効になったように見えても、有効な状態に設定されません。

Webhook URL を追加してから Webhook を有効にしたら、このページを再度読み込んで、これらの変更が正しく設定されていることを確認してください。

![LINE のスクリーンショット (Webhook)](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>ボットをテストする

これらの手順を完了すると、ボットは、LINE 上のユーザーと通信できるように正しく構成され、テストの準備が整います。

### <a name="add-your-bot-to-your-line-mobile-app"></a>ボットを LINE のモバイル アプリに追加する

LINE の Developers Console で、[チャネル基本設定] ページの下部にボットの QR コードが表示されています。 

LINE のモバイル アプリで、右端のナビゲーション タブ (3 つのドット **[...]** ) に移動し、QR コードのアイコンをタップします。 

![LINE のスクリーンショット (モバイル アプリ)](./media/channels/LINE-screenshot-12.jpg)

Developers Console の QR コードで QR コード リーダーをポイントします。 これで、LINE のモバイル アプリでボットとやり取りして、ボットをテストできるようになりました。

### <a name="automatic-messages"></a>自動メッセージ

ボットのテストを開始すると、`conversationUpdate` アクティビティで指定したものではない、予期しないメッセージがボットから送信される場合があります。  次のようなダイアログが表示されることがあります。

![LINE のスクリーンショット (会話)](./media/channels/LINE-screenshot-conversation.jpg)

これらのメッセージが送信されないようにするには **[LINE@機能の利用] > [自動応答メッセージ] を [利用しない]** に設定する必要があります。

![LINE のスクリーンショット (自動応答)](./media/channels/LINE-screenshot-10.png)

また、このメッセージを表示したままにすることもできます。 その場合は、[設定はこちら] をクリックして、メッセージを編集することをお勧めします。

![LINE のスクリーンショット (自動応答の設定)](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>トラブルシューティング

* ボットのメッセージに全く応答しない場合は、Azure portal でお使いのボットに移動して **Web チャットでテスト** でテストしてみましょう。  
    * そこでボットが機能しているのに LINE では応答しない場合は、LINE の Developers Console を再度読み込んでから、上記の Webhook の手順を繰り返します。 必ず **Webhook URL** を設定してから Webhook を有効にしてください。
    * ボットで **Web チャットでテスト** がうまく動作しない場合は、ボットの問題をデバッグしてから、再度このページで LINE のチャネルの構成を再設定します。

