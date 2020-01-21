---
title: FormFlow の高度な機能 - Bot Service
description: FormFlow と Bot Framework SDK for .NET を使用してユーザー エクスペリエンスをカスタマイズする方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5620fd3a0e26cf7b56772e6bc8f47b8ceac596cd
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796671"
---
# <a name="advanced-features-of-formflow"></a>FormFlow の高度な機能

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

「[Basic features of FormFlow](bot-builder-dotnet-formflow.md)」(FormFlow の基本的機能) では、ごく般的なユーザー エクスペリエンスを提供する基本的な FormFlow 実装について説明しています。 FormFlow を使用してさらにカスタマイズされたユーザー エクスペリエンスを提供するために、フォームの初期状態を指定し、フィールド間の相互依存関係を管理してユーザー入力を処理するビジネス ロジックを追加しすることができます。また、プロンプトのカスタマイズ、テンプレートのオーバーライド、省略可能なフィールドの指定、ユーザー入力の照合、ユーザー入力の検証を行う属性を使用できます。 

## <a name="specify-initial-form-state-and-entities"></a>フォームの初期状態とエンティティを指定する

[FormDialog][formDialog] を起動するときに、必要に応じて状態のインスタンスを渡すことができます。 状態のインスタンスを渡すと、既定で FormFlow は、既に値が含まれるすべてのフィールドのステップをスキップします。そのため、ユーザーはこれらのフィールドの入力を求められません。 (初期状態で既に値が含まれるフィールドを含め) フォームのすべてのフィールドについてユーザーに入力を求めるように強制するには、`FormDialog` を起動するときに [FormOptions.PromptFieldsWithValues][promptFieldsWithValues] を渡します。 フィールドに初期値が含まれている場合、プロンプトの既定値としてその値が使用されます。

