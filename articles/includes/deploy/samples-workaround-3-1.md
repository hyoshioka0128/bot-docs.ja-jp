---
ms.openlocfilehash: fa4a49fea5bc2c85459e73fd89ec47a908864eb5
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80499825"
---
> [!IMPORTANT]
> Bot Framework 4.8 SDK のリリースでは、.NET Bot Framework サンプルで .NET Core 3.1 SDK を対象とするようになりました。
> すべての Azure データ センターがこのようなボットを構築するように構成されているわけではありません。
>
> Kudu を使用して .NET Core 3.1 アプリを構築できるセンターについては、[App Service の .NET Core](https://aspnetcoreon.azurewebsites.net/) のマップを参照してください。 (すべてのセンターで .NET Core 3.1 アプリを実行できます。)
>
> .NET Core 3.1 SDK を対象とするボットをデプロイするときに、Kudu を使用して .NET Core 3.1 アプリを構築できないセンターにデプロイする場合は、この回避策を使用してボット ファイルを準備し、圧縮します。それ以外の場合は、以降のセクションの手順を使用できます。
>
> 1. ボットのソリューションまたはプロジェクト ファイルが格納されているディレクトリで、このコマンドを実行します。
>
>    ```powershell
>    dotnet publish --configuration Release --runtime win-x86 --self-contained
>    ```
>
> 1. **/Release/netcoreapp3.1/publish/** フォルダーの内容を圧縮します。
