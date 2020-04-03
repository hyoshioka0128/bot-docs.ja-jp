---
title: Azure Bot Service でのシングル サインオン - Bot Service
description: Azure Bot Service でのシングルサインオンについて説明します。
keywords: Azure Bot Service, 認証, Bot Framework トークン サービス
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 02b5e318699a31ffcf6b42befda2a4bb88c8352c
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250181"
---
# <a name="single-sign-on"></a>シングル サインオン

シングル サインオン (SSO) を使用すると、仮想アシスタントや WebChat などのクライアントが、ユーザーに代わってボットやスキルと通信できるようになります。
現時点では、[Azure AD v2](./bot-builder-concept-identity-providers.md#azure-active-directory-identity-provider) ID プロバイダーのみがサポートされています。

SSO は次のようなシナリオに適用されます。

- 仮想アシスタントと 1 つ以上のスキル ボット。 ユーザーは仮想アシスタントに 1 回サインインできます。 その後、アシスタントはユーザーに代わって複数のスキルを呼び出します。 [仮想アシスタント](./bot-builder-virtual-assistant-introduction.md)に関する記事も参照してください。
- Web サイトに埋め込まれた WebChat。 ユーザーは Web サイトにサインインします。 その後、Web サイトはユーザーに代わってボットまたはスキルを呼び出します。

SSO には次の利点があります。

- ユーザーは、仮想アシスタントまたは Web サイトに既にサインインしている場合、再度ログインする必要はありません。
- 仮想アシスタントまたは Web サイトは、ユーザーのアクセス許可に関する情報を持ちません。

> [!NOTE]
> SSO は Bot Framework SDK v4.8 の新機能です。

## <a name="sso-components-interaction"></a>SSO コンポーネントの相互作用

次の時系列の図は、SSO のさまざまなコンポーネント間の相互作用を示しています。


- 次の図は、仮想アシスタント クライアントを使用する場合の通常のフローを示しています。

    ![仮想アシスタントでのボットの SSO](media/concept-bot-authentication/bot-auth-sso-va-time-sequence.PNG)


- WebChat クライアントを使用する場合の通常およびフォールバックのフローを次に示します。

    ![WebChat でのボットの SSO](media/concept-bot-authentication/bot-auth-sso-webchat-time-sequence.PNG)

    エラーが発生した場合、SSO は OAuth カードを表示する既存の動作にフォールバックします。
    エラーは、たとえば、ユーザーの同意が必要な場合やトークンの交換が失敗した場合などに発生することがあります。

フローを分析してみましょう。

1. クライアントは、ボットとの会話を開始して OAuth シナリオをトリガーします。
1. ボットは OAuth カードをクライアントに送り返します。
1. クライアントは、OAuth カードをユーザーに表示する前にインターセプトし、`TokenExchangeResource` プロパティが含まれているかどうかを確認します。
1. このプロパティが存在する場合、クライアントはボットに `TokenExchangeInvokeRequest` を送信します。 クライアントは、ユーザーの交換可能なトークンを持っている必要があります。これは Azure AD v2 トークンである必要があり、そのオーディエンスは `TokenExchangeResource.Uri` プロパティと同じである必要があります。 <!-- For an example on how to get the user's exchangeable token, please refer to this [Webchat Sample (TBD)](https://linkrequired). --> クライアントは、次に示す本文を持つ呼び出しアクティビティをボットに送信します。

    ```json
    {
        "type": "Invoke",
        "name": "signin/tokenExchange",
        "value": {
            "id": "<any unique Id>",
            "connectionName": "<connection Name on the skill bot (from the OAuth Card)>",
            "token": "<exchangeable token>"
        }
    }
    ```

1. ボットは `TokenExchangeInvokeRequest` を処理し、クライアントに `TokenExchangeInvokeResponse` を返します。 クライアントは、`TokenExchangeInvokeResponse` を受け取るまで待つ必要があります。

    ```json
    {
        "status": "<response code>",
        "body": {
            "id":"<unique Id>",
            "connectionName": "<connection Name on the skill bot (from the OAuth Card)>",
            "failureDetail": "<failure reason if status code is not 200, null otherwise>"
        }
    }
    ```

1. `TokenExchangeInvokeResponse` の `status` が `200` の場合、クライアントは OAuth カードを表示しません。 "*通常のフロー*" 図を参照してください。 他の `status` であるか、`TokenExchangeInvokeResponse` を受け取っていない場合、クライアントは OAuth カードをユーザーに表示します。 "*フォールバックのフロー*" 図を参照してください。 これにより、エラーが発生した場合、満たされなかっ依存関係 (ユーザーの同意など) がある場合に、SSO フローが通常の OAuthCard フローにフォールバックします。

<!--
This section belongs to a how to (sample) article (TBD).

## Create Azure AD applications

Currently SSO in botframework is only supported for aadV2 apps.
We need to create 2 applications - one for the client and one for the Bot.
Depending on the scenario, the client may be webchat or a virtual assistant.
The general case for a Bot would be a skill Bot.

## Client Azure AD app

The client AAD application will be used to create an exchangeable token that will be passed onto the bot.
For an example of how to create an AAD app, look at the [bot builder authentication docs](https://docs.microsoft.com/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=csharp#create-your-azure-ad-application).

## Service Azure AD app

1) Follow the steps on [Create your Azure AD application](https://docs.microsoft.com/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=csharp#create-your-azure-ad-application).
2) In the **Expose an api** panel, click **Add a scope**. Fill in the fields
    - Click the **Add scope button**.
    - Click the **Add a client application** button, and enter the app Id for the client AAD app. Select the Scope that you created in the previous step. This ensures that the user will not be asked to consent when the client tries to get an exchangeable token for this app's scope
3) In the **Manifest** panel, set the `accessTokenAcceptedVersion` key to be `2`.

## Service Auth Connection

Remove these links and add back to how to page after sample is posted to botbuilder-samples experimental folder

1) Follow the directions in the [bot builder authentication doc](https://docs.microsoft.com/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=csharp#azure-ad-v2)
2) In the **Expose an api** panel, copy the scope that you added earlier. Fill it in the **Token Exchange Uri** field.
3) Save the connection setting. -->