[LUIS](https://luis.ai/) のエンティティを渡して状態にバインドすることもできます。 `EntityRecommendation.Type` が C# クラスのフィールドへのパスである場合、`EntityRecommendation.Entity` は認識エンジンを介して渡され、フィールドにバインドされます。 FormFlow は、エンティティにバインドされているすべてのフィールドのステップをスキップします。そのため、ユーザーはこれらのフィールドの入力を求められません。 

## <a name="add-business-logic"></a>ビジネス ロジックを追加する 

フィールド値の取得時または設定時にフォーム フィールド間の相互依存関係を処理する場合や、特定のロジックを適用する場合は、検証機能内でビジネス ロジックを指定できます。 検証機能を使用すると、状態を操作し、以下を含む [ValidateResult][validateResult] オブジェクトを返すことができます。 

- 値が無効である理由を説明するフィードバック文字列
- 変換された値
- 値を明確にするための一連の選択肢

このコード例は、`Toppings` フィールドの検証機能を示しています。 フィールドの入力に `ToppingOptions.Everything`列挙値が含まれている場合、この機能で `Toppings` フィールド値にトッピングの完全な一覧が含まれていることを確認します。

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

検証機能に加え、"everything"、"not" などのユーザーの表現を照合する [Term](#match-user-input-using-the-terms-attribute) 属性を追加することができます。

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

このスニペットは、前述の検証機能を使用して、ユーザーが "everything but Jalapenos" (ハラペーニョ以外のすべて) を要求した場合のボットとユーザー間の対話を示しています。 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>FormFlow の属性

以下の C# 属性をクラスに追加して、FormFlow ダイアログの動作をカスタマイズすることができます。

| Attribute | 目的 |
|----|----| 
| [Describe][describeAttribute] | テンプレートやカードにフィールドや値を表示する方法を変更します |
| [数値][numericAttribute] | 数値フィールドの許容値を制限します |
| [省略可能][optionalAttribute] | フィールドに省略可能とマークします |
| [パターン][patternAttribute] | 文字列フィールドを検証する正規表現を定義します |
| [Prompt][promptAttribute] | フィールドのプロンプトを定義します |
| [テンプレート][templateAttribute] | プロンプトまたはプロンプトの値を生成するために使用するテンプレートを定義します |
| [Terms][termsAttribute] | フィールドまたは値を照合する入力用語を定義します |

## <a name="customize-prompts-using-the-prompt-attribute"></a>Prompt 属性を使用してプロンプトをカスタマイズする

既定のプロンプトはフォームのフィールドごとに自動的に生成されますが、`Prompt` 属性を使用して任意のフィールドにカスタム プロンプトを指定することができます。 たとえば、`SandwichOrder.Sandwich` フィールドの既定プロンプトが "Please select a sandwich" (サンドイッチを選択してください) の場合、`Prompt` 属性を追加してそのフィールドのカスタム プロンプトを指定できます。

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

この例では、[パターン言語](bot-builder-dotnet-formflow-pattern-language.md)を使用して、実行時にフォーム データをプロンプトに動的に設定します。`{&}` はフィールドの説明に置き換えられ、`{||}` は列挙の選択肢の一覧に置き換えられます。 

> [!NOTE]
> 既定では、フィールドの説明はフィールドの名前から生成されます。 フィールドのカスタムの説明を指定するには、`Describe` 属性を追加します。

このスニペットは、前述の例で指定されたカスタマイズされたプロンプトを示しています。

```console
What kind of sandwich would you like?
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

`Prompt` 属性で、フォームにプロンプトを表示する方法に影響を与えるパラメーターを指定することもできます。 たとえば、`ChoiceFormat` パラメーターを指定すると、フォームの選択肢一覧の表示方法が決まります。

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

この例で、`ChoiceFormat` パラメーターの値は、選択肢が (番号付き一覧ではなく) 箇条書き一覧として表示されることを示しています。

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>Template 属性を使用してプロンプトをカスタマイズする

`Prompt` 属性を使用すると、1 つのフィールドのプロンプトをカスタマイズできますが、`Template` 属性を使用すると、FormFlow がプロンプトを自動生成する際に使用する既定のテンプレートを置き換えることができます。 このコード例では、`Template` 属性を使用して、フォームがすべての列挙フィールドを処理する方法を再定義します。 この属性は、ユーザーが 1 つの項目のみを選択できることを示し、[パターン言語](bot-builder-dotnet-formflow-pattern-language.md)を使用してプロンプト テキストを設定し、1 行に 1 項目のみを表示するように指定しています。 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

このスニペットは、`Bread` フィールドと `Cheese` フィールドの結果のプロンプトを示しています。

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

`Template` 属性を使用して、FormFlow がプロンプトの生成に使用する既定のテンプレートを置き換える場合、フォームが生成するプロンプトとメッセージに変化を加えることができます。 そのために、[パターン言語](bot-builder-dotnet-formflow-pattern-language.md)を使用して複数のテキスト文字列を定義することができます。その結果、フォームでプロンプトまたはメッセージを表示する必要があるたびに、使用できる選択肢からランダムに選択されるようになります。

このコード例では、[TemplateUsage.NotUnderstood][notUnderstood] テンプレートを再定義して 2 種類のメッセージを指定しています。 ユーザーの入力を理解できないとボットが伝える必要がある場合、ボットは 2 つのテキスト文字列のいずれかをランダムに選択してメッセージの内容を決定します。 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

このスニペットは、ボットとユーザー間の対話の結果例を示しています。 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>Optional 属性を使用してフィールドを省略可能と指定する

フィールドを省略可能と指定するには、`Optional` 属性を使用します。 このコード例は、`Cheese` フィールドを省略可能と指定しています。

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

フィールドが省略可能であり、値が指定されていない場合、現在の選択肢は "No Preference" (設定なし) と表示されます。

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

フィールドが省略可能で、ユーザーが値を指定した場合、一覧内の最後の選択肢として "No Preference" (設定なし) が表示されます。

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>Terms 属性を使用してユーザー入力を照合する

FormFlow を使用して構築されたボットにユーザーがメッセージを送信すると、ボットはユーザー入力を用語の一覧と照合することでユーザー入力の意味を特定しようと試みます 既定では、フィールドまたは値に以下の手順を適用して用語一覧を生成します。 

1. 大文字と小文字の変化とアンダースコア (_) で中断します。
2. 各 <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-gram</a> を最大長まで生成します。
3. "s?" を 各単語の末尾に追加します (複数形をサポートするため)。 

たとえば、値 "AngusBeefAndGarlicPizza" で次のような用語が生成されます。 

- 'angus?'
- 'beefs?'
- 'garlics?'
- 'pizzas?'
- 'angus? beefs?'
- 'garlics? pizzas?'
- 'angus beef and garlic pizza'

この既定の動作をオーバーライドして、ユーザー入力をフィールドまたはフィールドの値と照合するために使用される用語の一覧を定義するには、`Terms` 属性を使用します。 たとえば、`Terms` 属性 (と正規表現) を使用して、ユーザーが "rotisserie" という単語のスペルを誤る可能性を示すことができます。 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

`Terms` 属性を使用すると、ユーザー入力を有効な選択肢のいずれかと一致させることができる可能性が高くなります。 この例の `Terms.MaxPhrase` パラメーターによって、`Language.GenerateTerms` は用語の追加のバリエーションを生成します。 

このスニペットは、ユーザーが "Rotisserie" のスペルを誤ったときにボットとユーザー間の対話の結果を示しています。

```console
What kind of sandwich would you like?
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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>Numeric 属性または Pattern 属性を使用してユーザー入力を検証する

数値フィールドの許容値の範囲を制限するには、`Numeric` 属性を使用します。 このコード例では、`Numeric` 属性を使用して、`Rating` フィールドの入力を 1 から 5 の間の数値にする必要があることを指定しています。 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

特定のフィールドの値に必要な形式を指定するには、`Pattern` 属性を使用します。 このコード例では、`Pattern` 属性を使用して、`PhoneNumber` フィールドの値に必要な形式を指定しています。

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>まとめ

この記事では、FormFlow を使用してカスタマイズされたユーザー エクスペリエンスを提供する方法について説明しました。カスタマイズには、フォームの初期状態を指定し、フィールド間の相互依存関係を管理してユーザー入力を処理するビジネス ロジックを追加しました。また、プロンプトのカスタマイズ、テンプレートのオーバーライド、省略可能なフィールドの指定、ユーザー入力の照合、ユーザー入力の検証を行う属性を使用しました。 FormFlow を使用してユーザー エクスペリエンスをカスタマイズするその他の方法については、「[FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)」を参照してください。

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>その他のリソース

- [FormFlow の基本的機能](bot-builder-dotnet-formflow.md)
- [FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)
- [フォーム コンテンツのローカライズ](bot-builder-dotnet-formflow-localize.md)
- [JSON スキーマによるフォームの定義](bot-builder-dotnet-formflow-json-schema.md)
- [パターン言語によるユーザー エクスペリエンスのカスタマイズ](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
