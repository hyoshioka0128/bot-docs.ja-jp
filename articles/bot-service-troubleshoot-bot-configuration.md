---
title: ボット構成の問題のトラブルシューティング - Bot Service
description: デプロイされたボットでの構成の問題を解決する方法。
keywords: トラブルシューティング, 構成, Web チャット, 問題。
author: jonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/20/2020
ms.openlocfilehash: 775ffd3a72f69ad721eacbe31b27ddbe5b1fa5e5
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77519971"
---
# <a name="troubleshoot-bot-configuration-issues"></a>ボット構成の問題のトラブルシューティング

ボットでは、さまざまなエラー (応答できない、エラーをスローする、あるチャンネルでは動作しているが他では動作しないなど) が発生する可能性があります。 ボットのトラブルシューティングの最初の手順は、Web チャットでテストすることです。 これにより、自分のボットに固有の問題か (ボットがどのチャネルでも機能しない)、特定のチャネルに固有の問題か (機能するチャネルと機能しないチャネルがある) を判断できます。

## <a name="test-in-web-chat"></a>Web チャットでのテスト

1. [Azure Portal](https://portal.azure.com/) でボット リソースを開きます。
1. **[Test in Web Chat]\(Web チャットでのテスト\)** ウィンドウを開きます。
1. ボットにメッセージを送信します。

![Web チャットでのテスト](./media/test-in-webchat.png)

ボットが想定されている出力で応答しない場合は、「[ボットが Web チャットで機能しない](#bot-does-not-work-in-web-chat)」に進みます。 それ以外の場合は、「[ボットが Web チャットでは機能するが他のチャネルでは機能しない](#bot-works-in-web-chat-but-not-in-other-channels)」に進みます。

## <a name="bot-does-not-work-in-web-chat"></a>ボットが Web チャットで機能しない

ボットが機能しない理由は複数考えられます。 ほとんどは、ボット アプリケーションが停止していてメッセージを受信できない場合や、ボットがメッセージを受信しても応答に失敗する場合です。 考えられる原因のいくつかを次に示します。

- ボットがダウンしていて、到達できない。
- ボットがクラッシュしている。
- ボットのエンドポイントが正しくない。
- ボットは正常にメッセージを受信しているが、応答できない。

ボットが動作しているかどうかを確認するには:

1. **[概要]** ウィンドウを開きます。
1. **[Messaging endpoint]\(メッセージング エンドポイント\)** をコピーし、ブラウザーに貼り付けます。

エンドポイントから HTTP エラー 404 または 405 が返される場合は、ボットが到達可能であり、ボットがメッセージに応答できることを示しています。 タイムアウトの問題を調査する場合は、[タイムアウト](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)または [HTTP 5xx エラーでの失敗](bot-service-troubleshoot-500-errors.md)に関する記事をご覧ください。

エンドポイントから "このサイトに到達できない" または "このページに到達できない" というエラーが返される場合は、ボットが停止しているため、再デプロイする必要があることを示しています。

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>ボットが Web チャットでは機能するが他のチャネルでは機能しない

ボットが Web チャットでは想定どおりに機能し、他のチャネルでは失敗する場合、次の理由が考えられます。

- [ボット構成の問題のトラブルシューティング](#troubleshoot-bot-configuration-issues)
  - [Test in Web Chat](#test-in-web-chat)
  - [ボットが Web チャットで機能しない](#bot-does-not-work-in-web-chat)
  - [ボットが Web チャットでは機能するが他のチャネルでは機能しない](#bot-works-in-web-chat-but-not-in-other-channels)
    - [チャネル構成の問題](#channel-configuration-issues)
    - [チャネル固有の動作](#channel-specific-behavior)
    - [チャネルの停止](#channel-outage)
  - [その他のリソース](#additional-resources)

### <a name="channel-configuration-issues"></a>チャネル構成の問題

ボットのユーザー名とパスワードなどのチャンネル構成パラメーターが正しく設定されていないか、外部から変更された可能性があります。 たとえば、ボットが特定のページの Facebook チャンネルで構成され、そのページが後で削除された場合などです。 最も簡単な解決策は、チャネルを削除し、チャネル構成を新たにやり直すことです。

Bot Framework でサポートされているチャネルの構成手順については、以下のリンクを参照してください。

- [Cortana](bot-service-channel-connect-cortana.md)
- [Direct Line](bot-service-channel-connect-directline.md)
- [Email](bot-service-channel-connect-email.md)
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

一部の機能の実装はチャネルによって異なります。 たとえば、すべてのチャネルでアダプティブ カードがサポートされているわけではありません。 ほとんどのチャンネルでアクション (ボタン) がサポートされますが、チャンネル固有の方法でレンダリングされます。 一部のメッセージの種類が、チャネルによって動作が異なる場合は、[チャネル リファレンス](bot-service-channels-reference.md)記事を調べます。

個々のチャネルに役立つその他のリンクを次に示します。

- [ボットを Microsoft Teams アプリに追加する](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Facebook:Messenger プラットフォームの概要](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Cortana スキル設計の原則](https://docs.microsoft.com/cortana/skills/design-principles)
- [Skype for Developers](https://dev.skype.com/bots)
- [Slack:ボットとの対話を有効にする](https://api.slack.com/bot-users)

### <a name="channel-outage"></a>チャネルの停止

一部のチャネルでサービスの中断が発生することがあります。 通常、このような停止が長時間続くことはありません。 ただし、停止が疑われる場合は、チャンネルの Web サイトまたはソーシャル メディアを調べてください。

チャネルが停止しているかどうかを判断するもう 1 つの方法は、テスト ボット (単純なエコー ボットなど) を作成してチャネルを追加することです。 テスト ボットが動作するチャネルと動作しないチャネルがある場合、問題の原因が運用環境のボットにないことを示します。

## <a name="additional-resources"></a>その他のリソース

[ボットをデバッグする](bot-service-debug-bot.md)方法に関する記事、およびそのセクションに示されているデバッグに関するその他の記事をご覧ください。
