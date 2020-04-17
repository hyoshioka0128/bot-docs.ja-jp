---
title: ボットを Office 365 の電子メールに接続する - Bot Service
description: Office 365 で電子メールを送受信するようにボットを構成する方法について説明します。
keywords: Office 365, ボット チャネル, 電子メール, 電子メールの資格情報, Azure Portal, カスタム電子メール
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/15/2019
ms.openlocfilehash: 8ca2735dac838982af3ba97221e07d80155153b7
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75792093"
---
# <a name="connect-a-bot-to-office-365-email"></a>ボットを Office 365 の電子メールに接続する

ボットは、他の[チャネル](~/bot-service-manage-channels.md)に加え、Office 365 の電子メールを使用してユーザーと通信できます。 ボットは、電子メール アカウントにアクセスするように構成されている場合は、新しい電子メールが届いたときにメッセージを受信します。 その後、ボットは、ビジネス ロジックが指示するとおりに応答できます。 たとえば、ボットは、電子メールが受信されたことを確認する、次のような応答メッセージを送信できます。"ご注文 ありがとうございます。 今すぐ処理を開始いたします。"

> [!WARNING]
> 望まれていない電子メールや承認されていない電子メールを一括送信するボットなどの "スパムボット" の作成は、Bot Framework の[倫理規定](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm)違反です。

> [!NOTE]
> Microsoft Exchange Server を使用している場合は、電子メール チャネルを構成する前に、[自動検出](https://docs.microsoft.com/exchange/client-developer/exchange-web-services/autodiscover-for-exchange)を最初に有効にしていることを確認してください。 

## <a name="configure-email-credentials"></a>電子メールの資格情報を構成する

Office 365 の資格情報を電子メール チャネル構成に入力することで、電子メール チャネルにボットを接続できます。
AAD に代わる任意のベンダーを使用するフェデレーション認証はサポートされていません。

> [!NOTE]
> 個人の電子メール アカウントをボットで使用しないでください。これを行うと、その電子メール アドレスに送信されたすべてのメッセージがボットに転送されます。 その結果、ボットが不適切な応答を送信元に送信する可能性があります。 このため、ボットでは、専用の O365 メール アカウントを使用する必要があります。

電子メール チャンネルを追加するには、[Azure Portal](https://portal.azure.com/) でボットを開き、 **[チャンネル]** ブレードをクリックし、 **[電子メール]** をクリックします。 有効な電子メールの資格情報を入力し、 **[保存]** をクリックします。

![電子メールの資格情報の入力](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

電子メール チャネルは、現時点では Office 365 でのみ動作します。 その他の電子メール サービスは、現時点ではサポートされていません。

## <a name="customize-emails"></a>電子メールをカスタマイズする

電子メール チャネルは、`channelData` プロパティを使用したより高度なカスタマイズされた電子メールを作成するためのカスタム プロパティの送信をサポートします。

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

次のメッセージの例は、これらの `channelData` プロパティを含む JSON ファイルを示しています。

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
`channelData` の使用方法の詳細については、[Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) のサンプルまたは [.NET](~/dotnet/bot-builder-dotnet-channeldata.md) のドキュメントをご覧ください。
::: moniker-end

::: moniker range="azure-bot-service-4.0"
`channelData` を使用する方法について詳しくは、[チャネル固有の機能を実装する方法](~/v4sdk/bot-builder-channeldata.md)に関する記事をご覧ください。
::: moniker-end

## <a name="other-considerations"></a>その他の考慮事項

お客様のボットが、受信電子メール メッセージへの応答として 15 秒内に 200 OK の HTTP 状態コードを返さない場合、電子メール チャネルはメッセージを再送信しようとするため、お客様のボットは同じ電子メール メッセージ アクティビティを数回受け取る可能性があります。 詳細については、「**How bots work (ボットのしくみ)** 」の「[HTTP details (HTTP の詳細)](v4sdk/bot-builder-basics.md#http-details)」セクション、および[タイムアウト エラーのトラブルシューティング](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)の方法に関する記事を参照してください。

> [!NOTE]
> ご使用の Office 365 アカウントで MFA が有効になっている場合は、まず所定のアカウントの MFA を無効にしてから、アカウントのメール チャネルの構成を行ってください。 そうでない場合、接続に失敗します。 

## <a name="additional-resources"></a>その他のリソース

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* ボットを[チャネル](~/bot-service-manage-channels.md)に接続する
* Bot Framework SDK for .NET を使用して[チャネル固有の機能を実装する](dotnet/bot-builder-dotnet-channeldata.md)
* [チャネルのリファレンス](bot-service-channels-reference.md)記事を読み、各チャネルでサポートされる機能についての詳しい情報を入手する
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* ボットを[チャネル](~/bot-service-manage-channels.md)に接続する
* Bot Framework SDK for .NET を使用して[チャネル固有の機能を実装する](~/v4sdk/bot-builder-channeldata.md)
* [チャネルのリファレンス](bot-service-channels-reference.md)記事を読み、各チャネルでサポートされる機能についての詳しい情報を入手する
::: moniker-end

