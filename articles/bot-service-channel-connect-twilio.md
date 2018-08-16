---
title: ボットを Twilio に接続する | Microsoft Docs
description: ボットの Twilio への接続を構成する方法について説明します。
keywords: Twilio, ボット チャネル, SMS, アプリ, 電話, Twilio の構成, クラウド通信, テキスト
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301216"
---
# <a name="connect-a-bot-to-twilio"></a>ボットを Twilio に接続する

Twilio クラウド通信プラットフォームを使用してユーザーと通信するようにボットを構成できます。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>SMS メッセージを送受信するための Twilio アカウントにログインするかアカウントを作成する

Twilio アカウントを持っていない場合は、<a href="https://www.twilio.com/try-twilio" target="_blank">新しいアカウントを作成します</a>

## <a name="create-a-twiml-application"></a>TwiML アプリケーションを作成する

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">TwiML アプリケーションを作成します</a>

![アプリを作成する](~/media/channels/twi-StepTwiml.png)

 [Messaging]\(メッセージング\) の [Request URL]\(要求 URL\) は **https://sms.botframework.com/api/sms** にする必要があります。

## <a name="select-or-add-a-phone-number"></a>電話番号を選択または追加する

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">電話番号を選択または追加します</a>。 番号をクリックして、作成した TwiML アプリケーションに追加します。

![電話番号を設定する](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>メッセージングに使用するアプリケーションを指定する
**[Messaging]\(メッセージング\)** セクションで、**[TwiML App]\(TwiML アプリ\)** を作成した TwiML アプリの名前に設定します。
後で使うので、**[Phone Number]\(電話番号\)** の値をコピーします。

![アプリを指定する](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>資格情報を収集する

<a href="https://www.twilio.com/user/account/settings" target="_blank">資格情報を収集</a>し、"目" のアイコンをクリックして認証トークンを表示します。

![アプリの資格情報を収集する](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>資格情報を送信する

電話番号、accountSID、および先ほどコピーした認証トークンを入力し、**[Submit Twilio Credentials]\(Twilio 資格情報の送信\)** をクリックします。

## <a name="enable-the-bot"></a>ボットを有効にする
**[Enable this bot on SMS]\(このボットを SMS で有効にする\)** をオンにします。 次に、**[I'm done configuring SMS]\(SMS の構成が終了しました\)** をクリックします。

これらの手順を完了すると、ボットは、Twilio を使ってユーザーと通信するように正しく構成されます。

