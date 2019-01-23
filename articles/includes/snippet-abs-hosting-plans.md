---
ms.openlocfilehash: 8eb125b2d0f0c9981cafb763ba809c7d042d8f77
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225987"
---
Bot Service では、ボット用に 2 種類のホスティング プランを提供しています。 ボット ソース コードのプランの切り替えは、手動によるプロセスとなります。   

## <a name="app-service-plan"></a>App Service プラン

App Service プランを使用するボットは、コストとスケーリングが予測可能な定義済みの容量を割り当てるように設定できる標準的な Azure Web アプリです。 このホスティング プランを使用するボットの場合、次のことができます。

* 高度なブラウザー内コード エディターを使用してボットのソース コードをオンラインで編集する。
* Visual Studio を使用して、ご利用になる C# ボットをダウンロード、デバッグ、および再発行する。
* Visual Studio Online および Github で継続的デプロイを容易に設定する。
* Bot Framework SDK 用に準備されたサンプル コードを使用する。

## <a name="consumption-plan"></a>従量課金プラン

従量課金プランを使用するボットは、<a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> で実行され、実行単位支払いの Azure Functions 価格が使用されるサーバーレス ボットです。 このホスティング プランを使用するボットでは、膨大なトラフィックの急増に対処するためにスケーリングすることができます。 基本的なブラウザー内コード エディターを使用してボットのソース コードをオンラインで編集することができます。 従量課金プランのボットのランタイム環境の詳細については、<a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions の従量課金プランと App Service プラン</a>に関するページを参照してください。
