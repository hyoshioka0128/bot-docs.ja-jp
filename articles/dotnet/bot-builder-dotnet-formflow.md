---
title: FormFlow の基本的機能 | Microsoft Docs
description: Bot Framework SDK for .NET 内で FormFlow を使用して会話フローを導く方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4e9aa1b7bffd55518bd4ef03512d873ac48b49a7
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297858"
---
# <a name="basic-features-of-formflow"></a>FormFlow の基本的機能

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[ダイアログ](bot-builder-dotnet-dialogs.md)は非常に強力で柔軟性がありますが、サンドイッチの注文などのガイド付きの会話を処理するには、多大な労力を必要とすることがあります。 会話の各ポイントには、次に起こることについての多くの可能性があります。 たとえば、あいまいさの排除やヘルプの提供を必要とする場合もあれば、前に戻ったり、進行状況を示したりすることが必要な場合もあります。 Bot Framework SDK for .NET 内で **FormFlow** を使用することで、このようなガイド付きの会話を管理するプロセスを大幅に簡素化できます。 

FormFlow では、指定したガイドラインに基づいて、ガイド付きの会話を管理するために必要なダイアログが自動的に生成されます。 FormFlow を使用すると、ダイアログを独自に作成して管理することによって得られる柔軟性が損なわれますが、FormFlow を使用してガイド付きの会話を設計すると、ボットの開発に要する時間を大幅に短縮できます。 さらに、FormFlow で生成されたダイアログと他の種類のダイアログを組み合わせてボットを作成することもできます。 たとえば、FormFlow ダイアログではユーザーを誘導してフォームを完成させるプロセスを進め、[LuisDialog][LuisDialog] ではユーザー入力を評価して意図を判断することができます。

この記事では、FormFlow の基本的機能を使用してユーザーから情報を収集するボットを作成する方法について説明します。

## <a id="forms-and-fields"></a>フォームとフィールド

FormFlow を使用してボットを作成するには、ボットがユーザーから収集する必要がある情報を指定する必要があります。 たとえば、ボットの目的がユーザーのサンドイッチ注文を受けることである場合、ボットが注文を完了するために必要なデータのフィールドが含まれたフォームを定義する必要があります。 フォームを定義するには、ボットがユーザーから収集するデータを表す 1 つ以上のパブリック プロパティを含む C# クラスを作成します。 各プロパティは、次のいずれかのデータ型である必要があります。

- 整数 (sbyte、byte、short、ushort、int、uint、long、ulong)
- 浮動小数点 (float、double)
- string
- DateTime
- Enumeration
- 列挙型のリスト

どのデータ型も Null 許容にすることができます。これを使用して、フィールドに値がないことをモデル化できます。 フォーム フィールドが Null 許容ではない列挙型プロパティに基づいている場合、列挙型の値 **0** は **null** を表す (つまり、フィールドに値がないことを示す) ので、列挙値は **1** から開始する必要があります。 FormFlow では、プロパティの他のすべての型とメソッドは無視されます。

複雑なオブジェクトの場合、最上位の C# クラス用のフォームと複雑なオブジェクト用の別のフォームを作成する必要があります。 一般的な[ダイアログ](bot-builder-dotnet-dialogs.md) セマンティクスを使用して、フォームをまとめて構成できます。 [Advanced.IField][iField] を実装するか、[Advanced.Field][field] を使用し、その中でディクショナリを設定することによって、フォームを直接定義することもできます。 

> [!NOTE]
> フォームは、C# クラスまたは JSON スキーマを使用して定義できます。 この記事では、C# クラスを使用してフォームを定義する方法を説明します。 JSON スキーマの使用方法の詳細については、「[Define a form using JSON schema (JSON スキーマによるフォームの定義)](bot-builder-dotnet-formflow-json-schema.md)」をご覧ください。

## <a name="simple-sandwich-bot"></a>簡単なサンドイッチ ボット

ユーザーのサンドイッチの注文を受けるように設計された簡単なサンドイッチ ボットのこの例について考えてみましょう。 

### <a id="create-class"></a>フォームを作成する

`SandwichOrder` クラスではフォームを定義し、列挙型ではサンドイッチを作るためのオプションを定義します。 このクラスには、[FormBuilder][formBuilder] を使用してフォームを作成し、簡単なウェルカム メッセージを定義する静的 `BuildForm` メソッドも含まれています。 

FormFlow を使用するには、まず、`Microsoft.Bot.Builder.FormFlow` 名前空間をインポートする必要があります。

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>フォームをフレームワークに接続する 

フォームをフレームワークに接続するには、フォームをコントローラーに追加する必要があります。 この例では、`Conversation.SendAsync` メソッドが静的 `MakeRootDialog` メソッドを呼び出し、このメソッドが `FormDialog.FromForm` メソッドを呼び出して `SandwichOrder` フォームを作成します。 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>実際の動作を確認する

