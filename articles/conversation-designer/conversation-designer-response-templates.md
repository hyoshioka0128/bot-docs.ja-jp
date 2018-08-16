---
title: 応答テンプレート | Microsoft Docs
description: Conversation Designer ボットの応答テンプレートを設定する方法について説明します。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b09b7fa5f5672c121711deb2edcdc9718a79983f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304185"
---
# <a name="response-template-for-conversation-designer-bots"></a>Conversation Designer ボットの応答テンプレート
> [!IMPORTANT]
> Conversation Designer は、一部のお客様はまだご利用いただけません。 Conversation Designer の可用性の詳細については、今年中に提供される予定です。

言語生成を使用すると、ボットは可変の応答メッセージを使用してリッチかつ自然な形でユーザーとコミュニケーションができます。 これらのメッセージは、Conversation Designer の応答テンプレートによって管理されます。

応答テンプレートを使用すると、応答の再利用が可能になり、ボットの応答で一貫したトーンや言葉遣いを維持しやすくなります。 

## <a name="create-response-templates"></a>応答テンプレートの作成

Conversation Designer では、ユーザーに応答を送信することが必要なあらゆる状況で再利用できる応答テンプレートを作成することができます。 

単純応答テンプレートは、可能性のある音声および表示上の発話の 1 回限りのコレクションを定義します。 その後、ボットの応答、プロンプト状態、またはアダプティブ カードでこれらのテンプレートを使用して、完全に解決された文字列を再構成することができます。

