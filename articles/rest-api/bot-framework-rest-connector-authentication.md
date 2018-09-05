---
title: 認証の要求 | Microsoft Docs
description: Bot Connector API および Bot State API で API 要求を認証する方法を説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 998586820a0489bc4cca1d25b53cb6ac8162c452
ms.sourcegitcommit: 0b2be801e55f6baa048b49c7211944480e83ba95
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/28/2018
ms.locfileid: "43115047"
---
# <a name="authentication"></a>Authentication

ご利用のボットと Bot Connector サービスとの通信は、セキュリティ保護されたチャネル (SSL/TLS) で HTTP を使用して行われます。 ご利用のボットから Connector サービスに要求を送信するときは、Connector サービスでボットの ID を検証するために使用できる情報を含める必要があります。 同様に、Connector サービスからご利用のボットに送信される要求には、ボットでサービスの ID を検証するために使用できる情報が含まれている必要があります。 この記事では、ボットと Bot Connector サービスの間で行われるサービス レベルの認証について、そのテクノロジと要件を説明します。 利用する認証コードを独自に記述する場合、ボットと Bot Connector サービスの間でメッセージを交換できるようにするには、この記事で説明するセキュリティ プロシージャを実装する必要があります。

> [!IMPORTANT]
> 独自の認証コードを記述する場合は、すべてのセキュリティ プロシージャを正しく実装することが不可欠です。 この記事のすべての手順を実装すれば、攻撃者が、ご利用のボットに送信されたメッセージを読み取ったり、ボットになりすましてメッセージを送信したり、秘密鍵を盗んだりする危険性を軽減できます。 

[Bot Builder SDK for .NET](../dotnet/bot-builder-dotnet-overview.md) または [Bot Builder SDK for Node.js](../nodejs/index.md) をお使いの場合、この記事で説明するセキュリティ プロシージャは SDK によって自動的に実装されるため、お客様が実装する必要はありません。 [登録](../bot-service-quickstart-registration.md)の際にご利用のボットのために取得した AppID とパスワードを使用してプロジェクトを設定するだけで、あとは SDK によって処理されます。

