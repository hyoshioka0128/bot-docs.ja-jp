---
ms.openlocfilehash: aa609e588ab1cf914a9a978228a879d3f3102e19
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80250231"
---

<!-- Azure AD v2 settings -->
<!-- Fixed ID -->

| **プロパティ** | **説明** | **Value** |
|---|---|---|
|**名前** | 接続の名前 | \<接続の名前\> <img width="300px">|
|**サービス プロバイダー**| Azure AD ID プロバイダー | `Azure Active Directory v2` |
|**クライアント ID** | Azure AD ID プロバイダー アプリの ID| \<AAD プロバイダー アプリの ID\> |
|**クライアント シークレット** | Azure AD ID プロバイダー アプリのシークレット| \<AAD プロバイダー アプリのシークレット\> |
|**テナント ID** | | \<ディレクトリ (テナント) ID\> または `common`。 注を参照 |
|**スコープ** |Azure AD ID プロバイダー アプリに付与した API アクセス許可のスペース区切りリスト| `openid` `profile` `Mail.Read` `Mail.Send` `User.Read` `User.ReadBasic.All` などの値 |
|**Token Exchange URL** (トークン交換 URL) |Azure AD v2 で SSO のために使用されます| |
| | |

**注**

- 次のいずれかを選択した場合は、AAD ID プロバイダー アプリ用に記録した**テナント ID** を入力します。

    - [*Accounts in this organizational directory only (Microsoft only - Single tenant)* ] (この組織ディレクトリのみに含まれるアカウント (Microsoft のみ - 単一テナント))

    - [*Accounts in any organizational directory (Microsoft AAD directory - Multi tenant)* ] (任意の組織ディレクトリ内のアカウント (Microsoft AAD ディレクトリ - マルチテナント))
- [*Accounts in any organizational directory (Any AAD directory - Multi tenant and personal Microsoft accounts e.g. Skype, Xbox, Outlook.com)* ] (任意の組織ディレクトリ内のアカウント (任意の AAD ディレクトリ - マルチテナント) と個人の Microsoft アカウント (Skype、Xbox、Outlook.com など)) を選択した場合、テナント ID の代わりに「`common`」を入力します。 それ以外の場合、AAD ID プロバイダー アプリは ID が選択されているテナントを通じて検証され、個人の MS アカウントは除外されます。
- [スコープ] はスペースで区切った値のリストであり、大文字と小文字が区別されます。
