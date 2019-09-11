---
title: Azure Bot Service でのユーザー認証 | Microsoft Docs
description: Azure Bot Service のユーザー認証機能について説明します。
keywords: Azure Bot Service, 認証, Bot Framework トークン サービス
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b4b71f7679ad6eea283437acb5a8293855219cf2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299507"
---
# <a name="user-authentication-within-a-conversation"></a>会話内のユーザー認証

ボットがユーザーに代わって一定の操作 (メールの確認、カレンダーの参照、フライト状況の確認、発注など) を実行するには、Microsoft Graph や GitHub、組織の REST サービスなど、外部サービスを呼び出す必要があります。
外部サービスにはそれぞれ、それらの呼び出しをセキュリティ保護する方法が備わっています。そのような呼び出しをセキュリティ保護する際に一般的なのは、その外部サービスでユーザーを一意に識別する "_ユーザー トークン_" (JWT とも呼ばれる) を使用して、それらの要求を発行する方法です。

外部サービスの呼び出しをセキュリティ保護するには、ボットがそのサービスのユーザー トークンを取得できるように、ユーザーにサインインを要求する必要があります。
多くのサービスでは、OAuth または OAuth2 プロトコルによるトークンの取得がサポートされています。
Azure Bot Service には、OAuth プロトコルと連携し、トークン ライフサイクルを管理する特別なサインイン カードおよびサービスが用意されています。ボットでは、これらの機能を使用してユーザー トークンを取得できます。

- ボット構成の一環として、"_OAuth 接続設定_" が Azure の Azure Bot Service リソース内に登録されます。

    各接続設定には、使用される外部サービスまたは ID プロバイダーの情報のほか、有効な OAuth クライアント ID およびシークレット、有効にする OAuth スコープ、その外部サービスまたは ID プロバイダーによって必要とされるその他の接続メタデータが含まれています。

- OAuth 接続設定は、ユーザーのサインインとユーザー トークンの取得を楽にするためにボットのコード内で使用されます。

これらのサービスはサインイン ワークフローに含まれます。

![認証の概要](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>Bot Framework トークン サービスについて

Bot Framework トークン サービスの役割は次のとおりです。

- 幅広い外部サービスとの OAuth プロトコルの使用を容易にすること。
- 特定のボット、チャネル、会話、ユーザーのトークンを安全に格納すること。
- トークンの更新の試行など、トークンのライフサイクルを管理すること。

たとえば、ユーザーの最新のメールを確認できるボットでは、Microsoft Graph API を使用して、Azure Active Directory のユーザー トークンを取得します。 ボットの開発者は、デザイン時に (Azure portal を使用して) Azure Active Directory アプリケーションを Bot Framework トークン サービスに登録してから、ボットの OAuth 接続設定 (名前は `GraphConnection`) を構成します。 ユーザーがボットを操作する際のワークフローは次のとおりです。

1. ユーザーがボットに "私のメールを確認してください" と指示します。
1. このメッセージが含まれたアクティビティがユーザーから Bot Framework チャネル サービスに送信されます。 アクティビティ内の `userid` フィールドが設定されていることがチャネル サービスによって確認され、メッセージがボットに送信されます。

    ユーザー ID はチャネル固有です (ユーザーの Facebook ID や SMS 電話番号など)。

1. メッセージ アクティビティがボットに届き、意図がユーザーのメールを確認することであると特定されます。 ボットは Bot Framework トークン サービスに要求を実行し、OAuth 接続設定 `GraphConnection` のユーザー ID のトークンが既にあるかどうかを問い合わせます。
1. このユーザーがボットを操作したのはこれが初めてであるため、Bot Framework トークン サービスにはまだこのユーザー用のトークンがありません。_NotFound_ が結果としてボットに返されます。
1. ボットによって、`GraphConnection` という接続名が含まれた OAuthCard が作成され、このカードを使用してサインインするようユーザーに要求する応答が返されます。
1. アクティビティが Bot Framework チャネル サービスを通過します。これにより、Bot Framework トークン サービスが呼び出され、この要求で有効な OAuth サインイン URL が作成されます。 このサインイン URL が OAuthCard に追加され、そのカードがユーザーに返されます。
1. OAuthCard のサインイン ボタンをクリックしてサインインするよう、ユーザーにメッセージが表示されます。
1. ユーザーがサインイン ボタンをクリックすると、チャネル サービスによって Web ブラウザーが開かれます。また、外部サービスが呼び出され、そのサインイン ページが読み込まれます。
1. ユーザーが外部サービスのこのページにサインインします。 それが終わると、外部サービスが Bot Framework トークン サービスとの OAuth プロトコル交換を完了します。その結果、外部サービスが Bot Framework トークン サービスにユーザー トークンを送信します。 Bot Framework トークン サービスがこのトークンを安全に格納し、このトークンと共にアクティビティをボットに送信します。
1. ボットがトークン付きのアクティビティを受け取り、それを使用して Graph API に対して呼び出しを実行できるようになります。

## <a name="securing-the-sign-in-url"></a>サインイン URL のセキュリティ保護

Bot Framework でユーザー ログインを容易にする際に重要な考慮事項は、サインイン URL をセキュリティ保護する方法です。 ユーザーにサインイン URL が表示されるとき、その URL は、そのボットの特定の会話 ID とユーザー ID に関連付けられています。 この URL は共有しないようにする必要があります。特定のボットの会話で不適切なサインインが行われる原因となるためです。 サインイン URL の共有に関するセキュリティ攻撃を緩和するには、マシンとサインイン URL をクリックする人物が会話ウィンドウを "_所有_" していることを保証する必要があります。

Cortana、Teams、Direct Line、WebChat など、いくつかのチャネルでは、ユーザーに通知することなくこれを行えます。 たとえば、WebChat では、サインイン フローが WebChat 会話と同じブラウザーで実行されるように、セッション Cookie を使用します。 しかし、他のチャネルでは多くの場合、ユーザーに 6 桁の "_マジック コード_" が表示されます。 これは組み込みの多要素認証に類似しています。Bot Framework トークン サービスは、ユーザーが最終的な認証を完了するまでトークンをボットにリリースしないためです。これにより、サインインした人物は、6 桁のコードを入力してチャット エクスペリエンスにアクセスすることになります。

## <a name="azure-activity-directory-application-registration"></a>Azure Activity Directory アプリケーションの登録

Azure のボット サービスとして登録されたすべてのボットでは、Azure Activity Directory (AD) アプリケーション ID が使用されます。 このアプリケーション ID とパスワードをユーザーのサインインに再利用**しない**ことが重要です。 Azure Bot Service の Azure AD アプリケーション ID は、ボットと Bot Framework チャネル サービスの間のサービス間通信をセキュリティ保護するためのものです。 ユーザーを Azure AD にサインインさせたい場合は、適切なアクセス許可とスコープを備えた別個の Azure AD アプリケーション登録を作成する必要があります。

## <a name="configure-an-oauth-connection-setting"></a>OAuth の接続設定の構成

OAuth 接続設定を登録して使用する方法の詳細については、[ボットへの認証の追加](bot-builder-authentication.md)に関するページを参照してください。
