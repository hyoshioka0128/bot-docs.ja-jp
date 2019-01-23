---
title: Bot Framework 認証のトラブルシューティング | Microsoft Docs
description: ボットの認証エラーをトラブルシューティングする方法について説明します。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/17
ms.openlocfilehash: 2335ac34292e224f44a09820574f3bd9de00eda4
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224657"
---
# <a name="troubleshooting-bot-framework-authentication"></a>Bot Framework 認証のトラブルシューティング

このガイドは、一連のシナリオを評価してどこに問題があるかを判断することで、ボットの認証問題をトラブルシューティングするのに役立ちます。 

> [!NOTE]
> このガイドのすべての手順を完了するには、[Bot Framework Emulator][Emulator] をダウンロードして使用する必要があります。さらに、<a href="https://dev.botframework.com" target="_blank">Bot Framework Portal</a> でボットの登録設定にアクセスできる必要があります。

## <a id="PW"></a> アプリ ID とパスワード

ボットのセキュリティは、Bot Framework にボットを登録するときに取得する **Microsoft アプリ ID** と **Microsoft アプリ パスワード**によって構成されます。 これらの値は、通常、ボットの構成ファイル内に指定され、Microsoft アカウント サービスからアクセス トークンを取得するために使用されます。 

まだ行っていない場合は、[ボットを Azure にデプロイ](~/bot-builder-howto-deploy-azure.md)し、認証のために使用できる **Microsoft アプリ ID** と **Microsoft アプリ パスワード**を取得してください。 

> [!NOTE]
> デプロイ済みのボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID と MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」をご覧ください。

## <a name="step-1-disable-security-and-test-on-localhost"></a>手順 1:セキュリティを無効にし、localhost 上でテストする

この手順では、セキュリティが無効なときに、ボットが localhost 上でアクセス可能であり、機能することを確認します。 

> [!WARNING]
> ボットのセキュリティを無効にすると、不明な攻撃者がユーザーを偽装する可能性があります。 次の手順は、保護されているデバッグ環境で作業中の場合のみ実装してください。

### <a id="disable-security-localhost"></a>セキュリティを無効にする

ボットのセキュリティを無効にするには、その構成設定を編集して、アプリ ID とパスワードを削除します。 

::: moniker range="azure-bot-service-3.0"

Bot Framework SDK for .NET を使用している場合は、Web.config ファイル内の次の設定を編集します。 

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Bot Framework SDK for Node.js を使用している場合は、次の値を編集します (または該当する環境変数を更新します)。

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

::: moniker-end

::: moniker range="azure-bot-service-4.0"

Bot Framework SDK for .NET を使用している場合は、`.bot` ファイル内の次の設定を編集します。

```json
"services": [
  {
    "appId": "<your app ID>",
    "appPassword": "<your app password>",
  }
]
```

Bot Framework SDK for Node.js を使用している場合は、次の値を編集します (または該当する環境変数を更新します)。

```javascript
const adapter = new BotFrameworkAdapter({
    appId: null,
    appPassword: null
});
```

構成に `.bot` ファイルを使用している場合は、`appId` と `appPassword` を `""` に更新できます。

::: moniker-end

### <a name="test-your-bot-on-localhost"></a>ボットを localhost 上でテストする 

次に、Bot Framework Emulator を使用して、ボットを localhost 上でテストします。

1. ボットを localhost 上で起動します。
2. Bot Framework Emulator を起動します。
3. エミュレーターを使用してボットに接続します。
    - エミュレーターのアドレス バーに「`http://localhost:port-number/api/messages`」と入力します。**port-number** は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 
    - **[Microsoft アプリ ID]** フィールドと **[Microsoft アプリ パスワード]** フィールドの両方が空白であることを確認します。
    - **[接続]** をクリックします。
4. ボットへの接続をテストするには、エミュレーターにテキストを入力し、Enter キーを押します。