単純応答テンプレートを追加するには、次の手順を実行します。
1. 左パネルから **[追加]** をクリックします。 コンテキスト メニューが表示されます。
2. **[Simple response]\(単純応答\)** をクリックします。 メイン コンテンツ パネルに編集ウィンドウが表示されます。
3. **[Simple response name]\(単純応答名\)** フィールドに、この単純応答の名前を入力します。
4. **[Bot's response to user]\(ユーザーへのボットの応答\)** フィールドに、応答を一度に 1 フレーズずつ入力します。
5. 単純応答の設定が終了したら、右上隅の **[保存]** をクリックします。 

たとえば、次の項目がある ("AckPhrase" という名前の) 受信確認フレーズ テンプレートを作成することができます。

- [OK]
- 承知しました
- お任せください
- 喜んでお手伝いします
- お手伝いします

このテンプレートを使用すると、会話ランタイムはこれらの文字列のいずれかにランダムに解決して、ボットの応答が退屈で単調な感じではなく自然に聞こえるようにします。

## <a name="conditional-response-templates"></a>条件付き応答テンプレート

条件付き応答テンプレートでは、条件付きの複数の応答を指定します。 記述するコールバック関数は、使用する応答文字列を指定の条件に基づいて言語生成エンジンが決定するのを補助します。 

たとえば時刻テンプレートは、実際の時刻に基づいて "おはようございます" または "おやすみなさい" に解決する必要があります。 

条件付き応答テンプレートを追加するには、次の手順を実行します。
1. 左パネルから **[追加]** をクリックします。 コンテキスト メニューが表示されます。
2. **[Conditional response]\(条件付き応答\)** をクリックします。 メイン コンテンツ パネルに編集ウィンドウが表示されます。
3. **[Conditional response name]\(条件付き応答名\)** フィールドに、このテンプレートの名前を入力します。
4. **[Conditional response]\(条件付き応答\)** フィールドに、フレーズを一度に 1 つずつ入力します。 たとえば、"timeOfDayTemplate" の場合、テンプレートで可能性のある 2 つの応答として「おはようございます」と「おやすみなさい」を入力します。
5. "おはようございます" 応答の **[条件名]** を "morning" と指定します。 同様に、"おやすみなさい" 応答の **[条件名]** を "evening" と指定します。 記述するカスタム スクリプトは、これらの条件のどちらか 1 つを返します。
6. **[Code to execute on run]\(動作時に実行するコード\)** フィールドで、コールバック関数名 (例: `fnResolveTimeOfDayTemplate`) を入力します。 次に、**[View code]\(コードの表示\)** をクリックし、*[スクリプト]** エディターを読み込みます。 そこから、このコールバック関数の実装を定義できます。
7. 右上隅にある **[保存]** をクリックして変更を保存し、このテンプレートを作成します。

*fnResolveTimeOfDayTemplate* コールバック関数のサンプル コード。 この関数は、応答によって指定された "morning" または "evening" の **[条件名]** のどちらかに一致する文字列を返します。

```javascript
module.exports.fnTimeOfDayTemplate = function(context) {
    var currentTime = new Date().getHours();
    if(currentTime >= 12) {
        return "evening!";
    } else {
        return "morning!";
    }
}
```

## <a name="send-a-response-to-user"></a>応答をユーザーに送信する

応答テンプレートを使用してユーザーに応答を送信するには、いずれかのテンプレート名を角かっこで囲みます。 たとえば、単純応答テンプレート "AckPhrase" をフィードバック状態で使用するには、**[Bot's response to user]\(ユーザーへのボットの応答\)** に「`[AckPhrase], I will get that done for you right away`」と入力します。

応答で角かっこを使用する場合、エスケープ文字として "\" を使用して、角かっこの解決をスキップするよう変換ランタイムに指示します。

エンティティが定義されている場合、それらをユーザーへの応答で使用することもできます。 エンティティ名を中かっこで囲むことにより、言語生成でエンティティを参照できます。 たとえば、`location` エンティティがボットにある場合、応答内で次のようにして参照できます: `[AckPhrase], I can help you find a table at {location}.`

## <a name="nesting-response-templates"></a>応答テンプレートの入れ子化

応答テンプレートを "入れ子" にする、つまり、ある応答テンプレートから別の応答テンプレートを参照することができます。 たとえば、単純応答テンプレート `AckPhrase` と条件付き応答テンプレート `timeOfDayTemplate` をベースに、両方のテンプレートを使用して最終的な応答を解決する新しい応答テンプレート `timeBasedGreeting` を作成することができます。 

> [!TIP]
> テンプレートの入れ子化は強力な機能ですが、必ず、入れ子にしたテンプレートで無限ループが発生しないことを確認してください。 つまり、`AckPhrase` が `timeOfDayTemplate` を呼び出し、`timeOfDayTemplate` が逆に `AckPhrase` を呼び出すような状況を避けてください。

たとえば、新しい**単純応答**を `timeBasedGreeting` という名前で作成し、このテンプレートで可能性がある **[Bot's response to user]\(ユーザーへのボットの応答\)** として次のテキストを入力します: `[timeOfDayTemplate] [AckPhrase], ... `

## <a name="define-user-utterance-as-help-tips"></a>ユーザーの発話をヘルプ ヒントとして定義する

ユーザーの**発話**を定義する間、**ヘルプ ヒント**として発話を設定することもできます。 発話を**ヘルプ ヒント**として設定するには、発話の右上にある `...` をクリックして **[Use as help tip]\(ヘルプ ヒントとして使用\)** を選択します。 

ヘルプ ヒントは、組み込みの言語生成テンプレート `[builtin-tasktips]` で、**[Bot's response to user]\(ユーザーへのボットの応答\)** フィールドを定義するすべての場所で使用されます。 たとえば、次のような応答を作成できます: `Sorry I did not understand that. Here are some things you can try - [builtin.tasktips]`

タスクに**ヘルプ ヒント**が定義されていない場合、`[builtin-tasktips]` は空文字列に解決します。 タスクに複数の**ヘルプ ヒント**が定義されている場合、`[builtin-tasktips]` は応答が与えられるたびに 1 つをランダムに選択します。

## <a name="next-step"></a>次のステップ
> [!div class="nextstepaction"]
> [アダプティブ カード](conversation-designer-adaptive-cards.md)
