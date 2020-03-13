---
title: WebChat を Direct Line App Service 拡張機能と共に使用します。
titleSuffix: Bot Service
description: WebChat を Direct Line App Service 拡張機能と共に使用します。
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: aaba2a9de9f35d7615b3a7e5ff077554ef03a2df
ms.sourcegitcommit: 2109d9da53fdf65966f33ed1fa628a40ec851d35
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2020
ms.locfileid: "78280123"
---
# <a name="use-webchat-with-the-direct-line-app-service-extension"></a>WebChat を Direct Line App Service 拡張機能と共に使用します。

この記事では、Direct Line App Service 拡張機能と共に WebChat を使用する方法について説明します。

## <a name="get-your-direct-line-secret"></a>Direct Line シークレットを取得する

最初の手順では、Direct Line シークレットを見つけます。 これを行うには、「[ボットを Direct Line に接続する](bot-service-channel-connect-directline.md)」の記事に記載されている手順に従ってください。

## <a name="get-the-preview-version-of-directlinejs"></a>DirectLineJS のプレビュー バージョンを取得する
DirectLineJS のプレビュー バージョンはこちらにあります。 https://github.com/Jeffders/DirectLineAppServiceExtensionPreview/tree/master/libraries

## <a name="integrate-webchat-client"></a>WebChat クライアントを統合する

一般に、このアプローチは以前と同じです。 ただし、双方向の **WebSocket** トラフィックをサポートする **WebChat** の新しいバージョンが作成されています。これは、[https://directline.botframework.com/](https://directline.botframework.com/) に接続する代わりに、ホストされているボットに直接接続します。
ボットの Direct Line URL は `https://<your_app_service>.azurewebsites.net/.bot/` になります。ここで、`/.bot/` 拡張機能は App Service の Direct Line **エンドポイント**です。
独自のドメイン名を構成できる場合でも、Direct Line REST API にアクセスするために `/.bot/` パスを追加する必要があります。

1. 「[認証](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)」の記事の手順に従って、トークンのシークレットを交換します。 ただし、`https://directline.botframework.com/v3/directline/tokens/generate` でトークンを取得するのではなく、`https://<your_app_service>.azurewebsites.net/.bot/v3/directline/tokens/generate` で Direct Line App Service 拡張機能から直接トークンを生成します。  

1. トークンを取得したら、WebChat を使用する Web ページを次の変更で更新できます。

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <title>Direct Line Streaming Sample</title>
    <script src="~/directLine.js"></script>
    <script src="https://cdn.botframework.com/botframework-webchat/master/webchat.js"></script>
    <style>
        html, body {
            height: 100%
        }

        body {
            margin: 0
        }

        #webchat,

        #webchat > * {
            height: 100%;
            width: 100%;
        }
    </style>
</head>

<body>
    <div id="webchat" role="main"></div>
    <script>
        const activityMiddleware = () => next => card => {
            if (card.activity.type === 'trace') {
                // Return false means, don't render the trace activities
                return () => false;
            } else {
                return children => next(card)(children);
            }
        };


        var dl = new DirectLine.DirectLine({
            secret: '<your token>',
            domain: 'https://<your_site>.azurewebsites.net/.bot/v3/directline',
            webSocket: true,
            conversationId: '<your conversation id>'
        });
        window.WebChat.renderWebChat({
            activityMiddleware,
            directLine: dl,
            userID: '<your generated user id>'
        }, document.getElementById('webchat'));
    </script>
</body>
</html>

```
