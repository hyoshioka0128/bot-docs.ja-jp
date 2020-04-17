---
title: ボット リソースの管理 - Bot Service
description: ボット ファイルの目的と用途について説明します。
keywords: ボット ファイル, .bot, .bot ファイル, msbot, ボット リソース, ボット リソースの管理
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 65b7b3c3da9fb04d1d086c5304439b53bbc30cf7
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791144"
---
# <a name="manage-bot-resources"></a>ボット リソースの管理

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

通常、ボットでは [LUIS.ai](https://luis.ai)、[QnaMaker.ai](https://qnamaker.ai) などのさまざまなサービスが使用されます。 ボットを開発する場合、それらすべてを追跡できる必要があります。 appsettings.json、web.config、または .env などのさまざまな方法を使用できます。 

> [!IMPORTANT]
> Bot Framework SDK 4.3 リリースより前、Microsoft ではリソース管理のメカニズムとして .bot ファイルを提供していました。 ただし今後は、これらのリソースを管理するために appsettings.json または .env ファイルを使用することをお勧めします。 .bot ファイルを使用するボットは、.bot ファイルが " **_非推奨_** " になった今でも機能します。 リソースの管理に .bot ファイルを使用している場合は、設定の移行のために適用される手順に従ってください。 

## <a name="migrating-settings-from-bot-file"></a>.bot ファイルから設定を移行する
次のセクションでは、.bot ファイルから設定を移行する方法について説明します。 ご自身に該当するシナリオに従ってください。

**シナリオ 1: .bot ファイルのあるローカル ボット**

このシナリオでは、.bot ファイルを使用するローカル ボットがあるが、Azure portal に "_ボットが移行されています_"。 以下の手順に従って、.bot ファイルから appsettings.json または .env ファイルに設定を移行します。

- .bot ファイルが暗号化されている場合は、次のコマンドを使用して暗号化解除する必要があります。

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

- 暗号化解除された .bot ファイルを開き、値をコピーして appsettings.json または .env ファイルに追加します。
- appsettings.json または .env ファイルから設定を読み取るようにコードを更新します。

# <a name="c"></a>[C#](#tab/csharp)

`ConfigureServices` メソッドの場合、ASP.NET Core によって提供される次のような構成オブジェクトを使用します。 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascript"></a>[JavaScript](#tab/js)

JavaScript の場合、次のような `process.env` オブジェクトから .env 変数の参照を解除します。
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

"*必要に応じて*"、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してそれらをボットに接続します。

**シナリオ 2:.bot ファイルを使用して Azure にデプロイされたボット**

このシナリオでは、.bot ファイルを使用して Azure portal に既にボットをデプロイ済みです。次に、.bot ファイルから .appsettings.json または .env ファイルに設定を移行します。

- Azure portal からボット コードをダウンロードします。 コードをダウンロードするときに、MicrosoftAppId、MicrosoftAppPassword、およびその他の設定が指定される appsettings.json または .env ファイルを含めるよう求められます。 
- "_ダウンロードした_" appsettings.json または .env ファイルを開きそこから設定を "_ローカル_" の appsettings.json または .env ファイルにコピーします。 必ず botSecret および botFilePath エントリをローカルの appsettings.json または .env ファイルから削除してください。
- appsettings.json または .env ファイルから設定を読み取るようにコードを更新します。

# <a name="c"></a>[C#](#tab/csharp)
`ConfigureServices` メソッドの場合、ASP.NET Core によって提供される次のような構成オブジェクトを使用します。 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascript"></a>[JavaScript](#tab/js)
JavaScript の場合、次のような `process.env` オブジェクトから .env 変数の参照を解除します。
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

**Azure portal** の **[アプリケーション設定]** セクションから `botFilePath` および `botFileSecret` を削除する必要もあります。

"*必要に応じて*"、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してそれらをボットに接続します。

**シナリオ #3:appsettings.json または .env ファイルを使用するボットの場合**

このシナリオでは、SDK 4.3 を使用して一からボットの開発を開始する場合で、移行する既存の .bot ファイルがない場合ついて説明しています。 お使いのボットで使用するすべての設定は、以下に示されているように、appsettings.json または .env ファイル内にあります。

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="c"></a>[C#](#tab/csharp)

上記の設定を C# コードで読み取るには、ASP、NET Core によって提供される構成オブジェクトを使用します。**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascript"></a>[JavaScript](#tab/js)
JavaScript の場合、`process.env` オブジェクト (**index.js** など) から .env 変数の参照を解除します。
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

必要に応じて、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してそれらをボットに接続します。

## <a name="additional-resources"></a>その他のリソース
- ボットをデプロイする手順については、[デプロイ](../bot-builder-deploy-az-cli.md)に関するトピックをご覧ください。
- [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) を使用して、クラウド アプリケーションやサービスで使用される暗号化キーとシークレットをセキュリティで保護して管理する方法について説明します。
