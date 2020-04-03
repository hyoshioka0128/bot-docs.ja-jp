---
title: Azure Bot Service でのユーザー認証 - Bot Service
description: Azure Bot Service のユーザー認証機能について説明します。
keywords: Azure Bot Service, 認証, Bot Framework トークン サービス
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c6566a0b4d1150b3662734200c3ffd7c62dff0f6
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250041"
---
# <a name="bot-authentication"></a>ボット認証

ユーザーに代わって、セキュリティで保護されたオンライン リソースにボットがアクセスする必要がある場合があります。それには、ボットがそのための権限を持っている必要があります。 認証はベアラー トークンの形式をとります。 これは、以下で説明する **Azure Bot Service** アーキテクチャに含まれる一連のコンポーネントによって実現されます。

1. **ボット チャネル登録アプリケーション**。 これは、ユーザーがボットとのチャネルを介して通信できるようにするために、Azure インフラストラクチャ内でボットを "*統合*" するメカニズムです。
1. **ボット**。 ボットは、Azure を含む任意の場所でホストできます。
1. **ID プロバイダー アプリケーション**。 このアプリケーションは、ユーザーの代わりに ボットがアクセスする必要がある、セキュリティで保護されたリソースごとに必要です。 これは、 ボットが Office 365 MSGraph などの*セキュリティで保護された外部リソースにアクセス*することを可能にする、**ID プロバイダー**です。 Azure Active Directory は、Microsoft セキュリティで保護されたリソースにアクセスするための ID プロバイダーです。 GitHub など、セキュリティで保護されたリソースにアクセスするための ID プロバイダーは他にも多く存在します。

次の図は、認証用の ID プロバイダーとして Azure AD を使用する Azure Bot Service のアーキテクチャを示しています。

![Azure Bot Service のアーキテクチャ](media/concept-bot-authentication/azure-bot-service-architecture.png)

ご利用の OAuth ナレッジを更新する場合は、次を参照してください。

