---
ms.openlocfilehash: 2413184fe88dcf5560599be63b4ee85667f680bd
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250211"
---
<!-- Azure AD v1 settings -->
<!-- Fixed ID -->
| **プロパティ** | **説明** | **Value** |
|---|---|---|
|**名前** | 接続の名前 | \<接続の名前\> <img width="300px">|
|**サービス プロバイダー**| Azure AD ID プロバイダー | `Azure Active Directory` |
|**クライアント ID** | Azure AD ID プロバイダー アプリの ID| \<AAD プロバイダー アプリの ID\> |
|**クライアント シークレット** | Azure AD ID プロバイダー アプリのシークレット| \<AAD プロバイダー アプリのシークレット\> |
|**付与タイプ** | | `authorization_code` |
|**ログイン URL** | | `https://login.microsoftonline.com` |
|**テナント ID** | | <ディレクトリ (テナント) ID> または `common`。 「注」を参照してください。|
|**リソース URL** | | `https://graph.microsoft.com/` |
|**スコープ** | | <leave it blank> |
|**Token Exchange URL** (トークン交換 URL) |Azure AD v2 で SSO のために使用されます| |
| | |


**注**

- 次のいずれかを選択した場合は、AAD ID プロバイダー アプリ用に記録した**テナント ID** を入力します。

    - [*Accounts in this organizational directory only (Microsoft only - Single tenant)* ] (この組織ディレクトリのみに含まれるアカウント (Microsoft のみ - 単一テナント))

    - [*Accounts in any organizational directory (Microsoft AAD directory - Multi tenant)* ] (任意の組織ディレクトリ内のアカウント (Microsoft AAD ディレクトリ - マルチテナント))
- [*Accounts in any organizational directory (Any AAD directory - Multi tenant and personal Microsoft accounts e.g. Skype, Xbox, Outlook.com)* ] (任意の組織ディレクトリ内のアカウント (任意の AAD ディレクトリ - マルチテナント) と個人の Microsoft アカウント (Skype、Xbox、Outlook.com など)) を選択した場合、テナント ID の代わりに「`common`」を入力します。 それ以外の場合、AAD ID プロバイダー アプリは ID が選択されているテナントを通じて検証され、個人の MS アカウントは除外されます。
