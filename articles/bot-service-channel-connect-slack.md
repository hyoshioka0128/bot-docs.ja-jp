---
title: ボットを Slack に接続する | Microsoft Docs
description: ボットの Slack への接続を構成する方法について説明します。
keywords: ボットの接続, ボット チャネル, Slack ボット, Slack メッセージング アプリ
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: e2176d3eb5584a1d9a234d4ab94c69451f0db6ef
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315228"
---
# <a name="connect-a-bot-to-slack"></a>ボットを Slack に接続する

Slack メッセージング アプリを使用して、複数のユーザーとコミュニケーションするようにボットを構成できます。

## <a name="create-a-slack-application-for-your-bot"></a>ボット用の Slack アプリケーションを作成する

[Slack](https://slack.com/signin) にログインし、[[create a Slack application]\(Slack アプリケーションの作成\)](https://api.slack.com/apps) チャネルに移動します。

![ボットを設定する](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>アプリを作成して開発 Slack チームを割り当てる

アプリ名を入力し、開発 Slack チームを選択します。 まだ開発 Slack チームのメンバーになっていない場合は、[作成または参加](https://slack.com/)します。

![アプリを作成する](~/media/channels/slack-CreateApp.png)

**[Create app (アプリの作成)]** をクリックします。 Slack でアプリが作成されて、クライアント ID とクライアント シークレットが生成されます。

## <a name="add-a-new-redirect-url"></a>新しいリダイレクト URL を追加する

次に、新しいリダイレクト URL を追加します。

1. **[OAuth & Permissions]\(OAuth とアクセス許可\)** タブを選択します。
2. **[Add a new Redirect URL]\(新しいリダイレクト URL の追加\)** をクリックします。
3. 「 https://slack.botframework.com 」を入力します。
4. **[追加]** をクリックします。
5. **[Save URLs]\(URL の保存\)** をクリックします。

![リダイレクト URL を追加する](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Slack ボット ユーザーを作成する

ボット ユーザーを追加すると、ボットにユーザー名を割り当てて、ボットを常にオンラインとして表示するかどうかを選択できます。

1. **[Bot Users]\(ボット ユーザー\)** タブを選択します。
2. **[Add a Bot User]\(ボット ユーザーの追加\)** をクリックします。

![ボットを作成する](~/media/channels/slack-CreateBot.png)

**[Add Bot User]\(ボット ユーザーの追加\)** をクリックして設定を検証し、**[Always Show My Bot as Online]\(ボットを常にオンラインとして表示する\)** をクリックして **[On]\(オン\)** に設定した後、**[Save Changes]\(変更の保存\)** をクリックします。

![ボットを作成する](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>ボットのイベントをサブスクライブする

以下の手順に従って、ボットの 6 つの特定のイベントをサブスクライブします。 ボットのイベントをサブスクライブすると、指定した URL でのユーザー アクティビティがアプリに通知されます。

> [!TIP]
> ボット ハンドルはボットの名前です。 ボットのハンドルを調べるには、[https://dev.botframework.com/bots](https://dev.botframework.com/bots) にアクセスして、ボットを選択し、ボットの名前を記録します。

1. **[Event Subscriptions]\(イベント サブスクリプション\)** タブを選択します。
2. **[Enable Events]\(イベントを有効にする\)** をクリックして **[On]\(オン\)** にします。
3. **[Request URL]\(要求 URL\)** に次の URL を入力します。ただし、`{YourBotHandle}` は実際のボット ハンドルに置き換えます。 このチュートリアルで使用するボット ハンドルは、testChannels です。
        `https://slack.botframework.com/api/Events/{YourBotHandle}`
4. **[Subscribe to Workspace Events]\(ワークスペースのイベントをサブスクライブ\)** で、**[Add Workspace Event]\(ワークスペース イベントの追加\)** をクリックします。
5. イベントの一覧で、次の 6 つのイベント タイプを選択します。
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

![イベントをサブスクライブする](~/media/channels/slack-SubscribeEvents.png)
6. **[変更を保存]** をクリックします。

## <a name="add-and-configure-interactive-messages-optional"></a>対話型メッセージを追加して構成する (省略可能)

ボットがボタンなどの Slack 固有の機能を使う場合は、次の手順のようにします。

1. **[Interactive Components]\(対話型コンポーネント\)** タブを選択し、**[Enable Interactive Components]\(対話型コンポーネントを有効にする\)** をクリックします。
2. **[Request URL]\(要求 URL\)** として「`https://slack.botframework.com/api/Actions`」と入力します。
3. **[Save changes]\(変更の保存\)** ボタンをクリックします。

![メッセージを有効にする](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>資格情報を収集する

**[Basic Information]\(基本情報\)** タブを選択し、**[App Credentials]\(アプリの資格情報\)** セクションまでスクロールします。
Slack ボットの構成に必要なクライアント ID、クライアント シークレット、検証トークンが表示されます。

![資格情報を収集する](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>資格情報を送信する

別のブラウザー ウィンドウで、Bot Framework サイト (`https://dev.botframework.com/`) に戻ります。

1. **[My bots]\(マイ ボット\)** を選択し、Slack に接続するボットを選択します。
2. **[チャネル]** セクションで、Slack アイコンをクリックします。
3. **[Enter your Slack credentials]\(Slack 資格情報の入力\)** セクションで、Slack の Web サイトからコピーしたアプリの資格情報を適切なフィールドに貼り付けます。
4. **[Landing Page URL]\(ランディング ページの URL\)** は省略可能です。 省略しても変更してもかまいません。
5. **[Save]** をクリックします。

![資格情報を送信する](~/media/channels/slack-SubmitCredentials.png)

指示に従って、開発 Slack チームへの Slack アプリのアクセスを承認します。

## <a name="enable-the-bot"></a>ボットを有効にする

[Configure Slack]\(Slack の構成\) ページで、[Save]\(保存\) ボタンのスライダーが **[Enabled]\(有効\)** に設定されていることを確認します。
ボットは、Slack のユーザーと通信するように構成されています。

## <a name="create-an-add-to-slack-button"></a>Slack に追加ボタンを作成する

[このページ](https://api.slack.com/docs/slack-button)の「*Add the Slack button*」(Slack ボタンを追加する) セクションでは、Slack ユーザーがボットを検索するときに使用できる HTML が提供されています。
ボットでこの HTML を使うには、(`https://` で始まる) href の値を、ボットの Slack チャネルの設定で示されている URL に置き換えます。
置き換える URL を取得するには、次の手順のようにします。

1. [https://dev.botframework.com/bots](https://dev.botframework.com/bots) で、対象のボットをクリックします。
2. **[チャネル]** をクリックし、**[Slack]** というエントリを右クリックして、**[Copy link]\(リンクのコピー\)** をクリックします。 この URL がクリップボードにコピーされます。
3. この URL をクリップボードから Slack ボタンの HTML に貼り付けます。 この URL で、このボットに対して Slack によって提供される href の値を置き換えます。

承認されたユーザーは、この変更した HTML で提供される **[Add to Slack]\(Slack に追加\)** ボタンをクリックして、Slack からボットにアクセスできます。