C# クラスを使用してフォームを定義し、フォームをフレームワークに接続するだけで、FormFlow がボットとユーザー間の会話を自動的に管理できるようになりました。 以下に示す対話例は、FormFlow の基本的機能を使用して作成されたボットの機能を示しています。 各対話では、 **>** 記号はユーザーが応答を入力するポイントを示しています。 

#### <a name="display-the-first-prompt"></a>最初のプロンプトを表示する 

このフォームでは、`SandwichOrder.Sandwich` プロパティを設定します。 フォームによって、"Please select a sandwich" というプロンプトが自動的に生成されます。プロンプトの "sandwich" という単語はプロパティ名の `Sandwich` から派生しています。 `SandwichOptions` 列挙型では、ユーザーに提示される選択肢が定義されています。各列挙値は、大文字小文字の変化とアンダースコアに基づいて、自動的に単語に分割されます。

```console
Please select a sandwich
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

#### <a name="provide-guidance"></a>ガイダンスを提供する

ユーザーは、会話の任意の時点で「help」と入力することで、フォームへの入力に関するガイダンスを得ることができます。 たとえば、ユーザーがサンドイッチ プロンプトで「help」と入力すると、ボットは次のガイダンスで応答します。 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>次のプロンプトに進む

ユーザーが最初のサンドイッチ プロンプトに応答して「2」と入力すると、フォームで定義された次のプロパティ (`SandwichOrder.Length`) のプロンプトが表示されます。

```console
Please select a sandwich
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>前のプロンプトに戻る 

ユーザーが会話のこの時点で「back」と入力すると、前のプロンプトが返されます。 プロンプトには、ユーザーの現在の選択 ("Black Forest Ham") が示されます。ユーザーは別の番号を入力してその選択を変更することも、「c」と入力してその選択を確定することもできます。

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>ユーザー入力を明確にする

ユーザーが選択を示す際に (番号ではなく) テキストで応答し、ユーザー入力が複数の選択肢と一致する場合、ボットは自動的に明確化を要求します。 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

ユーザー入力が有効な選択肢のいずれとも直接一致しない場合、ユーザーに明確化を求めるメッセージが自動的に表示されます。

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

ユーザー入力がプロパティの複数の選択肢を示しており、指定された選択肢のいずれかをボットが理解できない場合、ユーザーに明確化を求めるメッセージが自動的に表示されます。

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>現在の状態を表示する

ユーザーが注文の任意の時点で「status」と入力すると、ボットの応答には、既に指定されている値と指定する必要がある値が示されます。 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>選択を確認する

ユーザーがフォームへの入力を完了すると、ボットはユーザーに選択を確認するよう求めます。

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

ユーザーが「no」と入力して応答すると、ボットはユーザーが以前の選択を更新できるようにします。 ユーザーが「yes」を入力して応答すると、フォームの入力が完了し、呼び出し元のダイアログに制御が戻されます。 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>終了と例外の処理

ユーザーがフォームに「quit」と入力した場合や、会話のある時点で例外が発生した場合、ボットはイベントが発生したステップ、イベントが発生したときのフォームの状態、イベントの前に正常に完了していたフォームのステップを把握する必要があります。 フォームは、`FormCanceledException<T>` クラスを使用してこの情報を返します。 

次のコード例は、例外をキャッチし、発生したイベントに応じてメッセージを表示する方法を示しています。 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>まとめ

この記事では、FormFlow の基本的機能を使用して、次のことができるボットを作成する方法について説明しました。

- 会話を自動的に生成して管理する
- 明確なガイダンスとヘルプを提供する
- 番号とテキストエントリの両方を理解する
- 理解しているものとそうでないものについてユーザーにフィードバックを提供する 
- 必要に応じて明確化のための質問をする 
- ユーザーがステップ間を移動できるようにする

FormFlow の基本的機能で十分な場合もありますが、FormFlow のより高度な機能をボットに組み込むことで得られる潜在的なメリットを検討する必要があります。 詳細については、「[Advanced features of FormFlow (FormFlow の高度な機能)](bot-builder-dotnet-formflow-advanced.md)」および「[Customize a form using FormBuilder (FormBuilder によるフォームのカスタマイズ)](bot-builder-dotnet-formflow-formbuilder.md)」をご覧ください。

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>次の手順

FormFlow により、ダイアログの開発が簡素化されます。 FormFlow の高度な機能を使用すると、FormFlow オブジェクトの動作をカスタマイズできます。

> [!div class="nextstepaction"]
> [FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>その他のリソース

- [FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)
- [フォーム コンテンツのローカライズ](bot-builder-dotnet-formflow-localize.md)
- [JSON スキーマによるフォームの定義](bot-builder-dotnet-formflow-json-schema.md)
- [パターン言語によるユーザー エクスペリエンスのカスタマイズ](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