> [!WARNING]
> 2016 年 12 月、Bot Framework セキュリティ プロトコル v3.1 には、トークンの作成と検証の際に使用されるいくつかの値に変更が加えられました。 2017 年の冬に Bot Framework セキュリティ プロトコル v3.2 が登場し、トークンの作成と検証の際に使用される値に変更が加えられました。
> 詳細については、「[Security protocol changes](#security-protocol-changes)」 (セキュリティ プロトコルの変更) を参照してください。

## <a name="authentication-technologies"></a>認証テクノロジ

ボットと Bot Connector の間で信頼を確立するために、次の 4 つの認証テクノロジが使用されます。

| テクノロジ | 説明 |
|----|----|
| **SSL/TLS** | SSL/TLS は、すべてのサービス間接続に対して使用されます。 `X.509v3` 証明書は、すべての HTTPS サービスの ID を確立するために使用されます。 **クライアントでは、サービスが信頼でき、有効であることを確認するために、必ずサービス証明書を調べる必要があります**  (この手法では、クライアント証明書は使用されません)。 |
| **OAuth 2.0** | Microsoft Account (MSA)/AAD v2 ログイン サービスに対して OAuth 2.0 ログインを使用すると、ボットからメッセージを送信する際に使用できるセキュリティ トークンが作成されます。 このトークンはサービス間トークンであり、ユーザー ログインは不要です。 |
| **JSON Web トークン (JWT)** | JSON Web トークンは、ボットとの間で送受信されるトークンをエンコードするために使用されます。 **クライアントでは、受信したすべての JWT トークンを、この記事で説明している要件に従って検証する必要があります。** |
| **OpenID メタデータ** | Bot Connector サービスからは、既知の静的なエンドポイントにある OpenID メタデータに対して独自の JWT トークンによる署名を行うためにこのサービスによって使用される、有効なトークンの一覧が公開されます。 |

この記事では、標準的な HTTPS と JSON を通してこれらのテクノロジを使用する方法を説明します。 特殊な SDK は不要ですが、OpenID 用のヘルパーなどは役立つことがあります。

## <a id="bot-to-connector"></a>使用しているボットから Bot Connector サービスへの認証要求

Bot Connector サービスと通信するには、次の形式を使用して、各 API 要求の `Authorization` ヘッダーでアクセス トークンを指定する必要があります。 

```http
Authorization: Bearer ACCESS_TOKEN
```

次の図に、ボットから Connector への認証の手順を示します。

![MSA ログイン サービスに対して認証し、次にボットに対して認証する](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> Bot Framework に[ご利用のボットを登録](../bot-service-quickstart-registration.md)して、その AppID とパスワードを取得する必要があります (この操作をまだ行っていない場合)。 アクセス トークンを要求するには、ボットの AppID とパスワードが必要になります。

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>手順 1: MSA/AAD v2 ログイン サービスにアクセス トークンを要求する

MSA/AAD v2 ログイン サービスにアクセス トークンを要求するには、次の要求を発行します。**MICROSOFT-APP-ID** および **MICROSOFT-APP-PASSWORD** は、ご利用のボットを Bot Framework に[登録](../bot-service-quickstart-registration.md)したときに取得した AppID とパスワードに置き換えてください。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>手順 2: MSA/AAD v2 ログイン サービスによる応答から JWT トークンを取得する

お使いのアプリケーションが MSA/AAD v2 ログイン サービスによって認証されると、JSON 応答の本体によってご自身のアクセス トークン、トークンのタイプ、トークンの有効期限 (秒単位) が指定されます。 

トークンを `Authorization` 要求のヘッダーに追加するときは、この応答で指定されている値のとおりに使用する必要があります (トークンの値をエスケープしたりエンコードしたりしないでください)。 アクセス トークンは、有効期限が切れるまで有効です。 トークンの有効期限が切れてもご利用のボットの実行に影響が及ばないように、トークンをキャッシュし、事前に最新の情報に更新しておくことが考えられます。

この例では、MSA/AAD v2 ログイン サービスからの応答を示します。

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>手順 3: 要求の Authorization ヘッダーで JWT トークンを指定する

Bot Connector サービスに API 要求を送信するときは、次の形式を使用して要求の `Authorization` ヘッダーでアクセス トークンを指定する必要があります。

```http
Authorization: Bearer ACCESS_TOKEN
```

Bot Connector サービスに送信するすべての要求で、`Authorization` ヘッダーにアクセス トークンを含める必要があります。 トークンが正しい形式であり、有効期限が切れておらず、MSA/AAD v2 ログイン サービスによって作成されたものである場合、要求は Bot Connector サービスによって認証されます。 トークンが要求の送信元ボットのものであることを確認するために、追加のチェックが実行されます。

次の例で、要求の `Authorization` ヘッダーでアクセス トークンを指定する方法を示します。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> JWT トークンは、Bot Connector サービスに送信する要求の `Authorization` ヘッダーにのみ指定します。 セキュリティ保護されていないチャネルでトークンを送信しないでください。また、他のサービスに送信する HTTP リクエストに含めないでください。 MSA/AAD v2 ログイン サービスから取得する JWT トークンは、パスワードと同様のものです。細心の注意を払って扱う必要があります。 トークンを所持していれば、誰でもそれを使用して使用しているボットの代理として操作を実行できてしまいます。 

#### <a name="bot-to-connector-example-jwt-components"></a>ボットからコネクタ: サンプル JWT コンポーネント

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> 実際の場面では、フィールドは異なる可能性があります。 すべての JWT トークンを上記で指定されているように作成し、検証します。

## <a id="connector-to-bot"></a> Bot Connector サービスからご利用のボットへの要求を認証する

Bot Connector サービスからご利用のボットに要求が送信されるときは、署名付きの JWT トークンが要求の `Authorization` ヘッダーで指定されます。 署名付き JWT トークンの認証を検証することにより、Bot Connector サービスからの呼び出しをご利用のボットで認証できます。 

次の図に、コネクタからボットへの認証の手順を示します。

![Bot Connector からご利用のボットへの呼び出しを認証する](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> 手順 2: OpenID メタデータ ドキュメントを取得する

OpenID メタデータ ドキュメントでは、Bot Connector サービスの有効な署名キーの一覧を含むもう 1 つのドキュメントの場所が指定されます。 OpenID メタデータ ドキュメントを取得するには、次の要求を HTTPS 経由で発行します。

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> これは静的な URL であり、アプリケーションにハードコーディングすることができます。 

次の例で、`GET` 要求に対して返される OpenID メタデータ ドキュメントを示します。 `jwks_uri` プロパティでは、Bot Connector サービスの有効な署名キーが含まれるドキュメントの位置が指定されます。

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> 手順 3: 有効な署名キーの一覧を取得する

有効な署名キーの一覧を取得するには、OpenID メタデータ ドキュメントで `jwks_uri` プロパティによって指定されている URL に、HTTPS 経由で `GET` 要求を発行します。 例: 

```http
GET https://login.botframework.com/v1/.well-known/keys
```

応答の本体には、ドキュメントが [JWK 形式](https://tools.ietf.org/html/rfc7517)で指定されますが、各キーに対応する追加のプロパティ `endorsements` も含まれています。 キーの一覧は比較的安定しており、長期間キャッシュしておくことができます (既定では、Bot Builder SDK の中に 5 日間)。

各キーの `endorsements` プロパティには、少なくとも 1 つの保証文字列が含まれており、これを使用することにより、受信した要求の [Activity][Activity] オブジェクト内で `channelId` プロパティによって指定されているチャネル ID が正しいことを検証できます。 保証が必要なチャネル ID の一覧は、各ボット内で設定できます。 既定では、これは公開済みのすべてのチャネル ID の一覧になりますが、ボットの開発者は一部のチャネル ID 値を選択し、何らかの方法でオーバーライドすることができます。 チャネル ID に対して保証が必要な場合は、以下が適用されます。

- ご利用のボットにそのチャネル ID で送信されるすべての [Activity][Activity] オブジェクトには、そのチャネルに対する保証で署名された JWT トークンが付随していることを要件とする必要があります。 
- 保証が存在しない場合、ご利用のボットは、**HTTP 403 (アクセス不可)** 状態コードを返すことによって要求を拒否する必要があります。

### <a name="step-4-verify-the-jwt-token"></a>手順 4: JWT トークンを検証する

Bot Connector サービスから送信されたトークンの信頼性を検証するには、要求の `Authorization` ヘッダーからトークンを抽出し、トークンを解析し、その内容を確認して署名を検証する必要があります。 

多くのプラットフォームに JWT 解析ライブラリが用意されており、大部分のユーザーは JWT トークン用の安全で信頼性のある解析を実装しています。それでも、通常はトークンの特定の特性 (発行者、オーディエンスなど) に正しい値が含まれるようにするために、これらのライブラリを設定する必要があります。 トークンの解析では、トークンが確実に以下の要件を満たすように、解析ライブラリを設定するか、独自の検証を作成する必要があります。

1. トークンが HTTP `Authorization` ヘッダーに含まれ、"Bearer" スキームで送信された。
2. トークンは有効な JSON であり、[JWT 標準](http://openid.net/specs/draft-jones-json-web-token-07.html)に準拠している。
3. トークンに "issuer" 要求が含まれ、その値が `https://api.botframework.com` である。
4. トークンに "audience" 要求が含まれ、その値がボットの Microsoft AppID と同じである。
5. トークンの有効期限が切れていない。 業界標準の clock-skew は 5 分です。
6. トークンに有効な暗号化署名があり、そのキーが、[手順 2](#openid-metadata-document) で取得した Open ID メタデータ ドキュメントの `id_token_signing_alg_values_supported` プロパティで指定されている署名アルゴリズムを使用して[手順 3](#connector-to-bot-step-3) で取得した OpenID キー ドキュメント内の一覧に含まれている。
7. トークンに "serviceUrl" 要求が含まれ、その値が、受信した要求の [Activity][Activity] オブジェクトのルートにある `servieUrl` プロパティと一致する。 

トークンがこれらの要件を満たさない場合、ご利用のボットは、**HTTP 403 (アクセス不可)** 状態コードを返すことによって要求を拒否する必要があります。

> [!IMPORTANT]
> これらの要件はすべて重要ですが、要件 4 と 6 は特に重要です。 これらの検証要件のすべてを実装しないと、ボットは攻撃に対して無防備になり、ボットから JWT トークンが漏洩するおそれがあります。

実装する際は、ボットに送信される JWT トークンの検証を無効にする手段が公開されないようにする必要があります。

#### <a name="connector-to-bot-example-jwt-components"></a>コネクタからボット: サンプル JWT コンポーネント

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 実際の場面では、フィールドは異なる可能性があります。 すべての JWT トークンを上記で指定されているように作成し、検証します。

## <a id="emulator-to-bot"></a> Bot Framework Emulator からご利用のボットへの要求を認証する

> [!WARNING]
> 2017 年の冬に、Bot Framework セキュリティ プロトコルの v3.2 が登場しました。 この新しいバージョンでは、Bot Framework Eumaltor とご利用のボットの間で交換されるトークンに、新しい "issuer" 値が追加されます。 この変更に備えるために、v3.1 と v3.2 の両方について issuer 値を確認する方法を以下の手順に示します。 

[Bot Framework Emulator](../bot-service-debug-emulator.md) は、ご利用のボットの機能をテストするために使用できるデスクトップ ツールです。 Bot Framework Emulator では、上記の説明と同じ[認証テクノロジ](#authentication-technologies)が使用されますが、実際の Bot Connector サービスを偽装することはできません。 代わりに、ボットによって作成されるものと同じトークンを作成するためにエミュレーターをご利用のボットに接続するときには、ご自身が指定する Microsoft AppID と Microsoft AppPassword が使用されます。 エミュレーターからご利用のボットに要求が送信されるときは、要求の `Authorization` ヘッダー内に JWT トークンが指定されるため、実質的にボット独自の資格情報を使用して要求が認証されます。 

認証ライブラリを実装してある場合に Bot Framework Emulator からの要求を受け入れるには、この追加的な検証パスを指定する必要があります。 このパスの構造は[コネクタからボット](#connector-to-bot)への検証パスと似ていますが、Bot Connector の OpenID ドキュメントの代わりに MSA の OpenID ドキュメントが使用されます。

次の図に、エミュレーターからボットへの認証の手順を示します。

![Bot Framework Emulator からご利用のボットへの呼び出しを認証する](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>手順 2: MSA OpenID メタデータ ドキュメントを取得する

OpenID メタデータ ドキュメントでは、有効な署名キーの一覧を含むもう 1 つのドキュメントの場所が指定されます。 MSA OpenID メタデータ ドキュメントを取得するには、次の要求を HTTPS 経由で発行します。

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

次の例で、`GET` 要求に対して返される OpenID メタデータ ドキュメントを示します。 `jwks_uri` プロパティでは、有効な署名キーが含まれるドキュメントの位置が指定されます。

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> 手順 3: 有効な署名キーの一覧を取得する

有効な署名キーの一覧を取得するには、OpenID メタデータ ドキュメントで `jwks_uri` プロパティによって指定されている URL に、HTTPS 経由で `GET` 要求を発行します。 例: 

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

応答の本体には、ドキュメントが [JWK 形式](https://tools.ietf.org/html/rfc7517)で指定されます。 

### <a name="step-4-verify-the-jwt-token"></a>手順 4: JWT トークンを検証する

エミュレーターから送信されたトークンの信頼性を検証するには、要求の `Authorization` ヘッダーからトークンを抽出し、トークンを解析し、その内容を確認して署名を検証する必要があります。 

多くのプラットフォームに JWT 解析ライブラリが用意されており、大部分のユーザーは JWT トークン用の安全で信頼性のある解析を実装しています。それでも、通常はトークンの特定の特性 (発行者、オーディエンスなど) に正しい値が含まれるようにするために、これらのライブラリを設定する必要があります。 トークンの解析では、トークンが確実に以下の要件を満たすように、解析ライブラリを設定するか、独自の検証を作成する必要があります。

1. トークンが HTTP `Authorization` ヘッダーに含まれ、"Bearer" スキームで送信された。
2. トークンは有効な JSON であり、[JWT 標準](http://openid.net/specs/draft-jones-json-web-token-07.html)に準拠している。
3. トークンに "issuer" 要求が含まれ、その値が `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` または `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` である  (両方の issuer 値を確認することで、セキュリティ プロトコル v3.1 と v3.2 両方の issuer 値を確実にチェックすることができます)。
4. トークンに "audience" 要求が含まれ、その値がボットの Microsoft AppID と同じである。
5. トークンに "appid" 要求が含まれ、その値がボットの Microsoft AppID と同じである。
6. トークンの有効期限が切れていない。 業界標準の clock-skew は 5 分です。
7. トークンに有効な暗号化署名があり、そのキーが、[手順 3](#emulator-to-bot-step-3) で取得した OpenID キー ドキュメント内の一覧に含まれている。

> [!NOTE]
> 要件 5 は、エミュレーター検証パスに固有です。 

トークンがこれらの要件を満たさない場合、ご利用のボットは、**HTTP 403 (アクセス不可)** 状態コードを返すことによって要求を終了する必要があります。

> [!IMPORTANT]
> これらの要件はすべて重要ですが、要件 4 と 7 は特に重要です。 これらの検証要件のすべてを実装しないと、ボットは攻撃に対して無防備になり、ボットから JWT トークンが漏洩するおそれがあります。

#### <a name="emulator-to-bot-example-jwt-components"></a>エミュレーターからボット: サンプル JWT コンポーネント

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 実際の場面では、フィールドは異なる可能性があります。 すべての JWT トークンを上記で指定されているように作成し、検証します。

## <a name="security-protocol-changes"></a>セキュリティ プロトコルの変更

> [!WARNING]
> セキュリティ プロトコル v3.0 に対するサポートは、**2017 年 7 月 31 日**で終了します。 独自の認証コード (お使いのボットの作成に Bot Builder SDK を使用していないもの) を作成してある場合は、以下に示す v3.1 の値を使用するようにアプリケーションを更新することにより、セキュリティ プロトコル v3.1 にアップグレードする必要があります。 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[ボットからコネクタへの認証](#bot-to-connector)

#### <a name="oauth-login-url"></a>OAuth ログイン URL

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth の範囲

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[コネクタからボットへの認証](#connector-to-bot)

#### <a name="openid-metadata-document"></a>OpenID メタデータ ドキュメント

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>JWT 発行者

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[エミュレーターからボットへの認証](#emulator-to-bot)

#### <a name="oauth-login-url"></a>OAuth ログイン URL

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth の範囲

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 |  ご利用のボットの Microsoft AppID + `/.default` |

#### <a name="jwt-audience"></a>JWT のオーディエンス

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | ご利用のボットの Microsoft AppID |

#### <a name="jwt-issuer"></a>JWT 発行者

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>OpenID メタデータ ドキュメント

| プロトコルのバージョン | 有効な値 |
|----|----|
| v3.1 および v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>その他のリソース

- [Bot Framework 認証のトラブルシューティング](../bot-service-troubleshoot-authentication-problems.md)
- [JSON Web トークン (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [JSON Web Signature (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [JSON Web Key (JWK) RFC 7517](https://tools.ietf.org/html/rfc7517)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
