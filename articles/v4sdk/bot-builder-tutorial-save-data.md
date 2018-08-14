---
title: チュートリアル - ユーザー状態データを保存する | Microsoft Docs
description: Bot Builder SDK でユーザー状態データを保存する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10c1cb240a22c1c16dd0d946ee55531d514f332e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303188"
---
# <a name="save-user-state-data"></a>ユーザー状態データを保存する

ユーザーがボットから入力を求められている場合、何らかの形式のストレージに情報の一部を保持したいと考えるのではないでしょうか。 Bot Builder SDK を使用すると、*メモリ内ストレージ*、*ファイル ストレージ*、データベース ストレージ (*CosmosDB* または *SQL*) を使用してユーザー入力を格納することができます。 

このチュートリアルでは、ミドルウェア レイヤーでストレージ オブジェクトを定義する方法、および永続化が可能なようにストレージ オブジェクトにユーザーを保存する方法について説明します。

## <a name="prequisite"></a>前提条件 

このチュートリアルは、[ウォーターフォールを使用した会話フローの管理](bot-builder-tutorial-waterfall.md)に関するチュートリアルに基づいて作成されています。

## <a name="add-storage-to-middleware-layer"></a>ミドルウェア レイヤーにストレージを追加する


## <a name="save-user-input-to-storage"></a>ストレージにユーザー入力を保存する

テーブル予約の会話を管理するには、4 つの手順を使用して**ウォーターフォール** ダイアログを定義することが必要になります。 この会話では、`TextPrompt` に加えて、`DatetimePrompt` と `NumberPrompt` も使用します。

`reserveTable` ダイアログは次のようになります。

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

これで、これをボット ロジックにフックする準備が整いました。

## <a name="start-the-dialog"></a>ダイアログを開始する

ご利用のボットの `processActivity()` メソッドを次のように変更します。

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

実行時に、ユーザーが文字列 `reserve table` を含むメッセージを送信するたびに、ボットによって `reserveTable` 会話が開始されます。

## <a name="next-steps"></a>次の手順

??? 

> [!div class="nextstepaction"]
> [ユーザー状態データを保存する](bot-builder-tutorial-save-data.md)
