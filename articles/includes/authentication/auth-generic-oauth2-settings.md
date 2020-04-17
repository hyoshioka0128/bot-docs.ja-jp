---
ms.openlocfilehash: 5f37468f6c0bd61c54cd1eef66f432c6044978fe
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80250201"
---
<!-- Generic Oauth2 provider settings -->
<!-- Fixed ID -->

| **プロパティ** | **説明** | **Value** |
|---|---|---|
|**名前** | 接続の名前 | \<接続の名前\> <img width="300px">|
| **サービス プロバイダー**| ID プロバイダー | ドロップダウン リストから、 **[Generic Oauth 2]\(汎用 OAuth 2\)** を選択します |
|**クライアント ID** | ID プロバイダーアプリの ID| \<プロバイダー ID\> |
|**クライアント シークレット** | ID プロバイダー アプリのシークレット| <プロバイダーのシークレット\> |
|**Authorization URL** (承認 URL) | | https://login.microsoftonline.com/common/oauth2/v2.0/authorize |
|*Authorization URL Query String* (承認 URL クエリ文字列) | | *?client_id={ClientId}&response_type=code&redirect_uri={RedirectUrl}&scope={Scopes}&state={State}* |
|**Token URL** (トークン URL) | | https://login.microsoftonline.com/common/oauth2/v2.0/token |
|*Token Body* (トークン本文) | トークン交換のために送信する本文 | *code={Code}&grant_type=authorization_code&redirect_uri={RedirectUrl}&client_id={ClientId}&client_secret={ClientSecret}* |
|**Refresh URL** (更新 URL) | | https://login.microsoftonline.com/common/oauth2/v2.0/token |
|*Refresh Body Template* (更新の本文テンプレート) | トークンの更新で送信する本文 | *refresh_token={RefreshToken}&redirect_uri={RedirectUrl}&grant_type=refresh_token&client_id={ClientId}&client_secret={ClientSecret}* |
|**スコープ** | 前に Azure AD 認証アプリに付与した API アクセス許可のコンマ区切りリスト | `openid` `profile` `Mail.Read` `Mail.Send` `User.Read` `User.ReadBasic.All` などの値|
