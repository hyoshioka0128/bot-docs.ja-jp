---
title: ボット構成の問題のトラブルシューティング | Microsoft Docs
description: デプロイされたボットでの構成の問題を解決する方法。
keywords: トラブルシューティング, 構成, Web チャット, 問題。
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/30/2019
ms.openlocfilehash: c208cef52d1850a00b62828ae0ea622a2606ec5b
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033424"
---
# <a name="troubleshoot-bot-configuration-issues"></a>ボット構成の問題のトラブルシューティング

ボットのトラブルシューティングの最初の手順は、Web チャットでテストすることです。 これにより、自分のボットに固有の問題か (ボットがどのチャネルでも機能しない)、特定のチャネルに固有の問題か (機能するチャネルと機能しないチャネルがある) を判断できます。

## <a name="test-in-web-chat"></a>Web チャットでのテスト

1. [Azure Portal](http://portal.azure.com/) でボット リソースを開きます。
1. **[Test in Web Chat]\(Web チャットでのテスト\)** ウィンドウを開きます。
1. ボットにメッセージを送信します。

![Web チャットでのテスト](./media/test-in-webchat.png)

ボットが想定されている出力で応答しない場合は、「[ボットが Web チャットで機能しない](#bot-does-not-work-in-web-chat)」に進みます。 それ以外の場合は、「[ボットが Web チャットでは機能するが他のチャネルでは機能しない](#bot-works-in-web-chat-but-not-in-other-channels)」に進みます。

## <a name="bot-does-not-work-in-web-chat"></a>ボットが Web チャットで機能しない

ボットが機能しない理由は複数考えられます。 ほとんどは、ボット アプリケーションが停止していてメッセージを受信できない場合や、ボットがメッセージを受信しても応答に失敗する場合です。

ボットが動作しているかどうかを確認するには:

1. **[概要]** ウィンドウを開きます。
1. **[Messaging endpoint]\(メッセージング エンドポイント\)** をコピーし、ブラウザーに貼り付けます。

エンドポイントから HTTP エラー 405 が返される場合は、ボットが到達可能であり、ボットがメッセージに応答できることを示しています。 ボットが[タイムアウト](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)しているか、[HTTP 5xx エラーで失敗](bot-service-troubleshoot-500-errors.md)しているかを調べる必要があります。

エンドポイントから "このサイトに到達できない" または "このページに到達できない" というエラーが返される場合は、ボットが停止しているため、再デプロイする必要があることを示しています。

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>ボットが Web チャットでは機能するが他のチャネルでは機能しない

ボットが Web チャットでは想定どおりに機能し、他のチャネルでは失敗する場合、次の理由が考えられます。

- [チャネル構成の問題](#channel-configuration-issues)
- [チャネル固有の動作](#channel-specific-behavior)
- [チャネルの停止](#channel-outage)

### <a name="channel-configuration-issues"></a>チャネル構成の問題

チャネル構成パラメーターが正しく設定されていないか、外部から変更された可能性があります。 たとえば、ボットで特定のページの Facebook チャネルが構成され、そのページが後で削除された場合があります。 最も簡単な解決策は、チャネルを削除し、チャネル構成を新たにやり直すことです。

Bot Framework でサポートされているチャネルの構成手順については、以下のリンクを参照してください。

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [電子メール](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>チャネル固有の動作

一部の機能の実装はチャネルによって異なります。 たとえば、すべてのチャネルでアダプティブ カードがサポートされているわけではありません。 ほとんどのチャネルはボタンをサポートしていますが、チャネル固有の方法でレンダリングされています。 一部のメッセージの種類が、チャネルによって動作が異なる場合は、[チャネル リファレンス](bot-service-channels-reference.md)を調べます。

個々のチャネルに役立つその他のリンクを次に示します。

- [ボットを Microsoft Teams アプリに追加する](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Facebook:Messenger プラットフォームの概要](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Cortana スキル設計の原則](https://docs.microsoft.com/cortana/skills/design-principles)
- [Skype for Developers](https://dev.skype.com/bots)
- [Slack:ボットとの対話を有効にする](https://api.slack.com/bot-users)

### <a name="channel-outage"></a>チャネルの停止

一部のチャネルでサービスの中断が発生することがあります。 通常、このような停止が長時間続くことはありません。 ただし、停止が疑われる場合は、チャネルの Web サイトまたはソーシャル メディアで確認してください。

チャネルが停止しているかどうかを判断するもう 1 つの方法は、テスト ボット (単純なエコー ボットなど) を作成してチャネルを追加することです。 テスト ボットが動作するチャネルと動作しないチャネルがある場合、問題の原因が運用環境のボットにないことを示します。

## <a name="additional-resources"></a>その他のリソース

[ボットをデバッグする](bot-service-debug-bot.md)方法に関する記事、およびそのセクションに示されているデバッグに関するその他の記事をご覧ください。