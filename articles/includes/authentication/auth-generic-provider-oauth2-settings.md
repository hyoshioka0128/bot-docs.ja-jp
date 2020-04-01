---
ms.openlocfilehash: a0d633a8b03474acc09da7c739aeb2b6c4882286
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250221"
---
<!-- Oauth 2 generic provider settings -->
<!-- Fixed ID -->

| **プロパティ** | **説明** | **Value** |
|---|---|---|
|**名前** | 接続の名前 | \<接続の名前\> <img width="300px">|
| **サービス プロバイダー**| ID プロバイダー | ドロップダウン リストから、 **[Oauth 2 Generic Provider]\(Oauth 2 汎用プロバイダー\)** を選択します |
|**クライアント ID** | ID プロバイダーアプリの ID| \<プロバイダー ID\> |
|**クライアント シークレット** | ID プロバイダー アプリのシークレット| <プロバイダーのシークレット\> |
|*Scope List Delimiter* (スコープ リストの区切り記号)|スコープ値の間に使用する文字 (多くの場合、スペースまたはコンマ) | *,* \<コンマを入力\> |
|**Authorization URL Template** (承認 URL テンプレート) || https://login.microsoftonline.com/common/oauth2/v2.0/authorize |
|*Authorization URL Query String* (承認 URL クエリ文字列) |承認 URL に追加されるクエリ文字列。任意の必要なパラメーターでテンプレート化されます ({ClientId} {ClientSecret} {RedirectUrl} {Scopes} {State})| *?client_id={ClientId}&response_type=code&redirect_uri={RedirectUrl}&scope={Scopes}&state={State}* |
|**Token URL Template** (トークン URL テンプレート) | | https://login.microsoftonline.com/common/oauth2/v2.0/token |
|*Token URL Query String Template* (トークン URL クエリ文字列テンプレート) | トークン交換のために送信する本文 |*?* \<疑問符を入力\>|
|*Token Body Template* (トークン本文テンプレート) | トークン交換のために送信する本文 | *code={Code}&grant_type=authorization_code&redirect_uri={RedirectUrl}&client_id={ClientId}&client_secret={ClientSecret}* |
|**Refresh URL Template** (更新 URL テンプレート) | | https://login.microsoftonline.com/common/oauth2/v2.0/token |
|*Refresh URL Query String Template* (更新 URL クエリ文字列テンプレート) |更新 URL に追加されるクエリ文字列。任意の必要なパラメーターでテンプレート化されます ({ClientId} {ClientSecret} {RedirectUrl} {Scopes} {State}) |*?* \<疑問符を入力\>|
|*Refresh Body Template* (更新の本文テンプレート) | トークンの更新で送信する本文 | *refresh_token={RefreshToken}&redirect_uri={RedirectUrl}&grant_type=refresh_token&client_id={ClientId}&client_secret={ClientSecret}* |
|**スコープ** | 前に Azure AD 認証アプリに付与した API アクセス許可のコンマ区切りリスト | `openid` `profile` `Mail.Read` `Mail.Send` `User.Read` `User.ReadBasic.All` などの値|
