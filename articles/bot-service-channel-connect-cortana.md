---
title: ボットを Cortana に接続する | Microsoft Docs
description: Cortana インターフェイスを介してアクセスできるようにボットを構成する方法について説明します。
keywords: cortana, ボット チャンネル, cortana の構成
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 04/30/2018
ms.openlocfilehash: caa7c71bc0b12ff6defb72f75dcb6d12ce512806
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167005"
---
# <a name="connect-a-bot-to-cortana"></a>ボットを Cortana に接続する

Cortana は、テキストによる会話だけでなく、音声メッセージも送受信できる音声認識対応チャンネルです。 Cortana に接続することを目的としたボットは、テキストだけでなく音声にも対応するように設計する必要があります。 Cortana "*スキル*" とは、Cortana クライアントを使用して呼び出すことができるボットです。 ボットを公開すると、利用可能なスキルの一覧にボットが追加されます。

Cortana チャンネルを追加するには、[Azure portal](https://portal.azure.com/) でボットを開き、 **[チャンネル]** ブレードをクリックして、 **[Cortana]** をクリックします。

![Cortana チャンネルを追加する](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>Cortana を構成する

ボットを Cortana チャンネルに接続すると、登録フォームにボットの基本情報が自動的に入力されます。 この情報を入念に確認してください。 このフォームは次のフィールドで構成されます。

| フィールド | 説明 |
|------|------|
| **[スキル] アイコン** | スキルが呼び出されたときに Cortana のキャンバスに表示されるアイコン。 これは、スキルを発見可能な場所 (Microsoft Store など) でも使用されます (最大 32 KB、PNG のみ)。|
| **[表示名]** | Cortana スキルの名前は、ビジュアル UI の上部に表示されます (最大 30 文字)。 |
| **呼び出し名** | スキルを呼び出すときにユーザーが呼びかける名前です。 3 つ以下の単語で構成され、発音が簡単なものにする必要があります。 この名前の選択方法の詳細については、「[Invocation Name Guidelines][invocation]」(呼び出し名ガイドライン) をご覧ください。|

![既定の設定](~/media/channels/cortana-defaultsettings.png)

>!注:Cortana は現在、Azure Active Directory (AAD) アカウント認証の使用をサポートしていません。 ボットを Cortana に正常に公開するには、Microsoft アカウント (MSA) を使用する必要があります。

## <a name="general-bot-information"></a>ボットの一般情報

**[Manage user identity through connected services]\(接続済みサービスによるユーザー ID の管理\)** セクションで、オプションをクリックして有効にします。 フォームに入力します。

アスタリスク (*) が付いているフィールドはすべて必須です。 ボットを Cortana に接続するには、事前に Azure に発行する必要があります。

![ユーザー ID の管理、パート 1](~/media/channels/cortana-manageidentity-1.png)
![ユーザー ID の管理、パート 2](~/media/channels/cortana-manageidentity-2.png)

### <a name="when-should-cortana-prompt-for-a-user-to-sign-in"></a>[When should Cortana prompt for a user to sign in]\(Cortana がユーザーにサインインを要求するタイミング\)

ユーザーがスキルを呼び出したときに Cortana でユーザーにサインインする場合は、 **[Sign in at invocation]\(呼び出し時にサインインする\)** を選択します。

Bot Service サインイン カードを使用してユーザーにサインインする場合は、 **[Sign in when required]\(必要に応じてサインインする\)** を選択します。 通常、このオプションは、認証が必要な機能を使用するときにのみユーザーにサインインする場合に使用します。 サインイン カードが添付ファイルとして含まれたメッセージがスキルから送信されると、Cortana はサインイン カードを無視し、アカウントの接続設定を使用して認可フローを実行します。

### <a name="account-name"></a>アカウント名

ユーザーがスキルにサインインするときに表示するスキルの名前。

### <a name="client-id-for-third-party-services"></a>[Client ID for third-party services]\(サード パーティ サービスのクライアント ID\)

ボットのアプリケーション ID。 ID はボットの登録時に取得済みです。

### <a name="space-separated-list-of-scopes"></a>[Space-separated list of scopes]\(スコープのスペース区切りリスト\)

サービスに必要なスコープを指定します (サービスのドキュメントを参照)。

### <a name="authorization-url"></a>[Authorization URL]\(承認 URL\)

`https://login.microsoftonline.com/common/oauth2/v2.0/authorize` を設定します。

### <a name="token-options"></a>[Token options]\(トークン オプション\)

**[POST]** を選択します。

### <a name="grant-type"></a>[付与タイプ]

コード付与フローを使用するには、 **[承認コード]** を選択します。暗黙的フローを使用するには、 **[暗黙]** を選択します。

### <a name="token-url"></a>[トークンURL]

**[承認コード]** 付与タイプの場合は、`https://login.microsoftonline.com/common/oauth2/v2.0/token` に設定します。

### <a name="client-secretpassword-for-third-party-services"></a>[Client secret/password for third party services]\(サード パーティ サービスのクライアント シークレット/パスワード\)

ボットのパスワード。 パスワードはボットの登録時に取得済みです。

### <a name="client-authentication-scheme"></a>[Client authentication scheme]\(クライアント認証方式\)

**[HTTP 基本]** を選択します。

### <a name="internet-access-required-to-authenticate-users"></a>[Internet access required to authenticate users]\(ユーザー認証にインターネット アクセスが必要\)

オフのままにしておきます。

### <a name="request-user-profile-data-optional"></a>[Request user profile data]\(ユーザー プロファイル データを要求する\) (省略可能)

Cortana では、さまざまな種類のユーザー プロファイル情報へのアクセスを提供します。この情報を使用して、特定のユーザー向けにボットをカスタマイズできます。 たとえば、スキルがユーザーの名前と場所にアクセスできる場合は、"Kamran さん、こんにちは。ワシントン州ベルビューで楽しい 1 日をお過ごしください" といったカスタム応答を使用できます。

**[Add a user profile request]\(ユーザー プロファイル要求の追加\)** をクリックし、ドロップダウン リストから必要なユーザー プロファイル情報を選択します。 ボットのコードからこの情報にアクセスする際に使用するフレンドリ名を追加します。

### <a name="deploy-on-cortana"></a>[Deploy on Cortana]\(Cortana にデプロイする\)

Cortana スキルの登録フォームへの入力が完了したら、 **[Deploy on Cortana]\(Cortana にデプロイする\)** をクリックして接続を完了します。 これにより、ボットの [チャンネル] ブレードに戻るので、ボットが Cortana に接続されていることを確認できます。

この時点で、ボットは Cortana スキルとしてお使いのアカウントにデプロイされています。

## <a name="next-steps"></a>次の手順

* [Cortana Skills Kit](https://aka.ms/CortanaSkillsKitOverview)
* [デバッグの有効化](bot-service-debug-cortana-skill.md)
* [Cortana スキルの公開][publish]

[invocation]: https://docs.microsoft.com/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/cortana/skills/publish-skill
[CortanaEntity]: https://aka.ms/cortana-channel-data
