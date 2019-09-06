---
title: 認証 |Microsoft Docs
description: Direct Line API v3.0 で API 要求を認証する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/22/2019
ms.openlocfilehash: 37e02a34e7b8ecc4d501ed7330b6f374548fd5a0
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076568"
---
# <a name="authentication"></a>Authentication

クライアントは、Direct Line API 3.0 に対する要求を、[Bot Framework Portal](../bot-service-channel-connect-directline.md) の Direct Line チャネル構成ページから取得する**シークレット**を使用するか、ランタイム時に取得する**トークン**を使用して認証できます。 シークレットまたはトークンは、次の書式で各要求の `Authorization` ヘッダー内に指定する必要があります。 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>シークレットとトークン

Direct Line **シークレット**は、関連付けられているボットに属するすべての会話にアクセスするために使用できるマスター キーです。 **シークレット**は、**トークン**を取得するために使用することもできます。 シークレットには有効期限がありません。 

Direct Line **トークン**は、1 つの会話にアクセスするために使用できるキーです。 トークンには有効期限がありますが、更新できます。

**シークレット** キーまたは**トークン**を使用するタイミングや、どちらを使用するかの決定は、セキュリティに関する考慮事項に基づいて行う必要があります。
シークレット キーの公開は、計画的に注意して行えば、許容される場合があります。 実際のところ、クライアントが正当であるかどうかを Direct Line で判断することを許可するため、これは既定の動作です。
ただし、一般に、ユーザー データを保持しようとしている場合は、セキュリティが問題になります。
詳細については、「[セキュリティに関する考慮事項](#security-considerations)」セクションを参照してください。

サービス間アプリケーションを作成する場合は、Direct Line API 要求の `Authorization` ヘッダー内に**シークレット**を指定することが最も簡単な方法です。 クライアントが Web ブラウザーまたはモバイル アプリで実行されるアプリケーションを記述する場合は、シークレットをトークン (1 つの会話でのみ機能し、更新されない限り有効期限が切れます) と交換し、その**トークン**を、Direct Line API 要求の `Authorization` ヘッダー内に指定できます。 自分にとって最適なセキュリティ モデルを選択してください。

> [!NOTE]
> Direct Line クライアント資格情報は、ボットの資格情報とは異なります。 これにより、キーを個別に変更でき、ボットのパスワードを公開せずにクライアント トークンを共有できます。 

## <a name="get-a-direct-line-secret"></a>Direct Line シークレットを取得する

<a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> で、ボット用の Direct Line チャネル構成ページを使用して、[Direct Line シークレットを取得](../bot-service-channel-connect-directline.md)できます。

![Direct Line 構成](../media/direct-line-configure.png)

## <a id="generate-token"></a> Direct Line トークンを生成する

1 つの会話にアクセスするために使用できる Direct Line トークンを生成するには、まず <a href="https://dev.botframework.com/" target="_blank">Bot Framework Portal</a> の Direct Line チャネル構成ページから Direct Line シークレットを取得します。 その後、次の要求を発行して、Direct Line シークレットを Direct Line トークンと交換します。

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

この要求の `Authorization` ヘッダーの **SECRET** を、Direct Line シークレットの値に置き換えます。

以下のスニペットは、トークン生成要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

要求ペイロードにはトークン パラメーターが含まれています。これは省略可能ですが、使用することをお勧めします。 Direct Line サービスに送り返すことができるトークンを生成するときに、次のペイロードを提供して、接続セキュリティを強化します。 これらの値を含めることによって、Direct Line によるユーザー ID と名前の追加セキュリティ検証が実行可能になり、悪意のあるクライアントによるこれらの値の改ざんが禁止されます。 これらの値を含めて、Direct Line の "_会話更新_" アクティビティ送信機能を向上させることもできます。これにより、ユーザーが会話に参加したときに直ちに会話更新を生成できるようになります。 この情報が指定されていない場合、ユーザーは、Direct Line が会話更新を送信する前に、コンテンツを送信する必要があります。

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| パラメーター | Type | 説明 |
| :--- | :--- | :--- |
| `user.id` | string | 省略可能。 トークン内でエンコードするためのチャネル固有のユーザー ID。 Direct Line ユーザーの場合、`dl_` で始まる必要があります。 会話ごとに一意のユーザー ID を作成できます。セキュリティを強化するために、この ID は推測できないものにします。 |
| `user.name` | string | 省略可能。 トークン内でエンコードするためのユーザーの表示用フレンドリ名。 |
| `trustedOrigins` | 文字列配列 | 省略可能。 トークン内に埋め込む信頼されたドメインの一覧。 これらはボットの Web チャット クライアントをホストできるドメインです。 これはご自身のボット用の Direct Line 構成ページの一覧と一致する必要があります。 |

### <a name="response"></a>Response

要求が成功した場合、応答には、1 つの会話で有効な `token` と、このトークンの有効期限が切れるまでの秒数を示す `expires_in` 値が含まれます。 トークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

### <a name="generate-token-versus-start-conversation"></a>トークンの生成と会話の開始

トークンの生成操作 (`POST /v3/directline/tokens/generate`) と[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)操作 (`POST /v3/directline/conversations`) は、どちらの操作も、1 つの会話にアクセスするために使用できる `token` を返すという点で類似しています。 ただし、会話の開始操作とは異なり、トークンの生成操作では、会話は開始されず、ボットへの接触は行われず、WebSocket のストリーミング URL は作成されません。 

トークンをクライアントに配布し、クライアントに会話を開始してほしい場合は、トークンの生成操作を使用します。 会話をすぐに開始するつもりの場合は、[会話の開始](bot-framework-rest-direct-line-3-0-start-conversation.md)操作を使用します。

## <a id="refresh-token"></a> Direct Line トークンを更新する

Direct Line トークンは、有効期限が切れていない限り、無制限に更新できます。 期限が切れたトークンは更新できません。 Direct Line トークンを更新するには、次の要求を発行します。 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

この要求の `Authorization` ヘッダーの **TOKEN_TO_BE_REFRESHED** を、更新する Direct Line トークンに置き換えます。

以下のスニペットは、トークン更新要求と応答の例を示しています。

### <a name="request"></a>Request

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

要求が成功した場合、応答には、前のトークンと同じ会話で有効な、新しい `token` と、新しいトークンの有効期限が切れるまでの秒数を示す `expires_in` 値が含まれます。 新しいトークンを有効のままにするには、有効期限が切れる前に[トークンを更新する](#refresh-token)必要があります。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="azure-bot-service-authentication"></a>Azure Bot Service 認証

このセクションに記載されている情報は、「[Azure Bot Service を介してボットに認証を追加する](../v4sdk/bot-builder-authentication.md)」の記事に基づいています。

**Azure Bot Service 認証**を使用すると、各種 ID プロバイダー (*Azure Active Directory*、*GitHub*、*Uber* など) に対してユーザーを認証し、これらから**アクセス トークン**を取得できます。 カスタム **OAuth2** ID プロバイダーの認証も構成できます。 これだけで、サポートされているすべての ID プロバイダーとチャネルで動作する **1 つの認証コード**を作成できます。 これらの機能を利用するには、次の手順を実行する必要があります。

1. ボットで、ID プロバイダーでのアプリケーション登録の詳細を含む `settings` を静的に構成します。
2. 前の手順で指定したアプリケーション情報に基づき、`OAuthCard` を使用して、ユーザーをサインインします。
3. **Azure Bot Service API** を使用してアクセス トークンを取得します。

### <a name="security-considerations"></a>セキュリティに関する考慮事項

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

[Web チャット](../bot-service-channel-connect-webchat.md)で *Azure Bot Service 認証*を使用する場合、注意する必要がある重要なセキュリティの考慮事項がいくつかあります。

1. **偽装**。 ここでの偽装とは、攻撃者がボットに自分を別人であると思い込ませることです。 Web チャットでは、攻撃者が自分の Web チャット インスタンスの**ユーザー ID を変えて**、他の誰かになりすます可能性があります。 これを防ぐために、推奨事項として、ボット開発者は**ユーザー ID を推測できないようにする**必要があります。 **強化された認証**オプションを有効にすると、Azure Bot Service ではすべてのユーザー ID の変更を検出して、拒否できます。 この場合、Direct Line からボットへのメッセージのユーザー ID (`Activity.From.Id`) は、Web チャットを初期化したときに使用したものと必ず同じになります。 この機能では、ユーザー ID は `dl_` で始まる必要があることに注意してください。
1. **ユーザー ID**。 次の 2 つのユーザー ID を取り扱う点に注意する必要があります。

    1. チャネルのユーザーの ID。
    1. ボットがやり取りする ID プロバイダーのユーザーの ID。
  
    ボットで、チャネルのユーザー A に ID プロバイダー P にサインインするよう要求する場合、サインイン プロセスでは、P にサインインするのはユーザー A であることを保証する必要があります。別のユーザー B がサインインを許可される場合、ユーザー A は、ボットを通じてユーザー B のリソースにアクセスできるようになります。 Web チャットでは、次に説明するように、確実に適切なユーザーがサインインするために 2 つのメカニズムが用意されています。

    1. 以前は、サインインの最後に、ランダムに生成された 6 桁のコード (別名マジック コード) がユーザーに表示されました。 ユーザーは、サインイン プロセスを完了するために、サインインを開始したやり取りにおいて、このコードを入力する必要があります。 このメカニズムでは、ユーザー エクスペリエンスが悪化する傾向があります。 また、フィッシング攻撃を受けやすくなります。 悪意のあるユーザーが、別のユーザーになりすましてサインインし、フィッシング詐欺を通じてマジック コードを入手できます。

    2. 前の方法で問題が発生したため、Azure Bot Service では、マジック コードが不要になりました。 Azure Bot Service では、サインイン プロセスは必ず Web チャット自体と**同じブラウザー セッション**でのみ完了します。 
    ボット開発者がこの保護を有効にするには、**ボットの Web チャット クライアントをホストできる信頼されたドメインの一覧**を含む **Direct Line トークン**を使用して、Web チャットを開始する必要があります。 以前は、ドキュメントに記載されていないオプション パラメーターを Direct Line トークン API に渡すことでのみ、このトークンを取得できました。 現在は、強化された認証オプションを使用して、Direct Line 構成ページで信頼されたドメイン (origin) の一覧を静的に指定できます。

    「[Azure Bot Service を介してボットに認証を追加する](../v4sdk/bot-builder-authentication.md)」も参照してください。

### <a name="code-examples"></a>コード例

次の .NET コントローラーは、強化された認証オプションが有効な状態で動作し、Direct Line トークンとユーザー ID を返します。

```csharp
public class HomeController : Controller
{
    public async Task<ActionResult> Index()
    {
        var secret = GetSecret();

        HttpClient client = new HttpClient();

        HttpRequestMessage request = new HttpRequestMessage(
            HttpMethod.Post,
            $"https://directline.botframework.com/v3/directline/tokens/generate");

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", secret);

        var userId = $"dl_{Guid.NewGuid()}";

        request.Content = new StringContent(
            JsonConvert.SerializeObject(
                new { User = new { Id = userId } }),
                Encoding.UTF8,
                "application/json");

        var response = await client.SendAsync(request);
        string token = String.Empty;

        if (response.IsSuccessStatusCode)
        {
            var body = await response.Content.ReadAsStringAsync();
            token = JsonConvert.DeserializeObject<DirectLineToken>(body).token;
        }

        var config = new ChatConfig()
        {
            Token = token,
            UserId = userId  
        };

        return View(config);
    }
}

public class DirectLineToken
{
    public string conversationId { get; set; }
    public string token { get; set; }
    public int expires_in { get; set; }
}
public class ChatConfig
{
    public string Token { get; set; }
    public string UserId { get; set; }
}

```

次の JavaScript コントローラーは、強化された認証オプションが有効な状態で動作し、Direct Line トークンとユーザー ID を返します。

```javascript
var router = express.Router(); // get an instance of the express Router

// Get a directline configuration (accessed at GET /api/config)
const userId = "dl_" + createUniqueId();

router.get('/config', function(req, res) {
    const options = {
        method: 'POST',
        uri: 'https://directline.botframework.com/v3/directline/tokens/generate',
        headers: {
            'Authorization': 'Bearer ' + secret
        },
        json: {
            User: { Id: userId }
        }
    };

    request.post(options, (error, response, body) => {
        if (!error && response.statusCode < 300) {
            res.json({ 
                    token: body.token,
                    userId: userId
                });
        }
        else {
            res.status(500).send('Call to retrieve token from Direct Line failed');
        } 
    });
});

```

## <a name="additional-resources"></a>その他のリソース

- [主要な概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [ボットを Direct Line に接続する](../bot-service-channel-connect-directline.md)
- [Azure Bot Service を介してボットに認証を追加する](../bot-builder-tutorial-authentication.md)
