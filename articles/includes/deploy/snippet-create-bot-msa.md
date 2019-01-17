---
ms.openlocfilehash: fed6222fb9cf2d7793776c5575dfbcda49b54224
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360919"
---
1. [**アプリケーション登録ポータル**](https://apps.dev.microsoft.com/)に移動します。
1. **[アプリの追加]** をクリックしてアプリケーションを登録して、**アプリケーション ID** を作成し、**新しいパスワードを生成**します。 既にアプリケーションとパスワードを持っていて、パスワードを思い出せない場合は、[アプリケーション シークレット] セクションで新しいパスワードを生成する必要があります。
1. `az bot create` コマンドで使用できるように、生成したアプリケーション ID と新しいパスワードの両方を控えておきます。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| オプション | 説明 |
|:---|:---|
| --name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| --location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| --lang | ボットの作成に使用する言語: `Csharp` または `Node`。既定値は `Csharp` です。 |
| --resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |
| --appid | ボットで使用される Microsoft アカウント ID (MSA ID)。 |
| --password | ボットの Microsoft アカウント (MSA) パスワード。 |