ボットが入力に応答し、エラーがチャット ウィンドウに表示されなければ、セキュリティが無効なときにボットはアクセス可能であり、機能することが確認されました。 [手順 2](#step-2) に進んでください。

チャット ウィンドウにエラーが示された場合は、エラーをクリックすると詳細が表示されます。 次のような一般的な問題があります。

* エミュレーター設定に指定されているボットのエンドポイントが正しくない。 URL に適切なポート番号と、末尾に適切なパスが含まれていることを確認します (例: `/api/messages`)。
* エミュレーター設定に指定されているボットのエンドポイントが `https` で始まっている。 localhost では、エンドポイントは、`http` で始める必要があります。
* エミュレーター設定で、**[Microsoft アプリ ID]** フィールドと **[Microsoft アプリ パスワード]** フィールドの値が指定されている。 どちらのフィールドも、空にする必要があります。
* ボットのセキュリティが無効になっていない。 ボットのアプリ ID とパスワードの値が、両方とも指定されていないことを[確認](#disable-security-localhost)します。

## <a id="step-2"></a>手順 2: ボットのアプリ ID とパスワードを確認する

この手順では、ボットが認証のために使用するアプリ ID とパスワードが有効であることを確認します  (これらの値がわからない場合は、今すぐ[取得](#PW)してください)。 

> [!WARNING]
> 次の手順は、`login.microsoftonline.com` の SSL 検証を無効にします。 この手順は、セキュリティで保護されたネットワークでのみ実行し、実行後にアプリケーションのパスワードを変更することを検討してください。

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>Microsoft ログイン サービスに HTTP 要求を発行する

これらの手順は、[cURL](https://curl.haxx.se/download.html) を使用して Microsoft ログイン サービスに HTTP 要求を発行する方法について説明しています。 Postman などの別のツールを使用できますが、要求が Bot Framework の[認証プロトコル](~/rest-api/bot-framework-rest-connector-authentication.md)に準拠していることを確認してください。

ボットのアプリ ID とパスワードが有効であることを確認するには、**cURL** を使用して次の要求を発行します。`APP_ID` と `APP_PASSWORD` を、ボットのアプリ ID とパスワードに置き換えてください。

> [!TIP]
> パスワードには、次の呼び出しを無効にする特殊文字が含まれている場合があります。 その場合は、パスワードを URL エンコードに変換してみてください。

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

この要求は、ボットのアプリ ID とパスワードを、アクセス トークンと交換することを試します。 要求が成功した場合は、`access_token` プロパティとその他のプロパティを含む JSON ペイロードが受信されます。 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

この要求が成功すれば、要求内に指定したアプリ ID とパスワードが有効であることが確認されました。 [手順 3](#step-3) に進んでください。

要求に対してエラーを受信した場合は、その応答をしらべて、エラーの原因を識別します。 応答でアプリ ID またはパスワードが無効であることが示された場合は、Bot Framework Portal から[正しい値を取得](#PW)し、新しい値で要求を再発行し、それらが有効であることを確認します。 

## 手順 3:セキュリティを有効にし、localhost 上でテストする<a id="step-3"></a>

この時点で、セキュリティが無効なときに、ボットが localhost 上でアクセス可能であり、機能すること、およびボットが認証のために使用するアプリ ID とパスワードが有効であることが確認されています。 この手順では、セキュリティが有効なときに、ボットが localhost 上でアクセス可能であり、機能することを確認します。

### <a id="enable-security-localhost"></a> セキュリティ チェックを有効にする

ボットのセキュリティは、ボットが localhost 上でのみ実行される場合でも、Microsoft のサービスに依存します。 ボットのセキュリティを有効にするには、その構成設定を編集して、[手順 2](#step-2) で確認したアプリ ID とパスワードの値を入力します。  さらに、パッケージが最新の状態であること (具体的には `System.IdentityModel.Tokens.Jwt` と `Microsoft.IdentityModel.Tokens`) を確認します。

Bot Framework SDK for .NET を使用している場合は、`appsettings.config` ファイルでこれらの設定を指定するか、または `.bot` ファイルで対応する値を指定します。

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

Bot Framework SDK for Node.js を使用している場合は、次の設定を入力します (または該当する環境変数を更新します)。

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> 自分のボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID and MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。

### <a name="test-your-bot-on-localhost"></a>ボットを localhost 上でテストする 

次に、Bot Framework Emulator を使用して、ボットを localhost 上でテストします。

1. ボットを localhost 上で起動します。
2. Bot Framework Emulator を起動します。
3. エミュレーターを使用してボットに接続します。
    - エミュレーターのアドレス バーに「`http://localhost:port-number/api/messages`」と入力します。**port-number** は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。 
    - ボットのアプリ ID を **[Microsoft アプリ ID]** フィールドに入力します。
    - ボットのパスワードを **[Microsoft アプリ パスワード]** フィールドに入力します。
    - **[接続]** をクリックします。
4. ボットへの接続をテストするには、エミュレーターにテキストを入力し、Enter キーを押します。

ボットが入力に応答し、エラーがチャット ウィンドウに表示されなければ、セキュリティが有効なときに、ボットがアクセス可能であり、機能することが確認されました。  [手順 4](#step-4) に進んでください。

チャット ウィンドウにエラーが示された場合は、エラーをクリックすると詳細が表示されます。 次のような一般的な問題があります。

* エミュレーター設定に指定されているボットのエンドポイントが正しくない。 URL に適切なポート番号と、末尾に適切なパスが含まれていることを確認します (例: `/api/messages`)。
* エミュレーター設定に指定されているボットのエンドポイントが `https` で始まっている。 localhost では、エンドポイントは、`http` で始める必要があります。
* エミュレーター設定で、**[Microsoft アプリ ID]** フィールドと **[Microsoft アプリ パスワード]** に有効な値が含まれていない。 両方のフィールドには、[手順 2](#step-2) で確認した該当する値を入力する必要があります。
* ボットのセキュリティが有効になっていない。 ボットの構成設定にアプリ ID とパスワードの両方の値が指定されていることを[確認](#enable-security-localhost)します。

## 手順 4:ボットをクラウドでテストする<a id="step-4"></a>

この時点で、セキュリティが無効なときに、ボットが localhost 上でアクセス可能であり、機能すること、ボットのアプリ ID とパスワードが有効であること、およびセキュリティが有効なときに、ボットが localhost 上でアクセス可能であり、機能することを確認しています。 この手順では、ボットをクラウドにデプロイし、セキュリティが有効なクラウドでボットがアクセス可能であり、機能することを確認します。 

### <a name="deploy-your-bot-to-the-cloud"></a>ボットをクラウドにデプロイする

Bot Framework では、ボットはインターネットからアクセス可能である必要があるため、Azure などのクラウド ホスティング プラットフォームにボットをデプロイする必要があります。 ボットをデプロイする前に、[手順 3](#step-3) の説明に従って、必ずセキュリティを有効にしてください。

> [!NOTE]
> クラウド ホスティング プロバイダーをまだ利用していない場合は、<a href="https://azure.microsoft.com/en-us/free/" target="_blank">無料アカウント</a>を登録できます。 

ボットを Azure にデプロイする場合は、アプリケーション用に SSL が自動的に構成され、それにより Bot Framework が必要とする **HTTPS** エンドポイントが有効になります。 別のクラウド ホスティング プロバイダーにデプロイする場合は、ボットが **HTTPS** エンドポイントを使用できるように、アプリケーションが SSL 用に構成されることを確認してください。

### <a name="test-your-bot"></a>ボットをテストする 

セキュリティが有効なクラウドでボットをテストするには、次の手順を完了します。

1. ボットが正常にデプロイされ、実行されていることを確認します。 
2. <a href="https://dev.botframework.com" target="_blank">Bot Framework Portal</a> にサインインします。
3. **[マイ ボット]** をクリックします。
4. テストするボットを選択します。
5. **[テスト]** をクリックして、埋め込み Web チャット コントロールでボットを開きます。
6. ボットへの接続をテストするには、Web チャット コントロールにテキストを入力し、Enter キーを押します。

チャット ウィンドウにエラーが示された場合は、エラー メッセージを使用して、エラーの原因を特定します。 次のような一般的な問題があります。 

* Bot Framework Portal でボット用の **[設定]** ページに指定された**メッセージング エンドポイント**が正しくない。 URL の末尾に適切なパスが含まれていることを確認します (例: `/api/messages`)。
* Bot Framework Portal でボット用の **[設定]** ページに指定された**メッセージング エンドポイント**が `https` で始まっていない、または Bot Framework で信頼されていない。 ボットは、有効なチェーンで信頼された証明書を持っている必要があります。
* ボットの構成にアプリ ID またはパスワードが不足しているか、値が正しくない。 ボットの構成設定で、アプリ ID とパスワードの両方に有効な値が指定されていることを[確認](#enable-security-localhost)します。

ボットが入力に適切に応答すれば、セキュリティが有効なクラウドでボットがアクセス可能であり、機能することが確認されました。 この時点で、ボットは、セキュリティで保護された状態で、Skype、Facebook Messenger、Direct Line などの[チャネルに接続する](~/bot-service-manage-channels.md)準備が整いました。

## <a name="additional-resources"></a>その他のリソース

上記の手順を完了した後で、問題が発生した場合は、以下を実行できます。

* Bot Framework Emulator と <a href="https://ngrok.com/" target="_blank">ngrok</a> を使用して、[クラウド内のボットをデバッグする](~/bot-service-debug-emulator.md)。
* [Fiddler](https://www.telerik.com/fiddler) などのプロキシ化ツールを使用して、ボットとの間の HTTPS トラフィックを検査する。 *Fiddler はマイクロソフト製品ではありません。*
* Bot Framework で使用される認証テクノロジーについては、[Bot Connector 認証ガイド][BotConnectorAuthGuide]を参照してください。
* Bot Framework の[サポート][Support] リソースを使用して、他のユーザーにヘルプを要請する。 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
