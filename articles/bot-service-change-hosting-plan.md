---
title: C# ボットを従量課金プランから App Service プランに移行する | Microsoft Docs
description: Bot Service の C# ボットを従量課金ホスティング プランから App Service ホスティング プランに移行します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: e463b272385b97e630d4087908aa82e23a70fea9
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568189"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>Bot Service のホスティング プランを変更する

このトピックでは、従量課金プランの C# スクリプト ボットを App Service プランの C# ボットに移行する方法について説明します。 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>App Service プランのボットの利点

App Service プランのボットは、Azure Web アプリとして実行します。 Web アプリのボットは、従量課金プランのボットではできないことを実行できます。

- Web アプリのボットは、カスタム ルート定義を追加できます。
- Web アプリのボットは、Websocket サーバーを有効にできます。 
- 従量課金ホスティング プランを使用するボットには、Azure Functions で実行されるすべてのコードと同じ制限があります。 詳しくは、「<a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions の従量課金プランと App Service プラン</a>」をご覧ください。

## <a name="download-your-existing-bot-source"></a>既存のボット ソースをダウンロードする

以下の手順に従って、既存のボットのソース コードをダウンロードします。

1. Azure ボット内で **[設定]** タブをクリックし、**[継続的デプロイ]** セクションを展開します。  
2. 青いボタンをクリックして、ボットのソース コードを含む zip ファイルをダウンロードします。  
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]
    ![ボットの zip ファイルをダウンロードする](~/media/continuous-deployment-consumption-download.png)
3. ダウンロードした zip ファイルの内容をローカル フォルダーに抽出します。 


## <a name="create-a-bot-template"></a>ボットのテンプレートを作成する

Bot Service では、従量課金プランのボットも App Service プランのボットもテンプレートは同じです。 従量課金プランのボットから移行するには、Bot Service で同じテンプレートを基にして新しい App Service プランのボットを作成します。 テンプレートは同じでも基になるコードは 2 つのホスティング タイプで異なることがありますが、新しい Web アプリの構造と構成機能は既存のボットで使われているものと似ています。

## <a name="download-the-new-bot-source"></a>新しいボットのソースをダウンロードする

以下の手順に従って、新しいボットのソース コードをダウンロードします。

1. Azure ボット内で、**[BUILD]\(ビルド\)** タブをクリックし、**[Download source code]\(ソース コードのダウンロード\)** セクションを探して、**[Download zip file]\(zip ファイルのダウンロード\)** をクリックします。 
2. ダウンロードした zip ファイルの内容をローカル フォルダーに抽出します。

## <a name="add-source-files-to-new-solution"></a>ソース ファイルを新しいソリューションに追加する

いくつかの .csx ファイルが、新しいソリューションでは .cs ファイルとしてコンパイルされて実行される場合があります。 `run.csx` を除くソリューション内の各 .csx ファイルに対して、.cs ファイルを作成します。 `run.csx` のロジックは手動で移行します。 .cs ファイルでは、クラスの宣言および省略可能な名前空間の宣言を追加することが必要な場合があります。

## <a name="migrate-runcsx-logic-into-your-project"></a>run.csx のロジックをプロジェクトに移行する

C# スクリプト プロジェクトの `Run` メソッドでは、異なる `ActivityTypes` 値が処理されます。 アクティビティ処理ロジックを `MessageController.cs` の `MessageController.Post` メソッドにインポートします。

## <a name="remove-compiler-keywords"></a>コンパイラ キーワードを削除する

C# スクリプト ファイルには、`#r` キーワードを使用して参照モジュールが含まれていることがあります。 これらの行を削除し、参照としてこれらを Visual Studio プロジェクトに追加します。 また、他のソース コード ファイルをファイルのコンパイルに挿入する `#load` キーワードも削除します。 代わりに、プロジェクトに関するすべての `.csx` ファイルを `.cs` ソース コードとして追加します。

## <a name="add-references-from-projectjson"></a>project.json から参照を追加する

従量課金プランのボットが `project.json` ファイルで NuGet の参照を追加している場合は、これらの参照を新しい Visual Studio ソリューションに追加します。そのためには、ソリューション エクスプローラーでプロジェクトを右クリックして、**[参照の追加]** をクリックします。

### <a name="add-references-that-were-implicit"></a>暗黙で行われていた参照を追加する

従量課金プランの Bot Service ボットでは、これらの参照がすべての .csx ソース ファイルに暗黙で含まれています。 .cs ソース ファイルに移行されたソースでは、これらのクラスに対する明示的な参照を追加することが必要な場合があります。

- `Microsoft.Azure.WebJobs.Host` 名前空間の型を提供する NuGet パッケージ `Microsoft.Azure.WebJobs` の `TraceWriter`。 
- NuGet パッケージ `Microsoft.Azure.WebJobs.Extensions` のタイマー トリガー
- `Newtonsoft.Json`、`Microsoft.ServiceBus`、および他の自動的に参照されるアセンブリ
- `System.Threading.Tasks` および他の自動的にインポートされる名前空間

その他のガイダンスについては、「<a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>Publishing a .NET class library as a Function App</a>」(Function App としての .NET クラス ライブラリの公開) の「*Converting to class files*」(クラス ファイルへの変換) をご覧ください。

## <a name="debug-your-new-bot"></a>新しいボットをデバッグする

ローカルなデバッグは App Service プランのボットの方が従量課金プランのボットよりはるかに簡単です。 [エミュレーター](bot-service-debug-emulator.md)を使って、移行したコードをローカルにデバッグできます。

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>Visual Studio から発行する、または継続的デプロイを設定する

最後に、`.PublishSettings` ファイルをインポートして **[発行]** をクリックするか、または[継続的配置を設定する](bot-service-debug-bot.md)ことにより、移行されたソース コードを Bot Service に発行します。
