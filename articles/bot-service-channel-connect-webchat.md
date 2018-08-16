---
title: ボットを Web チャット チャンネルに接続する | Microsoft Docs
description: Web チャット チャンネルに接続されたボットに、Web ページの Web チャット コントロールを使用する方法について説明します。
keywords: web チャット, ボット チャンネル, web ページ, 秘密鍵, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: b560f9f43fc596bc8062676136819922d227d37b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301266"
---
# <a name="connect-a-bot-to-web-chat"></a>ボットを Web チャットに接続する
Bot Service を使用して[ボットを作成](bot-service-quickstart.md)すると、Web チャット チャンネルが自動的に構成されます。 Web チャット チャンネルには、ユーザーが Web ページで直接ボットと対話できるようにする、Web チャット コントロールが含まれています。

![Web チャットのサンプル](~/media/bot-service-channel-webchat/webchat-sample.png)

Bot Framework ポータル内の Web チャット チャンネルには、Web チャット コントロールを Web ページに埋め込むために必要なものがすべて含まれています。 Web チャット コントロールを使用するために必要なのは、ボットの秘密鍵を取得し、コントロールを Web ページに埋め込むことだけです。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a id="step-1">ボットの秘密鍵を取得する</a>

1. [Azure portal](http://portal.azure.com) でボットを開き、**[チャンネル]** ブレードをクリックします。

2. **[Web チャット]** チャンネルの **[編集]** をクリックします。  
![Web チャット チャンネル](~/media/bot-service-channel-webchat/bot-service-channel-list.png)

3. **[Secret keys]\(秘密鍵\)** で、最初の鍵の **[表示]** をクリックします。  
![秘密鍵](~/media/bot-service-channel-webchat/secret-key.png)

4. **秘密鍵**と**埋め込みコード**をコピーします。

5. **[Done]** をクリックします。

## <a name="embed-the-web-chat-control-in-your-website"></a>Web チャット コントロールを Web サイトに埋め込む

2 つの方法のいずれかを使用して、Web チャット コントロールを Web サイトに埋め込むことができます。

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>方法 1 - シークレットを非表示にしておき、シークレットをトークンと交換し、埋め込みを生成する

サーバー間の要求を実行して Web チャット シークレットを一時トークンと交換できる場合や、自分のボットを他の開発者が Web サイトに簡単に埋め込むことができないようにする場合は、この方法を使用します。 この方法を使用しても、自分のボットを他の開発者が Web サイトに埋め込むことを完全に防ぐことができるわけではありませんが、これを行うことが難しくなります。

シークレットをトークンと交換し、埋め込みを生成するには、次の手順に従います。

1. `https://webchat.botframework.com/api/tokens` に対して **GET** 要求を発行し、`Authorization` ヘッダーを使用して Web チャット シークレットを渡します。 下記の要求の例に示すように、`Authorization` ヘッダーでは `BotConnector` スキームを使用し、シークレットが含まれています。

2. **GET** 要求への応答には、**iframe** 内で Web チャット コントロールをレンダリングして会話を開始する際に使用できるトークンが含まれています (トークンは引用符で囲まれています)。 トークンは 1 つの会話でのみ有効です。別の会話を開始するには、新しいトークンを生成する必要があります。

3. Bot Framework ポータル内の Web チャット チャンネルからコピーした `iframe` **埋め込みコード**内で (前述の[手順 1.](#step-1) を参照)、`s=` パラメーターを `t=` に変更し、"YOUR_SECRET_HERE" をトークンに置き換えます。 

> [!NOTE]
> トークンは、有効期限が切れる前に自動的に更新されます。 

##### <a name="example-request"></a>要求の例

```requestGET https://webchat.botframework.com/api/tokens Authorization: BotConnector YOUR_SECRET_HERE
```

##### Example response 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>iframe の例 (トークンを使用)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>html コードの例
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a>方法 2 - シークレットを使用して、Web チャット コントロールを Web サイトに埋め込む

自分のボットを他の開発者が Web サイトに簡単に埋め込むことができるようにする場合は、この方法を使用します。 

> [!WARNING]
> この方法を使用すると、他の開発者は埋め込みコードをコピーするだけで、Web サイトにボットを埋め込むことができます。

`iframe` タグ内でシークレットを指定して、Web サイトにボットを埋め込むには、次の手順に従います。

1. Bot Framework ポータル内の Web チャット チャンネルから `iframe` **埋め込みコード**をコピーします (前述の[手順 1.](#step-1) を参照)。

2. その**埋め込みコード**内で、"YOUR_SECRET_HERE" を、同じページからコピーした**秘密鍵**の値に置き換えます。

##### <a name="example-iframe-using-secret"></a>iframe の例 (シークレットを使用)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>Web チャット コントロールのスタイルを設定する

`iframe` の `style` 属性を使用して `height` と `width` を指定することで、Web チャット コントロールのサイズを変更できます。

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![チャット コントロール クライアント](~/media/chatwidget-client.png)

## <a name="additional-resources"></a>その他のリソース

GitHub で Web チャット コントロールの[ソース コードをダウンロード](https://github.com/Microsoft/BotFramework-WebChat)できます。
