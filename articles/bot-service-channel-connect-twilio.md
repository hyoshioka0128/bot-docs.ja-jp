---
title: ボットを Twilio に接続する | Microsoft Docs
description: ボットの Twilio への接続を構成する方法について説明します。
keywords: Twilio, ボット チャネル, SMS, アプリ, 電話, Twilio の構成, クラウド通信, テキスト
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 817623dd04612cd07d8877c8e9a199c05a2fd9e8
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693621"
---
# <a name="connect-a-bot-to-twilio"></a>ボットを Twilio に接続する

Twilio クラウド通信プラットフォームを使用してユーザーと通信するようにボットを構成できます。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>SMS メッセージを送受信するための Twilio アカウントにログインするかアカウントを作成する

Twilio アカウントを持っていない場合は、<a href="https://www.twilio.com/try-twilio" target="_blank">新しいアカウントを作成します</a>。

## <a name="create-a-twiml-application"></a>TwiML アプリケーションを作成する

手順に従って、<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">TwiML アプリケーションを作成</a>します。

![アプリを作成する](~/media/channels/twi-StepTwiml.png)

**[Properties]\(プロパティ\)** の下で、**フレンドリ名**を入力します。 このチュートリアルでは、例として "My TwiML app" を使用します。 [Voice]\(音声\) の下の **[REQUEST URL]\(要求 URL\)** は空のままにすることができます。 **[Messaging]\(メッセージング\)** の下の **[Request URL]\(要求 URL\)** は `https://sms.botframework.com/api/sms` にする必要があります。

## <a name="select-or-add-a-phone-number"></a>電話番号を選択または追加する

<a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">こちら</a>の手順に従って、コンソール サイト経由で検証済みの呼び出し元の ID を追加します。 終了すると、 **[Manage Numbers]\(番号の管理\)** の下の **[Active Numbers]\(アクティブな番号\)** に検証済みの番号が表示されます。

![電話番号を設定する](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>音声とメッセージングに使用するアプリケーションを指定する

番号をクリックし、 **[Configure]\(構成\)** に移動します。 [Voice]\(音声\) と [Messaging]\(メッセージング\) の両方で、 **[CONFIGURE WITH]\(構成の対象\)** を [TwiML App]\(TwiML アプリ\) に設定し、 **[TWIML APP]\(TwiML アプリ\)** を "My TwiML app" に設定します。 終了したら、 **[Save]\(保存\)** をクリックします。

![アプリケーションを指定する](~/media/channels/twi-StepPhone2.png)

**[Manage Numbers]\(番号の管理\)** に戻ります。[Voice]\(音声\) と [Messaging]\(メッセージング\) の両方の構成が [TwiML App]\(TwiML アプリ\) に変更されています。

![指定された番号](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>資格情報を収集する

[コンソール ホームページ](https://www.twilio.com/console/)に戻ります。次のように、アカウント SID と認証トークンがプロジェクト ダッシュボードに表示されます。

![アプリの資格情報を収集する](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>資格情報を送信する

別のウィンドウで、Bot Framework サイト (https://dev.botframework.com/) に戻ります。 

- **[My bots]\(マイ ボット\)** を選択し、Twilio に接続するボットを選択します。 これにより Azure portal に移動します。
- **[Bot Management]\(ボット管理\)** の下の **[チャネル]** を選択します。 Twilio (SMS) アイコンをクリックします。
- 前に記録した電話番号、アカウント SID、および認証トークンを入力します。 終了したら、 **[保存]** をクリックします。

![資格情報を送信する](~/media/channels/twi-StepSubmit.png)

これらの手順を完了すると、ボットは、Twilio を使ってユーザーと通信するように正しく構成されます。

## <a name="also-available-as-an-adapter"></a>アダプターとしても使用可能

このチャネルも[アダプターとして使用可能な](https://botkit.ai/docs/v4/platforms/twilio-sms.html)します。 アダプターとチャネルを選択するために、次を参照してください。[現在使用可能なアダプター](bot-service-channel-additional-channels.md#currently-available-adapters)します。