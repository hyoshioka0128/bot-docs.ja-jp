---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405538"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">オープン ソースの Web チャット コントロール</a>は、[Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle) を使用してボットと通信し、クライアントとボット間での `activities` のやり取りを可能にします。 最も一般的な種類のアクティビティは `message` ですが、他の種類もあります。 たとえば、`typing` というアクティビティの種類は、ユーザーが入力中であることか、ボットが応答をまとめるために動作中であることを示します。 

アクティビティの種類を `event` に設定すれば、バックチャネル メカニズムを使用して、ユーザーに情報を表示せずに、クライアントとボット間で情報を交換できます。 Web チャット コントロールは、`type="event"` のすべてのアクティビティを自動的に無視します。