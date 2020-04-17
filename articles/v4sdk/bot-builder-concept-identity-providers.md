---
title: Azure Bot Service の ID プロバイダー - Bot Service
description: Azure Bot Service の ID プロバイダーについて説明します。
keywords: Azure Bot Service, 認証, Bot Framework トークン サービス
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/11/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 966e484e4179c8256522c51b7873c1cb106e47a0
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80250191"
---
# <a name="identity-providers"></a>ID プロバイダー

ID プロバイダーは、ユーザー ID またはクライアント ID を認証し、使用可能なセキュリティ トークンを発行します。 ユーザー認証をサービスとして提供します。

Web アプリケーションなどのクライアント アプリケーションは、信頼できる ID プロバイダーに認証を委任します。 このようなクライアント アプリケーションは、フェデレーションされていると言い、フェデレーション ID を使用します。

信頼できる ID プロバイダーを使用すると:

- シングル サインオン (SSO) 機能が有効になり、アプリケーションはセキュリティで保護された複数のリソースにアクセスできるようになります。
- クラウド コンピューティング リソースとユーザーの間の接続が容易になり、ユーザーを再認証する必要性が少なくなります。

## <a name="single-sign-on"></a>シングル サインオン

シングル サインオンとは、ユーザーが 1 つの資格情報セットを使用してシステムに 1 回ログオンすることで、複数のアプリケーションまたはサービスにアクセスすることを許可する認証プロセスを指します。

ユーザーは、1 つの ID とパスワードを使用してログインし、複数の関連するソフトウェア システムへのアクセス権を取得します。 詳細については、「[シングル サインオン](./bot-builder-concept-sso.md)」を参照してください。

多くの ID プロバイダーは、ユーザー トークンを失効させ、関連付けられているアプリケーションやサービスへのアクセスを終了するサインアウト操作をサポートしています。


> [!IMPORTANT]
> SSO により、ユーザーが資格情報を入力しなければならない回数を減らすことで、使いやすさが向上します。 また、潜在的な攻撃対象領域を減らすことで、セキュリティが強化されます。

## <a name="azure-active-directory-identity-provider"></a>Azure Active Directory の ID プロバイダー

Azure Active Directory (AD) は、Microsoft Azure の ID サービスで、ID 管理とアクセス制御の機能を提供します。 **OAuth2.0** などの業界標準プロトコルを使用して、ユーザーを安全にサインインさせることができます。

次に示すように、異なる設定を持つ 2 つの AD ID プロバイダーの実装から選択できます。

> [!Note]
> ここで説明する設定は、Azure ボット登録アプリケーションで **OAuth 接続設定**を構成するときに使用します。 「[ボットに認証を追加する](bot-builder-authentication.md)」を参照してください。

# <a name="azure-ad-v1"></a>[Azure AD v1](#tab/adv1)

### <a name="azure-ad-v1"></a>Azure AD v1

ここに示す設定を使用して、Azure AD 開発者プラットフォーム (v1.0) ( **Azure AD v1** エンドポイントとも呼ばれる) を構成します。これにより、Microsoft の職場または学校アカウントを使用してユーザーを安全にサインインさせるアプリを構築できます。
詳細については、「[開発者向け Azure Active Directory (v1.0) の概要](https://docs.microsoft.com/azure/active-directory/azuread-dev/v1-overview)」を参照してください。

[!INCLUDE [azure-ad-v1-settings](~/includes/authentication/auth-aad-v1-settings.md)]

# <a name="azure-ad-v2"></a>[Azure AD v2](#tab/adv2)

### <a name="azure-ad-v2"></a>Azure AD v2

ここに示す設定を使用して、Azure AD プラットフォーム (v1.0) の進化版である、Microsoft ID プラットフォーム (v2.0) ( **Azure AD v2** エンドポイントとも呼ばれる) を構成します。  これにより、ボットは、Microsoft Graph などの Microsoft API またはカスタム API を呼び出すためにトークンを取得できます。 
詳細については、「 [Microsoft ID プラットフォーム (v2.0) の概要](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)」を参照してください。

AD v2v の設定により、ボットは、Microsoft Graph API を介して Office 365 データにアクセスでるようになります。

[!INCLUDE [azure-ad-v2-settings](~/includes/authentication/auth-aad-v2-settings.md)]

---

関連項目

- [Microsoft ID プラットフォーム (v2.0) に更新する理由](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)

- [Microsoft ID プラットフォーム (旧称: 開発者向け Azure Active Directory)](https://docs.microsoft.com/azure/active-directory/develop/)

## <a name="other-identity-providers"></a>他の ID プロバイダー

Azure では、複数の ID プロバイダーがサポートされています。 次の Azure コンソール コマンドを実行して、関連する詳細情報と共に完全な一覧を取得できます。

```cmd
az bot authsetting list-providers
```

また、ボット登録アプリの OAuth 接続設定を定義するときに、[Azure portal](https://ms.portal.azure.com/) でこれらのプロバイダーの一覧を表示できます。

![Azure ID プロバイダー](media/concept-bot-authentication/bot-auth-identity-providers.png)


### <a name="oauth-generic-providers"></a>OAuth 汎用プロバイダー

Azure では、独自の ID プロバイダーを使用できる汎用 OAuth2 がサポートされています。

以下に示すように、異なる設定を持つ 2 つの汎用 ID プロバイダーの実装から選択できます。

> [!Note]
> ここで説明する設定は、Azure ボット登録アプリケーションで **OAuth 接続設定**を構成するときに使用します。


# <a name="generic-oauth-2"></a>[汎用 OAuth 2](#tab/ga2)

### <a name="generic-oauth-2"></a>汎用 OAuth 2

このプロバイダーを使用して、Azure AD プロバイダー (特に AD v2) と同様の想定を持つ汎用 OAuth2 ID プロバイダーを構成します。 クエリ文字列と要求本文のペイロードが固定されているため、プロパティの数は限られています。 入力値について、さまざまな URI、クエリー文字列、本文に対するパラメーターがどのように中かっこ {} に入れらるかを確認できます。

[!INCLUDE [generic-oauth2-settings](~/includes/authentication/auth-generic-oauth2-settings.md)]


# <a name="oauth-2-generic-provider"></a>[OAuth 2 汎用プロバイダー](#tab/a2gp)

### <a name="oauth-2-generic-provider"></a>OAuth 2 汎用プロバイダー

より高い柔軟性が求められる場合は、このプロバイダーを使用して、汎用 OAuth 2 サービス プロバイダーを構成します。 より多くの構成パラメーターを必要とします。 この構成では、承認、更新、およびトークン変換用の URL テンプレート、クエリー文字列テンプレート、および本文テンプレートを指定します。 入力値について、さまざまな URI、クエリー文字列、本文に対するパラメーターがどのように中かっこ {} に入れらるかを確認できます。

[!INCLUDE [generic-provider-oauth2-settings](~/includes/authentication/auth-generic-provider-oauth2-settings.md)]

---
