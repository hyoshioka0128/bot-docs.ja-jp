---
title: Node.js ボットと Direct Line App Service 拡張機能
titleSuffix: Bot Service
description: Node.js ボットが Direct Line App Service 拡張機能を使って機能するようにする
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: dev
ms.date: 01/15/2020
ms.openlocfilehash: 68961133eef80b22939bd4ad251a22f10f874c2c
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "81395665"
---
# <a name="configure-nodejs-bot-for-extension"></a>拡張機能のための Node.js ボットの構成

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

この記事では、**名前付きパイプ**を使って機能するようにボットを更新する方法、およびボットがホストされている **Azure App Service** リソースで Direct Line App Service 拡張機能を有効にする方法について説明します。  

## <a name="prerequisites"></a>前提条件

次に説明する手順を実行するには、**Azure App Service** リソースと、関連する **App Service** が Azure にある必要があります。

## <a name="enable-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

このセクションでは、ボットのチャネル構成からのキーと、ボットがホストされている **Azure App Service** リソースを使用して Direct Line App Service 機能拡張を有効にする方法について説明します。

## <a name="update-nodejs-bot-to-use-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を使用するように Node.js ボットを更新する

1. Node.js ボットを Direct Line App Service 拡張機能と共に使用するには、BotBuilder v4.7.0 以上が必要です。
1. SDK の v4.x を使用して既存のボットを更新するには
    1. ボットのルート ディレクトリで `npm install --save botbuilder` を実行して、最新のパッケージに更新します。
1. アプリで **Direct Line App Service 拡張機能の名前付きパイプ** を使用できるようにします:
    - ボットの index.js を (アダプターとボットの割り当ての下で) 更新して、次のものを含めます。
    
    ```Node.js
    
    adapter.useNamedPipe(async (context) => {
        await myBot.run(context);
        },
        process.env.APPSETTING_WEBSITE_SITE_NAME + '.directline'
    );
    ```   

1. `index.js` ファイルを保存します。
1. 既定の `Web.Config` ファイルを更新して、Direct Line App Service 拡張機能に必要な `AspNetCore` ハンドラーをサービス要求に追加します。
    - ボットの `wwwroot` ディレクトリで `Web.Config` ファイルを見つけ、既定の内容を次のように置き換えます。
    
    ```XML
    
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.webServer>
        <handlers>      
          <add name="aspNetCore" path="*/.bot/*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
          <add name="iisnode" path="*" verb="*" modules="iisnode" />
        </handlers>
       </system.webServer>
    </configuration>
    ```
    
1. `appsettings.json` ファイルを開き、次の値を入力します。
    1. `"MicrosoftAppId": "<secret Id>"`
    1. `"MicrosoftAppPassword": "<secret password>"`

    これらの値は、**appid** と、サービス登録グループに関連付けられた **appSecret** です。

1. ボットを Azure App Service に**発行**します。

### <a name="gather-your-direct-line-app-service-extension-keys"></a>Direct Line App Service 拡張機能のキーを収集する

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、目的の **Azure Bot Service** リソースを見つけます。
1. **[チャネル]** をクリックして、ボットのチャネルを構成します。
1. **Direct Line** チャネルがまだ有効になっていない場合は、クリックして有効にします。 
1. 既に有効になっている場合は、[Connect to channels]\(チャネルに接続\) テーブルの Direct Line 行で **[編集]** リンクをクリックします。
1. **[アプリ サービスの拡張キー]** セクションまで下にスクロールします。 
1. **[表示] リンク**をクリックしていずれかのキーを表示し、その値をコピーします。

![App Service 拡張機能キー](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Direct Line App Service 拡張機能を有効にする

1. ブラウザーで [Azure portal](https://portal.azure.com/) に移動します。
1. Azure portal で、ボットをホストしている、またはホストする予定の Web アプリの **Azure App Service** リソース ページを見つけます
1. **[Configuration]\(構成\)** をクリックします。 *[アプリケーションの設定]* セクションで、次の新しい設定を追加します。

    |名前|値|
    |---|---|
    |DirectLineExtensionKey|<App_Service_Extension_Key_From_Section_1>|
    |DIRECTLINE_EXTENSION_VERSION|latest|

1. *[構成]* セクション内で、 **[全般設定]** セクションをクリックし、 **[Web ソケット]** を有効にします。
1. **[保存]** をクリックして設定を保存します。 これにより、Azure App Service が再起動されます。

### <a name="confirm-direct-line-app-extension-and-the-bot-are-initialized"></a>Direct Line アプリ拡張機能とボットが初期化されていることを確認します

1. ブラウザーで、 https://<your_app_service>.azurewebsites.net/.bot に移動します。 すべて正しければ、ページは JSON コンテンツ `{"v":"123","k":true,"ib":true,"ob":true,"initialized":true}` を返します。 これは、**すべてが正常に動作している**場合に取得される情報です。ここでは、次のようになります。

    - **v** によって、Direct Line App Service 拡張機能 (ASE) のビルドバージョンが表示されます。
    - **k** によって、Direct Line ASE がその構成から App Service 拡張機能のキーを読み取れるかどうかを決定します。 
    - **initialized** によって、Direct Line ASE が App Service 拡張機能キーを使用して Azure Bot Service からボット メタデータをダウンロードできるかどうかを決定します。
    - **ib** によって、Direct Line ASE がボットとの受信接続を確立できるかどうかを決定します。
    - **ob** によって、Direct Line ASE がボットとの送信接続を確立できるかどうかを決定します。 
