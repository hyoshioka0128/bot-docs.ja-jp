---
title: .bot ファイルでリソースを管理する | Microsoft Docs
description: ボット ファイルの目的と用途について説明します。
keywords: ボット ファイル, .ボット, ボット ファイル, ボット リソース, ボット リソースの管理
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/30/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14552c55da4b1f9b581b81917496de179e92762b
ms.sourcegitcommit: 8d8b197c3f30593c4a5e4a6395ba5eff60dbd740
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/02/2019
ms.locfileid: "58811504"
---
# <a name="manage-bot-resources"></a>ボット リソースの管理

通常、ボットでは [LUIS.ai](https://luis.ai)、[QnaMaker.ai](https://qnamaker.ai) などのさまざまなサービスが使用されます。 ボットを開発する場合、それらすべてを追跡できる必要があります。 appsettings.json、web.config、または .env などのさまざまな方法を使用できます。 

> [!IMPORTANT]
> Bot Framework SDK 4.3 リリースより前、Microsoft ではリソース管理のメカニズムとして .bot ファイルを提供していました。 ただし今後は、これらのリソースを管理するために appsettings.json または .env ファイルを使用することをお勧めします。 .bot ファイルを使用するボットは、.bot ファイルが "**_非推奨_**" になった今でも機能します。 リソースの管理に .bot ファイルを使用している場合は、設定の移行のために適用される手順に従ってください。 

## <a name="migrating-settings-from-bot-file"></a>.bot ファイルから設定を移行する
次のセクションでは、.bot ファイルから設定を移行する方法について説明します。 ご自身に該当するシナリオに従ってください。

**シナリオ 1:.bot ファイルのあるローカル ボット**

このシナリオでは、.bot ファイルを使用するローカル ボットがあるが、Azure portal に "_ボットが移行されています_"。 以下の手順に従って、.bot ファイルから appsettings.json または .env ファイルに設定を移行します。

- .bot ファイルが暗号化されている場合は、次のコマンドを使用して暗号化解除する必要があります。

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
```

- 暗号化解除された .bot ファイルを開き、値をコピーして appsettings.json または .env ファイルに追加します。
- appsettings.json または .env ファイルから設定を読み取るようにコードを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`ConfigureServices` メソッドの場合、ASP.NET Core によって提供される次のような構成オブジェクトを使用します。 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

JavaScript の場合、次のような `process.env` オブジェクトから .env 変数の参照を解除します。
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

必要に応じて、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してボットに接続します。

**シナリオ 2:.bot ファイルを使用して Azure にデプロイされたボット**

このシナリオでは、.bot ファイルを使用して Azure portal に既にボットをデプロイ済みです。次に、.bot ファイルから .appsettings.json または .env ファイルに設定を移行します。

- Azure portal からボット コードをダウンロードします。 コードをダウンロードするときに、MicrosoftAppId、MicrosoftAppPassword、およびその他の設定が指定される appsettings.json または .env ファイルを含めるよう求められます。 
- "_ダウンロードした_" appsettings.json または .env ファイルを開きそこから設定を "_ローカル_" の appsettings.json または .env ファイルにコピーします。 必ず botSecret および botFilePath エントリをローカルの appsettings.json または .env ファイルから削除してください。
- appsettings.json または .env ファイルから設定を読み取るようにコードを更新します。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConfigureServices` メソッドの場合、ASP.NET Core によって提供される次のような構成オブジェクトを使用します。 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
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

"_必要に応じて_"、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してそれらをボットに接続します。

**シナリオ #3:appsettings.json または .env ファイルを使用するボットの場合**

このシナリオでは、SDK 4.3 を使用して一からボットの開発を開始する場合で、移行する既存の .bot ファイルがない場合ついて説明しています。 お使いのボットで使用するすべての設定は、以下に示されているように、appsettings.json または .env ファイル内にあります。

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

上記の設定を C# コードで読み取るには、ASP、NET Core によって提供される構成オブジェクトを使用します。**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
JavaScript の場合、`process.env` オブジェクト (**index.js** など) から .env 変数の参照を解除します。
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

必要に応じて、リソースをプロビジョニングし、appsettings.json または .env ファイルを使用してそれらをボットに接続します。


## <a name="faq"></a>FAQ
**質問:** Azure portal で新しい V4 ボットを作成する必要があります。 Azure portal で .bot ファイルを削除する操作方法はどのように変わっていますか。

**A:** Azure portal でボットを作成するとき、.bot ファイルは作成されません。 **Azure portal** の **[アプリケーション設定]** セクションで ID またはキーを検索できます。 コードをダウンロードするとき、これらの設定は appsettings.json または .env ファイルに自動的に格納されます。 お使いのボットで、個々のサービスへの呼び出しを行う前に、設定を読み取るようにコードを更新できます。 ボット コードを更新したら、az bot publish を使用してボットをデプロイできます。

**質問:** V3 ボットはどうなっていますか。

**A:** V3 ボットのシナリオは、ボット ファイルのない V4 ボットのシナリオに似ています。 デプロイ プロセスは引き続き機能します。 

## <a name="additional-resources"></a>その他のリソース
- ボットをデプロイする手順については、[デプロイ](../bot-builder-deploy-az-cli.md)に関するトピックをご覧ください。
- キーとシークレットを保護するには、Azure Key Vault を使用することをお勧めします。 Azure Key Vault は、お使いのボットのエンドポイントやオーサリング キーなどのシークレットに安全にアクセスして格納するためのツールです。 これにより、[キー管理](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis)ソリューションが実現され、暗号化キーを簡単に作成して管理できるため、シークレットを厳格に制御することができます。


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->

