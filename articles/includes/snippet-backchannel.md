---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230549"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">オープン ソースの Web チャット コントロール</a>は、[Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle) を使用してボットと通信し、クライアントとボット間での `activities` のやり取りを可能にします。 最も一般的な種類のアクティビティは `message` ですが、他の種類もあります。 たとえば、`typing` というアクティビティの種類は、ユーザーが入力中であることか、ボットが応答をまとめるために動作中であることを示します。 

アクティビティの種類を `event` に設定すれば、バックチャネル メカニズムを使用して、ユーザーに情報を表示せずに、クライアントとボット間で情報を交換できます。 Web チャット コントロールは、`type="event"` のすべてのアクティビティを自動的に無視します。