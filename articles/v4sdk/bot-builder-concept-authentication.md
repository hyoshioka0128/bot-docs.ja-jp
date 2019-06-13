---
title: Azure Bot Service でのユーザー認証 |Microsoft Docs
description: Azure Bot Service でユーザー認証の機能について説明します。
keywords: azure bot service、認証、bot framework のトークン サービス
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6eea954f58096d89cd3278058146b93fa04f435f
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693762"
---
# <a name="user-authentication-within-a-conversation"></a>メッセージ交換内のユーザー認証

電子メールのチェック、予定表を参照して、フライトの状況でのチェックや、注文など、ユーザーに代わって特定の操作を実行するには、ボットは Microsoft Graph、GitHub、または会社の REST サービスなどの外部サービスを呼び出す必要があります。
各外部のサービスは、これらの呼び出しをセキュリティで保護する方法と、このような呼び出しをセキュリティで保護する一般的な方法を使用してそれらの要求を発行する、_ユーザー トークン_(JWT とも呼ばれます) の外部サービスでユーザーを一意に識別します。

セキュリティで保護された、ボットの外部サービスへの呼び出しはユーザーにサインインを要求する必要があります、ユーザーのサービスのトークンを獲得できるようにします。
多くのサービスでは、OAuth または OAuth2 プロトコルを使用してトークンの取得をサポートします。
Azure Bot Service は、特殊なサインインのカードと OAuth プロトコルを正常に機能し、トークン ライフ サイクルを管理サービスを提供し、ボットは、これらの機能を使用して、ユーザーのトークンを取得します。

- ボットの構成の一部として、 _OAuth 接続設定_が Azure での Azure Bot Service リソース内で登録されています。

    各接続設定には、外部サービスまたは有効な OAuth クライアント id とシークレット、OAuth スコープを有効にして、外部サービスまたは id で必要なその他の接続メタデータと共に、使用する id プロバイダーに関する情報が含まれています。プロバイダー。

- ボットのコードでは、OAuth 接続の設定はユーザーのサインインや、ユーザー トークンを取得するため使用されます。

サインインしているワークフローには、これらのサービスが関わります。

![認証の概要](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>Bot Framework のトークン サービスについて

Bot Framework のトークン サービスは責任を負います。

- さまざまな外部サービスの OAuth プロトコルの使用を促進します。
- 特定のボット、チャネル、メッセージ交換、およびユーザーのトークンを安全に格納します。
- トークン ライフ サイクル管理を含むトークンを更新しようとしています。

たとえば、Microsoft Graph API を使用して、ユーザーの最近の電子メールをチェックできるボットでは、Azure Active Directory のユーザー トークンが必要です。 デザイン時に、ボットの開発者は (Azure Portal) で、使用には、Bot Framework トークン サービスと Azure Active Directory アプリケーションを登録および OAuth の接続設定を構成 (という`GraphConnection`)、ボットの。 ユーザーがボットとの対話、ワークフローは次のようになります。

1. ユーザーは、ボット、「電子メールを確認してください」を確認します。
1. このメッセージのアクティビティは、Bot Framework チャネルのサービスに、ユーザーから送信されます。 チャネルのサービスにより、`userid`アクティビティ内のフィールドが設定されているし、ボットにメッセージを送信します。

    ユーザー ID、チャネルの特定など、ユーザーの facebook ID、または、SMS 電話番号。

1. ボットは、メッセージ アクティビティを受信し、ユーザーの電子メールをチェックするインテントであるかを決定します。 ボットはかどうか、既にトークンが OAuth 接続設定のユーザー Id の確認の Bot Framework のトークン サービスに対する要求を行う`GraphConnection`します。
1. これは、このユーザーがボットとの対話が初めてであるため、Bot Framework のトークン サービスは、このユーザーのトークンがないを返します、 _NotFound_ボットへの結果。
1. ボットの接続の名前を持つ、OAuthCard を作成します`GraphConnection`とこのカードを使用してサインインするかを確認するユーザーに返信します。
1. アクティビティは、Bot Framework Channel Service の有効な OAuth を作成するために Bot Framework のトークン サービスは、どの呼び出しでサインイン URL のこの要求を通過します。 このサインイン URL は、OAuthCard に追加され、カードがユーザーに返されます。
1. ユーザーには、OAuthCard のサインイン ボタンをクリックしてサインインするメッセージが表示されます。
1. ユーザーには、サインイン ボタンがクリックすると、チャネルのサービスは web ブラウザーを開きし、は、サインイン ページを読み込む外部サービスを呼び出し。
1. ユーザー サインインするとこのページに、外部のサービスです。 完了すると、外部のサービスは、Bot Framework のトークン サービス、その結果、Bot Framework のトークン サービスのユーザー トークンを送信する外部サービスとの交換が OAuth プロトコルを完了します。 安全に Bot Framework のトークン サービスは、このトークンを格納し、このトークンを使用してボットにアクティビティを送信します。
1. ボットは、トークンを使用してアクティビティを受信し、Graph API に対して呼び出しを行うために使用できます。

## <a name="securing-the-sign-in-url"></a>サインイン URL をセキュリティで保護します。

Bot Framework は、ユーザーのログインを容易に重要な要素は、サインイン URL をセキュリティで保護する方法を示します。 サインイン URL を使用して、ユーザーが表示されたら、この URL は特定のメッセージ交換 ID とそのボットのユーザー ID に関連付けられます。 ボットの特定のメッセージ交換が発生する不適切なサインインの原因となる、この URL は共有できません。 セキュリティ攻撃を軽減するために、サインイン URL を共有に関するコンピューターとサインイン URL をクリックした人が人であることを確認する必要がありますユーザー_を所有する_会話ウィンドウ。

Cortana、チーム、直線、および web チャットなどのいくつかのチャネルは、ユーザーのルーティング グループはこのことができます。 たとえば、web チャットはセッション cookie を使用して、サインイン フローによって web チャット会話と同じブラウザーで行われたことを確認します。 ただし、他のチャネルが、ユーザーには多くの場合が表示されます、6 桁_マジック コード_します。 これは、組み込みの多要素認証のようなユーザーが完了すると、最後の認証を入力してサインインしたユーザーがチャット エクスペリエンスへのアクセスを持つことを証明しない限りは、Bot Framework のトークン サービスに、ボットにトークンはリリースされません。6 桁のコード。

## <a name="azure-activity-directory-application-registration"></a>Azure Activity Directory アプリケーションの登録

Azure Bot Service が Azure のアクティビティ Directory (AD) アプリケーション ID を使用して登録されているすべてのボット 重要です**いない**でサインインするにはこのアプリケーション ID とパスワードを再利用するユーザー。 Azure Bot Service の Azure AD アプリケーション ID では、ボットと Bot Framework のチャネルのサービス間のサービス間通信をセキュリティで保護します。 サインインする場合、Azure AD にユーザーを個別に作成する必要があります、適切なアクセス許可スコープと Azure AD アプリケーションの登録。

## <a name="configure-an-oauth-connection-setting"></a>OAuth 接続の設定を構成します。

登録して、OAuth 接続設定を使用する方法の詳細について、次を参照してください。 [、ボットに追加の認証](bot-builder-authentication.md)します。
