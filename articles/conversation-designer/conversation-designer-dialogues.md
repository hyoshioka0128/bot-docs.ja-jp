---
title: Do アクションとしてのダイアログの定義 | Microsoft Docs
description: Do アクションとしてダイアログを設定する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d59df20821a7f63eb9ee5dea365597b1af839f75
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301569"
---
# <a name="define-a-dialogue-as-a-do-action"></a>Do アクションとしてのダイアログの定義
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

ダイアログは、特定のタスクを実現するための会話モデルを対象にしています。 たとえば、ユーザーによるテーブル予約を支援するカフェ ボットでは、"テーブルの予約" というタスクを定義できます。 **Do** アクションは、ボットとユーザーの会話フローをモデル化するダイアログにバインドされます。 

ダイアログは、ボットが、タスクを完了するためにユーザーとの会話のやり取りに従事する場合に特に役立ちます。  ユーザーが、タスクの完了に必要なすべての値を 1 回の発話で表現することはほとんどありません。 

テーブルを予約するには、場所、パーティの規模、日付、および時刻の指定が必要です。 テーブルを予約するボットは、ユーザーが発言する可能性のあるさまざまなフレーズを理解して処理する必要があります。 

- "*テーブルを予約する*": この場合、必須エンティティはまったく取得されませんでした。 ボットは、ユーザーとの会話に関与する必要があります。
- "*今度の土曜日の午後 7 時にテーブルを予約する*": この場合、日付と時刻の希望は指定されていますが、ボットは所望する場所とパーティの規模をさらに収集する必要があります。

会話中に、ユーザー入力に基づいて一部のエンティティが変更される可能性があります。 たとえば、ユーザーが "今度の日曜日の午後 7 時にレドモンドでテーブルを予約する" と言っているが、 レドモンドのお店が日曜日は 6 時に閉店になる場合、ボットのダイアログは、このような無効な要求を処理する必要があります。 

Conversation Designer には、会話フローの視覚化に役立つ、ドラッグ アンド ドロップ方式のダイアログ デザイナーが用意されています。 このダイアログ デザイナーには、会話フローのモデル化に使用できる 7 つの "*ダイアログの状態*" があります。

## <a name="dialogue-states"></a>ダイアログの状態

ダイアログは会話の状態で構成されます。 ダイアログ自体は構造化され方向付けされたフローとしてモデル化されます。このフローによって、会話フローを実行する方法を表す構造体が会話ランタイムに提供されます。

ダイアログには、使用できる状態が多数組み込まれています。 サポートされている組み込みの状態を以下に示します。

