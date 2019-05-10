---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035701"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| オプション | 説明 |
|:---|:---|
| --name | リソース グループの一意名。 名前にスペースやアンダースコアを含めないでください。 |
| --location | リソース グループの作成に使用する地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 場所の一覧を取得するには `az account list-locations` を使用します。 |