- [OAuth の優れた概要](https://aaronparecki.com/oauth-2-simplified/) 正式な仕様よりも簡単に理解できます
- [OAuth の仕様](https://oauth.net/2/)

## <a name="user-authentication-in-a-conversation"></a>会話でのユーザー認証

ボットがユーザーに代わって一定の操作 (メールの確認、カレンダーの参照、フライト状況の確認、発注など) を実行するには、Microsoft Graph や GitHub、組織の REST サービスなど、外部サービスを呼び出す必要があります。
各外部サービスには、これらの呼び出しをセキュリティで保護する方法があります。 これらの要求を発行する一般的な方法は、その外部サービスのユーザーを一意に識別する "*ユーザートークン*" を使用することです ([JSON Web トークン](https://jwt.io/introduction/) (JWT) とも呼ばれます)。

外部サービスの呼び出しをセキュリティ保護するには、ボットがそのサービスのユーザー トークンを取得できるように、ユーザーにサインインを要求する必要があります。
多くのサービスでは、**OAuth** または **OAuth2** プロトコルによるトークンの取得がサポートされています。

Azure Bot Service には、OAuth プロトコルと連携し、トークンのライフサイクルを管理する特別な**サインイン** カードおよびサービスが用意されています。 ボットは、これらの機能を使用してユーザー トークンを取得できます。

- ボット構成の一環として、**OAuth 接続**が Azure の Azure Bot Service リソース内に登録されます。

    この接続には、使用する **ID プロバイダー**の情報のほか、有効な OAuth クライアント ID およびシークレット、有効にする OAuth スコープ、その ID プロバイダーによって必要とされるその他の接続メタデータが含まれています。

- OAuth 接続は、ユーザーのサインインとユーザー トークンの取得を楽にするためにボットのコード内で使用されます。

次の画像は、認証プロセスに関連する要素を示しています。

![ボット認証コンポーネント](media/concept-bot-authentication/bot-auth-components.png)

## <a name="about-the-bot-framework-token-service"></a>Bot Framework トークン サービスについて

Bot Framework トークン サービスの役割は次のとおりです。

- 幅広い外部サービスとの OAuth プロトコルの使用を容易にすること。
- 特定のボット、チャネル、会話、ユーザーのトークンを安全に格納すること。
- ユーザー トークンを取得すること。
    > [!TIP]
    > ボットのユーザー トークンが期限切れになっている場合は、ボットが次のことを行います。
    >    - ユーザーをログアウトする
    >    - サインイン フローをもう一度開始する

たとえば、ユーザーの最新のメールを確認できるボットでは、Microsoft Graph API を使用して、**ID プロバイダー** (この場合は **Azure Active Directory**) のユーザー トークンを要求します。 デザイン時に、ボットの開発者は次の 2 つの重要な手順を実行します。

1. Azure Active Directory アプリケーション (ID プロバイダー) を、Azure Portal を使用して Bot Framework トークン サービスに登録します。
1. ボットの OAuth 接続を (たとえば `GraphConnection` という名前で) 構成します。

次の図は、Microsoft Graph サービスを使用して電子メール要求が行われた場合のユーザーとボットとの対話の時間シーケンスを示しています。

![ボット認証時間シーケンス](media/concept-bot-authentication/bot-auth-time-sequence.PNG)

1. ユーザーがボットに電子メール要求を行います。
1. このメッセージが含まれたアクティビティがユーザーから Bot Framework チャネル サービスに送信されます。 アクティビティ内の `userid` フィールドが設定されていることがチャネル サービスによって確認され、メッセージがボットに送信されます。

    > [!NOTE]
    > ユーザー ID はチャネル固有です (ユーザーの Facebook ID や SMS 電話番号など)。

1. ボットは Bot Framework トークン サービスに要求を実行し、OAuth 接続 `GraphConnection` のユーザー ID のトークンが既にあるかどうかを問い合わせます。
1. このユーザーがボットを操作したのはこれが初めてであるため、Bot Framework トークン サービスにはまだこのユーザー用のトークンがありません。*NotFound* が結果としてボットに返されます。

    > [!NOTE]
    > トークンが見つかった場合、認証手順はスキップされ、ボットは、格納されているトークンを使用して電子メール要求を行うことができます。

1. ボットによって、`GraphConnection` という接続名が含まれた OAuthCard が作成され、このカードを使用してサインインするようユーザーに要求する応答が返されます。
1. アクティビティが Bot Framework チャネル サービスを通過します。これにより、Bot Framework トークン サービスが呼び出され、この要求で有効な OAuth サインイン URL が作成されます。 このサインイン URL が OAuthCard に追加され、そのカードがユーザーに返されます。
1. OAuthCard のサインイン ボタンをクリックしてサインインするよう、ユーザーにメッセージが表示されます。
1. ユーザーがサインイン ボタンをクリックすると、チャネル サービスによって Web ブラウザーが開かれます。また、外部サービスが呼び出され、そのサインイン ページが読み込まれます。
1. ユーザーが外部サービスのこのページにサインインします。 次に、外部サービスが Bot Framework トークン サービスとの OAuth プロトコル交換を完了します。その結果、外部サービスが Bot Framework トークン サービスにユーザー トークンを送信します。 Bot Framework トークン サービスがこのトークンを安全に格納し、このトークンと共にアクティビティをボットに送信します。
1. ボットがトークン付きのアクティビティを受け取り、それを使用して MS Graph API に対して呼び出しを実行できるようになります。

## <a name="securing-the-sign-in-url"></a>サインイン URL のセキュリティ保護

Bot Framework でユーザー ログインを容易にする際に重要な考慮事項は、サインイン URL をセキュリティ保護する方法です。 ユーザーにサインイン URL が表示されるとき、その URL は、そのボットの特定の会話 ID とユーザー ID に関連付けられています。 この URL は共有しないようにする必要があります。特定のボットの会話で不適切なサインインが行われる原因となるためです。 サインイン URL の共有に関するセキュリティ攻撃を緩和するには、マシンとサインイン URL をクリックする人物が会話ウィンドウを "_所有_" していることを保証する必要があります。

Cortana、Microsoft Teams、Direct Line、WebChat など、いくつかのチャネルでは、ユーザーに通知することなくこれを行えます。 たとえば、WebChat では、サインイン フローが WebChat 会話と同じブラウザーで実行されるように、セッション Cookie を使用します。 しかし、他のチャネルでは多くの場合、ユーザーに 6 桁の "_マジック コード_" が表示されます。 これは組み込みの多要素認証に類似しています。Bot Framework トークン サービスは、ユーザーが最終的な認証を完了するまでトークンをボットにリリースしないためです。これにより、サインインした人物は、6 桁のコードを入力してチャット エクスペリエンスにアクセスすることになります。

> [!IMPORTANT]
> こちらの重要な[セキュリティに関する考慮事項](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#security-considerations)に留意してください。
> 追加情報は次のブログ記事で見つけることができます。[Azure Bot Service 認証でのWebChat の使用](https://blog.botframework.com/2018/09/01/using-webchat-with-azure-bot-services-authentication/)。


## <a name="azure-activity-directory-in-a-bot"></a>ボットの Azure Activity Directory

次に示すように、Azure にボットをデプロイすると、Azure Active Directory は非常に重要な役割を果たします。

### <a name="bot-registration"></a>ボットの登録

 たとえばボット チャネル登録などによってボットを Azure に登録すると、Active Directory 登録アプリケーションが作成されます。 このアプリケーションには、デプロイ用にボットを構成するために必要な、独自のアプリケーション ID (アプリ ID) とクライアント シークレット (パスワード) があります。 アプリ ID は、ボットと Bot Framework チャネル サービスの間のサービス間通信をセキュリティ保護するためにも必要です。

### <a name="bot-authentication"></a>ボット認証

Azure Active Directory は、 **OAuth2.0** などの業界標準プロトコルを使用して、ユーザーを安全にサインインさせることができるクラウドID プロバイダーです。 詳しくは、「[Azure Active Directory ID プロバイダー](bot-builder-concept-identity-providers.md#azure-active-directory-identity-provider)」をご覧ください。


### <a name="next-steps"></a>次のステップ

AD が果たす役割が理解できたところで、ボットを認証する方法を見てみましょう。

> [!div class="nextstepaction"]
> [ボットに認証を追加する](bot-builder-authentication.md)。

## <a name="see-also"></a>関連項目

- [ID プロバイダー](bot-builder-concept-identity-providers.md)
- [REST コネクタの認証](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-4.0)
- [REST Directline 認証](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)