---
title: FormBuilder によるフォームのカスタマイズ | Microsoft Docs
description: Bot Builder SDK for .NET の FormBuilder を使用して、会話のフローとコンテンツを動的に変更およびカスタマイズする方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3ad843530b70b75a6db728a399a315534d2234ac
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904195"
---
# <a name="customize-a-form-using-formbuilder"></a>FormBuilder によるフォームのカスタマイズ

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

「[FormFlow の基本的機能](bot-builder-dotnet-formflow.md)」は、非常に汎用的なユーザー エクスペリエンスを提供する基本的な FormFlow 実装について説明し、「[FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)」は、ビジネス ロジックと属性を使用してユーザー エクスペリエンスをカスタマイズする方法について説明しています。 この記事では、フォームがステップを実行するシーケンスを指定し、フィールド値、確認、およびメッセージを動的に定義することにより、[FormBuilder][formBuilder] を使用してユーザー エクスペリエンスをさらにカスタマイズする方法を説明します。 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>フィールド値、確認、およびメッセージの動的な定義

FormBuilder を使用して、フィールド値、確認、およびメッセージを動的に定義できます。

### <a name="dynamically-define-field-values"></a>フィールド値の動的な定義 

フットロング サンドウィッチを指定する任意の注文に無料の飲料またはクッキーを追加するように設計されたサンドウィッチ ボットは、無料アイテムに関するデータを保存するために `Sandwich.Specials` フィールドを使用します。 この場合、`Sandwich.Specials` フィールドの値は、注文にフットロング サンドウィッチが含まれるかどうかに応じて、注文ごとに動的に設定する必要があります。 

`Specials` フィールドはオプションとして指定され、優先順位がないことを示す [なし] が選択肢のテキストとして指定されます。

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

このコード例は、`Specials` フィールドの値を動的に設定する方法を示しています。 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

この例では、[Advanced.Field.SetType][setType] メソッドはフィールドの種類を指定します (`null` は列挙体フィールドを表します)。 [Advanced.Field.SetActive][setActive] メソッドは、サンドウィッチの長さが `Length.FootLong` である場合にのみフィールドが有効になるように指定します。 最後に、[Advanced.Field.SetDefine][setDefine] メソッドはフィールドを定義する非同期デリゲートを指定します。 デリゲートには、現在の状態オブジェクトと、動的に定義中の [Advanced.Field][field] が渡されます。 デリゲートは、動的に値を定義するためにフィールドの fluent メソッドを使用します。 この例では、値は文字列であり、`AddDescription` メソッドおよび `AddTerms` メソッドは各値の説明と用語を指定します。

> [!NOTE]
> フィールドの値を動的に定義するには、[Advanced.IField][iField] を実装するか、上の例に示すように [Advanced.FieldReflector][FieldReflector] クラスを使用してプロセスを効率化することができます。 

### <a name="dynamically-define-messages-and-confirmations"></a>メッセージと確認の動的な定義

また、FormBuilder を使用して、メッセージと確認を動的に定義することもできます。 それぞれのメッセージと確認は、フォームの前の手順が非アクティブまたは完了した場合にのみ実行されます。 

このコード例では、サンドウィッチのコストを計算する動的に生成された確認を示します。 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>FormBuilder によるフォームのカスタマイズ

このコード例では、FormBuilder を使用して、フォームの手順の定義、[選択内容の検証](bot-builder-dotnet-formflow-advanced.md#add-business-logic)、および[フィールド値と確認の動的な定義](#dynamically-define-field-values-confirmations-and-messages)を行います。 既定では、フォームでの手順はリストされているシーケンスで実行されます。 ただし、既に値が含まれているフィールドの場合、または明示的なナビゲーションが指定されている場合、手順がスキップされることがあります。 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

この例では、フォームでは次の手順が実行されます。

- ウェルカム メッセージの表示。 
- `SandwichOrder.Sandwich` の記入。 
- `SandwichOrder.Length` の記入。 
- `SandwichOrder.Bread` の記入。 
- `SandwichOrder.Cheese` の記入。 
- `SandwichOrder.Toppings` の記入および不足している値の追加 (ユーザーが `ToppingOptions.Everything` を選択した場合)。 -. 選択したトッピングを確認するメッセージの表示。 
- `SandwichOrder.Sauces` の記入。 
- `SandwichOrder.Specials` のフィールド値の[動的な定義](#dynamically-define-field-values)。 
- サンドウィッチのコスト確認の[動的な定義](#dynamically-define-messages-and-confirmations)。 
- `SandwichOrder.DeliveryAddress` の記入および結果の文字列の[確認](bot-builder-dotnet-formflow-advanced.md#add-business-logic)。 アドレスの先頭が数字でない場合、フォームはメッセージを返します。 
- カスタム プロンプトを使用した `SandwichOrder.DeliveryTime` の記入。 
- 注文の確認。 
- クラスに定義されているが、明示的に `Field` で参照されないすべての残りのフィールドの追加。 (例で `AddRemainingFields` メソッドを呼び出さなかった場合、フォームには明示的に参照されなかったフィールドは含まれません。) 
- お礼のメッセージの表示。 
- 注文を処理する `OnCompletionAsync` ハンドラーの定義。 

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>その他のリソース

- [FormFlow の基本的機能](bot-builder-dotnet-formflow.md)
- [FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)
- [フォーム コンテンツのローカライズ](bot-builder-dotnet-formflow-localize.md)
- [JSON スキーマによるフォームの定義](bot-builder-dotnet-formflow-json-schema.md)
- [パターン言語によるユーザー エクスペリエンスのカスタマイズ](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