- [**start**](#start-state): 会話フローの開始時の状態を表します。 すべてのダイアログに、start 状態が少なくとも 1 つ定義されている必要があります。
- [**return**](#return-state): 特定の会話フローの完了を表します。 会話フローが構成可能な場合、return は、会話ランタイムに対して、このダイアログの呼び出し元のいずれかに実行を返すように指示します。
- [**decision**](#decision-state): 会話フローの分岐点を表します。
- [**process**](#process-state): ボットがビジネス ロジックを実行している状態を表します。
- [**prompt**](#prompt-state): ユーザーに入力を求めることができる状態を表します。 
- [**feedback**](#feedback-state): フィードバックや確認をユーザーに提供できる状態を表します。 たとえば、予約が行われたことを確認するダイアログが挙げられます。
- [**module**](#module-state): 別のダイアログの呼び出しを表します。 ダイアログ フローは既定で構成可能であるため、共有されているダイアログまたはこのタスクで定義されている他のダイアログを、この状態から呼び出すことができます。

各会話状態は、ダイアログ デザイナーでダイアログ コネクタを使用して、別の会話状態に接続されます。

ダイアログの各状態には、関連付けられている状態エディターがあり、これを使用して、カスタム スクリプトのコールバック関数名を含む、その状態のプロパティを指定します。 **状態エディター**は、ダイアログ デザイナーのメイン ビュー ポート下部にサイズ変更ウィンドウとして位置していました。 このエディターを起動するには、ダイアログ デザイナーで状態をダブルクリックすると、**状態エディター**にその状態のプロパティが表示されます。
<!-- TODO: insert screenshot of the wrench in horizontal menu -->

次のサブセクションでは、これらの組み込みのダイアログの各状態について詳しく説明します。

### <a name="start-state"></a>start 状態

start 状態は、ダイアログの開始地点を示します。 必須の値は、状態の**名前**です。 既定の名前は "start" ですが、編集してこの状態の名前を変更できます。

### <a name="return-state"></a>return 状態

return 状態は、会話フローの特定の分岐の完了を表します。 ダイアログは構成可能であるため、呼び出し元のダイアログに実行を戻すように状態から会話ランタイムに指示することもできます。 必須の値は、状態の**名前**です。 既定の名前は "return" ですが、編集してこの状態の名前を変更できます。

### <a name="decision-state"></a>decision 状態

decision 状態は、会話フローの分岐を表します。 カスタム スクリプトを記述して、どの分岐に従うかを評価することができます。 ユーザー入力とビジネス ロジックに応じて、スクリプトから、遷移値の候補が 1 つ返されます。 各遷移値は、ダイアログの別の分岐を実行するように、会話ランタイムに要求します。

decision 状態の必須プロパティは以下のとおりです。
- **[Name]\(名前\)**: decision 状態の一意の名前。
- **[Code to execute on run]\(実行時に実行するコード\)**: 会話のどの分岐に従うかを判断するビジネス ロジックを実装するコールバック関数の名前。 

#### <a name="example-code-for-decision-state"></a>decision 状態のコード例

次のコールバック関数のサンプルは、どの分岐を実行するかを会話ランタイムに指示する決定を返します。

```javascript
module.exports.fnDecisionState = function(context) {
    var a = context.taskEntities['a'];
    if (a[0].value === '0') {
        return 'yes';
    }
    else if (a[0].value === '1') {
        return 'no';
    }
}
```

### <a name="process-state"></a>process 状態

process 状態は、ボットが会話を先に進めるか、または最終的なタスク完了アクションの実行を試みる、ダイアログ内のポイントを表します。 

process 状態の必須のプロパティは以下のとおりです。
- **[Name]\(名前\)**: process 状態の一意の名前。
- **[Code to execute on run]\(実行時に実行するコード\)**: ビジネス ロジックを実装するコールバック関数の名前。

#### <a name="example-code-for-process-state"></a>process 状態のコード例

次のコールバック関数のサンプルは、天候を取得し、ユーザーに気象情報を返します。

```javascript
module.exports.fnGetWeather = function(context) {
    var options =  {
        host: 'mock',
        path: '/get?a=b',
        method: 'get'
    };
    return http.request(options).then(function(response) {
        context.contextEntities['x'].value= response.statusCode;
        var jsonBody = JSON.parse(response.body);
          // understand response
        if (response.statusCode != "200") {
            // error
        }
    });
}
```

### <a name="prompt-state"></a>prompt 状態

prompt 状態は、ユーザーに特定の情報を要求します。 prompt 状態にはサブダイアログ システムが統合されており、本質的に複合的な状態です。 

prompt 状態内では、ユーザーに提供する実際の応答を定義し、必要に応じてアダプティブ カードを含めることができます。 次に、ユーザーの応答を解析および理解するためのトリガーを指定できます。 このトリガーには、LUIS、または正規表現を使用したカスタム コード認識エンジンを指定できます。  

ユーザーが無効な入力を指定している場合、ボットはユーザーに同じ情報を再要求できます。 この動作は、prompt 状態エディターでも定義できます。 

#### <a name="prompting-the-user"></a>ユーザーへのプロンプト

プロンプトの応答によって、入力を**ユーザーに求める**ときに使用するメッセージを指定できます。 たとえば、日付と時刻を収集するには、応答として "ご来店はいつがよろしいですか"  や "お食事の時刻はいつがご希望ですか" が考えられます。

#### <a name="prompt-listening-for-user-input"></a>ユーザー入力のリッスンのプロンプト

ユーザーに応答を求めた後、会話ランタイムは自動的にユーザー入力をリッスンし、発言内容を理解しようとします。 ユーザー入力と意図を理解しようとするように、LUIS またはカスタム コード認識エンジンに基づいてトリガーを構成します。 これは、タスク トリガーに似ています。

#### <a name="re-prompting-the-user"></a>ユーザーへの再プロンプト

再プロンプト セクションを使用して、試行ごとに応答を指定します。 再プロンプト セクション内の各行は、その特定ターンで使用される再プロンプト文字列に対応します。 最初の応答は最初の再プロンプトに使用され、2 つ目は 2 つ目に使用され、それ以降も同様です。 例: 

"*申し訳ありません。理解できませんでした。ご来店はいつがよろしいですか。*"
"*申し訳ありません。理解できておりません。もう一度お願いします。ご来店はいつがよろしいですか。*"

#### <a name="prompt-callback-functions"></a>コールバック関数のプロンプト

prompt 状態では、2 つの異なるコールバック関数を指定できます。 

1. **[Before every prompt and reprompt]\(すべてのプロンプトまたは再プロンプトの前に\)**: すべてのプロンプトまたは再プロンプトの前に、この関数を実行します。 このコールバック関数は、ブール型の戻り値を予期しています。true はこのプロンプトまたは再プロンプトを実行することを意味し、false はこのプロンプトまたは再プロンプトを実行しないことを意味します。 そのプロンプト実行に対応する現在のターンのインデックスを取得するには、`getTurnIndex()` を使用できます。
2. **[On responding]\(応答時\)**: プロンプトが生成されるたびに、ただしユーザーに送信される前に、この関数を実行します (応答の再プロンプトを含む)。 これによって、ユーザーに送信されるメッセージを変更するスクリプトが有効になります。

#### <a name="sample-code"></a>サンプル コード

このコード スニペットは、**実行前**コールバックの例を示しています。

```javascript
module.exports.fnBeforeExecuting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

このコード スニペットは、**プロンプト時**コールバックの例を示しています。

```javascript
module.exports.fnOnPrompting = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });
}
```

このコード スニペットは、**再プロンプト前**コールバックの例を示しています。

```javascript
module.exports.fnBeforeReprompting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

### <a name="feedback-state"></a>feedback 状態

この状態は、ユーザーに応答を返すために使用します。 この一般的なユース ケースとしては、タスク実行の試みの後の最終結果や、失敗の状況でのユーザーへの反応などがあります。 

各 feedback 状態では、一意の名前、複数の応答値の候補が必要であり、必要に応じてアダプティブ カードの定義を含めることができます。 アダプティブ カードの定義の詳細については、[こちら](conversation-designer-adaptive-cards.md)を参照してください。

各 feedback 状態では、**[On responding]\(応答時\)** コールバック関数も許可されます。ここではカスタム スクリプトを記述して、ユーザーに送信される前に必要に応じて、アクティビティのペイロードを変更することができます。 


```javascript
module.exports.fnOnResponding = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });

}
```

### <a name="module-state"></a>module 状態

module 状態は、参照を追加してサブダイアログを実行するために使用されます。 この状態は、ダイアログをつなぎ合わせるために使用します。 

各 module 状態では、一意の名前と、実行する特定のダイアログへのポインターが必要です。 実行されるダイアログのオプションの候補は、**共有ダイアログ**または特定の**タスク**の下に存在する必要があります。

## <a name="multiple-dialogues-under-a-task"></a>1 つのタスクでの複数のダイアログ

1 つのタスクで、複数のダイアログを指定できます。 タスクにダイアログを追加するには、単純にタスクを選択して、左側のツリー パネルで **[追加]** をクリックし、**[Add dialogue]\(ダイアログの追加\)** をクリックします。 これで、選択したタスクの下に新しいダイアログが追加されます。 

ダイアログは構成可能であるため、タスクの下の他のダイアログを呼び出すタスクに、ルート ダイアログのフローをバインドできます。 これによって、サブタスクをカプセル化し、再利用できます。 また、*module* 状態を使用して、会話フロー内でこれらのダイアログをつなぐこともできます。

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [応答のテンプレート](conversation-designer-response-templates.md)
