---
title: ボットの単体テスト - Bot Service
description: テスト フレームワークを使用してボットの単体テストを実行する方法について説明します。
keywords: ボット, ボットのテスト, ボットのテスト フレームワーク
author: gabog
ms.author: ggilaber
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: da99aa82c235bc8f530c9c2ad8d3ea1fa592b001
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798129"
---
# <a name="how-to-unit-test-bots"></a>ボットの単体テストを実行する方法

[!INCLUDE[applies-to](../includes/applies-to.md)]

このトピックでは、次の操作方法について説明します。

- ボットの単体テストを作成する
- Assert を使用して、予想した値に対してダイアログ ターンによって返されたアクティビティを確認する
- Assert を使用して、ダイアログによって返された結果を確認する
- さまざまな種類のデータ ドリブン テストを作成する
- ダイアログのさまざまな依存関係のモック オブジェクトを作成する (つまり、LUIS 認識エンジンなど)。

## <a name="prerequisites"></a>前提条件

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

このトピックで使用されている [CoreBot テスト](https://aka.ms/cs-core-test-sample) サンプルは、[Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) パッケージ、[XUnit](https://xunit.net/)、および [Moq](https://github.com/moq/moq) を参照して単体テストを作成します。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

このトピックで使用されている [CoreBot テスト](https://aka.ms/js-core-test-sample) サンプルは、[botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) パッケージおよび [Mocha](https://mochajs.org/) を参照して単体テストを作成し、[Mocha テスト エクスプローラー](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)を使用して VS Code でテスト結果を視覚化します。

---

## <a name="testing-dialogs"></a>ダイアログのテスト

Corebot サンプルでは、ダイアログの単体テストは `DialogTestClient` クラスを使用して行われます。これにより、コードを Web サービスにデプロイすることなく、ボットの外部で単独でテストするメカニズムが提供されます。

このクラスを使用すると、ダイアログの応答をターンごとに検証する単体テストを作成できます。 `DialogTestClient` クラスを使用した単体テストは、botbuilder ダイアログ ライブラリを使って作成された他のダイアログでも機能するはずです。

次の例は、`DialogTestClient` から派生したテストを示しています。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("Seattle");
Assert.Equal("Where are you traveling from?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("New York");
Assert.Equal("When would you like to travel?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("tomorrow");
Assert.Equal("OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("yes");
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);

reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

`DialogTestClient` クラスは、`Microsoft.Bot.Builder.Testing` 名前空間に定義されており、[Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) NuGet パッケージに含まれています。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

reply = await testClient.sendActivity('Seattle');
assert.strictEqual(reply.text, 'Where are you traveling from?');

reply = await testClient.sendActivity('New York');
assert.strictEqual(reply.text, 'When would you like to travel?');

reply = await testClient.sendActivity('tomorrow');
assert.strictEqual(reply.text, 'OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?');

reply = await testClient.sendActivity('yes');
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');

reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

`DialogTestClient` クラスは、[botbuilder-testing]() npm パッケージに含まれています。

---

### <a name="dialogtestclient"></a>DialogTestClient

`DialogTestClient` の最初のパラメーターはターゲット チャネルです。 これにより、ボットのターゲット チャネル (Teams、Slack、Cortana など) に基づいてさまざまなレンダリング ロジックをテストできます。 ターゲット チャネルが不明の場合は、`Emulator` または `Test` のチャネル ID を使用できますが、一部のコンポーネントでは、現在のチャネルによって動作が異なる場合があることに注意してください。たとえば、`ConfirmPrompt` は、`Test` チャネルと `Emulator` チャネルとで Yes/No のオプションを異なる方法でレンダリングします。 また、このパラメーターを使用して、チャネル ID に基づいてダイアログの条件付きレンダリング ロジックをテストすることもできます。

2番目のパラメーターは、テスト対象ダイアログのインスタンスです (注: **"sut"** は "System Under Test" (テスト対象のシステム) を表しています。この記事のコード スニペットでは、この頭文字を使用します)。

`DialogTestClient` コンストラクターには追加のパラメーターが用意されており、これを使用すると、必要に応じてクライアントの動作をさらにカスタマイズしたり、テスト対象のダイアログにパラメーターを渡したりできます。 ダイアログの初期化データを渡したり、カスタム ミドルウェアを追加したり、独自の TestAdapter と `ConversationState` インスタンスを使用したりできます。

### <a name="sending-and-receiving-messages"></a>メッセージの送受信

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

`SendActivityAsync<IActivity>` メソッドは、テキストの発話や `IActivity` をダイアログに送信することを可能にして、受信する最初のメッセージを返します。 `<T>` パラメーターは、キャストせずにアサートできるように、厳密に型指定された応答のインスタンスを返すために使用されます。

```csharp
var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);
```

一部のシナリオでは、ボットは 1 つのアクティビティへの応答で複数のメッセージを送信することがあります。この場合、`DialogTestClient` は応答をキューに入れ、ユーザーは `GetNextReply<IActivity>` メソッドを使用して、応答キューから次のメッセージをポップできます。

```csharp
reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

応答キューにこれ以上のメッセージがなくなると、`GetNextReply<IActivity>` は null 値を返します。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`sendActivity` メソッドは、テキストの発話や `Activity` をダイアログに送信することを可能にして、受信する最初のメッセージを返します。

```javascript
let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');
```

一部のシナリオでは、ボットは 1 つのアクティビティへの応答で複数のメッセージを送信することがあります。この場合、`DialogTestClient` は応答をキューに入れ、ユーザーは `getNextReply` メソッドを使用して、応答キューから次のメッセージをポップできます。

```javascript
reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

応答キューにこれ以上のメッセージがなくなると、`getNextReply` は null 値を返します。

---

### <a name="asserting-activities"></a>アクティビティのアサート

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

CoreBot サンプルのコードは、単に返されたアクティビティの `Text` プロパティをアサートします。 より複雑なボットでは `Speak`、`InputHint`、`ChannelData` などのような他のプロパティをアサートすることもできます。

```csharp
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);
Assert.Equal("One moment please...", reply.Speak);
Assert.Equal(InputHints.IgnoringInput, reply.InputHint);
```

これは、前述のように各プロパティを個別に確認することで実行できます。アクティビティをアサートするための独自のヘルパー ユーティリティを作成したり、[FluentAssertions](https://fluentassertions.com/) のような他のフレームワークを使用してカスタム アサーションを作成し、テスト コードを簡素化したりすることもできます。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

CoreBot サンプルのコードは、単に返されたアクティビティの `text` プロパティをアサートします。 より複雑なボットでは `speak`、`inputHint`、`channelData` などのような他のプロパティをアサートすることもできます。

```javascript
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');
assert.strictEqual(reply.speak, 'One moment please...');
assert.strictEqual(reply.inputHint, InputHints.IgnoringInput);
```

これは、前述のように各プロパティを個別に確認することで実行できます。アクティビティをアサートするための独自のヘルパー ユーティリティを作成したり、[Chai](https://www.chaijs.com/) のような他のライブラリを使用してカスタム アサーションを作成し、テスト コードを簡素化したりすることもできます。

---

### <a name="passing-parameters-to-your-dialogs"></a>ダイアログにパラメーターを渡す

`DialogTestClient` コンストラクターには、ダイアログにパラメーターを渡すために使用できる `initialDialogOptions` があります。 たとえば、このサンプルの `MainDialog` は、ユーザーの発話から解決するエンティティを使用して LUIS の結果から `BookingDetails` オブジェクトを初期化し、`BookingDialog` を開始するための呼び出しでこのオブジェクトを渡します。

これは、次のようにテストで実装できます。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var inputDialogParams = new BookingDetails()
{
    Destination = "Seattle",
    TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
};

var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut, inputDialogParams);

```

`BookingDialog` はこのパラメーターを受け取り、`MainDialog` から呼び出された場合と同じ方法で、テストでそれにアクセスします。

```csharp
private async Task<DialogTurnResult> DestinationStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    var bookingDetails = (BookingDetails)stepContext.Options;
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const inputDialogParams = {
    destination: 'Seattle',
    travelDate: formatDate(new Date().setDate(now.getDate() + 1))
};

const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut, inputDialogParams);
```

`BookingDialog` はこのパラメーターを受け取り、`MainDialog` から呼び出された場合と同じ方法で、テストでそれにアクセスできます。

```javascript
async destinationStep(stepContext) {
    const bookingDetails = stepContext.options;
    ...
}
```

---

### <a name="asserting-dialog-turn-results"></a>ダイアログ ターンの結果のアサート

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

`BookingDialog` や `DateResolverDialog` のような一部のダイアログは、呼び出し元のダイアログに値を返します。 `DialogTestClient` オブジェクトは、ダイアログによって返された結果を分析およびアサートするために使用できる `DialogTurnResult` プロパティを公開します。

次に例を示します。

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

...

var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
Assert.Equal("New York", bookingResults?.Origin);
Assert.Equal("Seattle", bookingResults?.Destination);
Assert.Equal("2019-06-21", bookingResults?.TravelDate);
```

`DialogTurnResult` プロパティを使用して、ウォーターフォールのステップによって返された中間結果を検査およびアサートすることもできます。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`BookingDialog` や `DateResolverDialog` のような一部のダイアログは、呼び出し元のダイアログに値を返します。 `DialogTestClient` オブジェクトは、ダイアログによって返された結果を分析およびアサートするために使用できる `dialogTurnResult` プロパティを公開します。

次に例を示します。

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

...

const bookingResults = client.dialogTurnResult.result;
assert.strictEqual('New York', bookingResults.destination);
assert.strictEqual('Seattle', bookingResults.origin);
assert.strictEqual('2019-06-21', bookingResults.travelDate);
```

`dialogTurnResult` プロパティを使用して、ウォーターフォールのステップによって返された中間結果を検査およびアサートすることもできます。

---

### <a name="analyzing-test-output"></a>テスト出力の分析

時々、テストをデバッグせずに、単体テストのトランスクリプトを読み取ってテストの実行を分析することが必要になることがあります。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

[Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) パッケージには、ダイアログで送受信されたメッセージをコンソールに記録する `XUnitDialogTestLogger` が含まれています。

このミドルウェアを使用するためには、テストでは、XUnit テスト ランナーによって提供される `ITestOutputHelper` オブジェクトを受け取るコンストラクターを公開し、`middlewares` パラメーターを通じて `DialogTestClient` に渡される `XUnitDialogTestLogger` を作成する必要があります。

```csharp
public class BookingDialogTests
{
    private readonly IMiddleware[] _middlewares;

    public BookingDialogTests(ITestOutputHelper output)
        : base(output)
    {
        _middlewares = new[] { new XUnitDialogTestLogger(output) };
    }

    [Fact]
    public async Task SomeBookingDialogTest()
    {
        // Arrange
        var sut = new BookingDialog();
        var testClient = new DialogTestClient(Channels.Msteams, sut, middlewares: _middlewares);

        ...
    }
}
```

次に示すのは、`XUnitDialogTestLogger` が構成されているときに出力ウィンドウに記録する内容の例です。

![XUnitMiddlewareOutput](media/how-to-unit-test/cs/XUnitMiddlewareOutput.png)

XUnit を使用しているときのコンソールへのテスト出力の送信に関する追加情報については、XUnit ドキュメントの「[出力のキャプチャ](https://xunit.net/docs/capturing-output.html)」を参照してください。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) パッケージには、ダイアログで送受信されたメッセージをコンソールに記録する `DialogTestLogger` が含まれています。

このミドルウェアを使用するには、単に `middlewares` パラメーターを通じて `DialogTestClient` にそれを渡すだけです。

```javascript
const client = new DialogTestClient('msteams', sut, testData.initialData, [new DialogTestLogger()]);
```

次に示すのは、`DialogTestLogger` が構成されているときに出力ウィンドウに記録する内容の例です。

![DialogTestLoggerOutput](media/how-to-unit-test/js/DialogTestLoggerOutput.png)

---

この出力は、継続的インテグレーションのビルド中にビルド サーバーにも記録され、ビルドの失敗を分析するのに役立ちます。

## <a name="data-driven-tests"></a>データ ドリブン テスト

多くの場合、ダイアログ ロジックは変わらず、会話のさまざまな実行パスはユーザー発話に基づきます。 会話のバリアントごとに 1 つの単体テストを作成するよりも、データ ドリブン テスト (パラメーター化テストとも呼ばれる) を使用した方が簡単です。

たとえば、このドキュメントの「概要」セクションにあるサンプル テストは、1 つの実行フローをテストする方法を示していますが、ユーザーが確認に対して「いいえ」と言った場合や、異なる日付が使用されている場合はどうなるでしょうか。

データ ドリブン テストでは、テストを書き直さずに、これらのすべての変更をテストできます。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

CoreBot サンプルでは、XUnit からの `Theory` テストを使用してテストをパラメーター化します。

### <a name="theory-tests-using-inlinedata"></a>InlineData を使用した理論テスト

次のテストでは、ユーザーが "cancel" と言ったときにダイアログがキャンセルされることを検査します。

```csharp
[Fact]
public async Task ShouldBeAbleToCancel()
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>("cancel");
    Assert.Equal("Cancelling...", reply.Text);
}
```

ダイアログをキャンセルするには、ユーザーは "quit"、"never mind"、および "stop it" と入力できます。 可能なすべての単語に対して新しいテスト ケースを作成するのではなく、`InlineData` 値の一覧を通じてパラメーターを受け入れる 1 つの `Theory` テスト メソッドを作成して、各テスト ケースのパラメーターを定義します。

```csharp
[Theory]
[InlineData("cancel")]
[InlineData("quit")]
[InlineData("never mind")]
[InlineData("stop it")]
public async Task ShouldBeAbleToCancel(string cancelUtterance)
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, middlewares: _middlewares);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>(cancelUtterance);
    Assert.Equal("Cancelling...", reply.Text);
}
```

新しいテストは、異なるパラメーターを使用して 4 回実行され、各ケースは、Visual Studio テスト エクスプローラーで `ShouldBeAbleToCancel` テストの下に子項目として表示されます。 以下に示すように、それらのいずれかが失敗した場合は、テスト一式を再実行するのではなく、失敗したシナリオを右クリックしてデバッグできます。

![InlineDataTestResults](media/how-to-unit-test/cs/InlineDataTestResults.png)

### <a name="theory-tests-using-memberdata-and-complex-types"></a>MemberData と複合型を使用した理論テスト

`InlineData` は、単純な値の型のパラメーター (文字列、整数など) を受け取る小規模なデータ ドリブン テストに役立ちます。

`BookingDialog` は `BookingDetails` オブジェクトを受け取って、新しい `BookingDetails` オブジェクトを返します。 このダイアログのテストのパラメーター化されていないバージョンは、次のようになります。

```csharp
[Fact]
public async Task DialogFlow()
{
    // Initial parameters
    var initialBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = null,
        TravelDate = null,
    };

    // Expected booking details
    var expectedBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = "New York",
        TravelDate = "2019-06-25",
    };

    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, initialBookingDetails);

    // Act/Assert
    var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
    ...

    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(expectedBookingDetails.Origin, bookingResults?.Origin);
    Assert.Equal(expectedBookingDetails.Destination, bookingResults?.Destination);
    Assert.Equal(expectedBookingDetails.TravelDate, bookingResults?.TravelDate);
}
```

このテストをパラメーター化するために、テスト ケース データを含む `BookingDialogTestCase` クラスを作成しました。 これには、初期 `BookingDetails` オブジェクト、予想される `BookingDetails`、およびユーザーから送信された発話を含む文字列の配列と、ターンごとのダイアログからの予想される応答が含まれています。

```csharp
public class BookingDialogTestCase
{
    public BookingDetails InitialBookingDetails { get; set; }

    public string[,] UtterancesAndReplies { get; set; }

    public BookingDetails ExpectedBookingDetails { get; set; }
}
```

また、テストで使用されるテスト ケースのコレクションを返す `IEnumerable<object[]> BookingFlows()` メソッドを公開するヘルパー `BookingDialogTestsDataGenerator` クラスも作成しました。

Visual Studio テスト エクスプローラーで各テスト ケースを別々の項目として表示するために、XUnit テスト ランナーでは、`BookingDialogTestCase` のような複合型が `IXunitSerializable` を実装することが要求されます。これを簡素化するために、Bot.Builder.Testing フレームワークでは、このインターフェイスを実装し、`IXunitSerializable` を実装する必要なしにテスト ケース データをラップするために使用できる `TestDataObject` クラスが提供されています。 

これらの 2 つのクラスの使用方法を示す `IEnumerable<object[]> BookingFlows()` のフラグメントを次に示します。

```csharp
public static class BookingDialogTestsDataGenerator
{
    public static IEnumerable<object[]> BookingFlows()
    {
        // Create the first test case object
        var testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails(),
            UtterancesAndReplies = new[,]
            {
                { "hi", "Where would you like to travel to?" },
                { "Seattle", "Where are you traveling from?" },
                { "New York", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            }, 
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };

        // Create the second test case object
        testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = null,
            },
            UtterancesAndReplies = new[,]
            {
                { "hi", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            },
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };
    }
}
```

テスト データを格納するためのオブジェクトと、テスト　ケースのコレクションを公開するクラスを作成したら、`InlineData` ではなく XUnit の `MemberData` 属性を使用してデータをテストにフィードします。`MemberData` の最初のパラメーターは、テスト ケースのコレクションを返す静的関数の名前で、2 番目のパラメーターは、このメソッドを公開するクラスの型です。

```csharp
[Theory]
[MemberData(nameof(BookingDialogTestsDataGenerator.BookingFlows), MemberType = typeof(BookingDialogTestsDataGenerator))]
public async Task DialogFlowUseCases(TestDataObject testData)
{
    // Get the test data instance from TestDataObject
    var bookingTestData = testData.GetObject<BookingDialogTestCase>();
    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, bookingTestData.InitialBookingDetails);

    // Iterate over the utterances and replies array.
    for (var i = 0; i < bookingTestData.UtterancesAndReplies.GetLength(0); i++)
    {
        var reply = await testClient.SendActivityAsync<IMessageActivity>(bookingTestData.UtterancesAndReplies[i, 0]);
        Assert.Equal(bookingTestData.UtterancesAndReplies[i, 1], reply?.Text);
    }

    // Assert the resulting BookingDetails object
    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Origin, bookingResults?.Origin);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Destination, bookingResults?.Destination);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.TravelDate, bookingResults?.TravelDate);
}
```

`DialogFlowUseCases` テストが実行されたときの Visual Studio テスト エクスプローラーでのテストの結果の例を次に示します。

![BookingDialogTests](media/how-to-unit-test/cs/BookingDialogTestsResults.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="simple-data-driven-tests"></a>単純なデータ ドリブン テスト

次のテストでは、ユーザーが "cancel" と言ったときにダイアログがキャンセルされることを検査します。

```javascript
describe('ShouldBeAbleToCancel', () => {
    it('Should cancel', async () => {
        const sut = new TestCancelAndHelpDialog();
        const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

        // Execute the test case
        let reply = await client.sendActivity('Hi');
        assert.strictEqual(reply.text, 'Hi there');
        assert.strictEqual(client.dialogTurnResult.status, 'waiting');

        reply = await client.sendActivity('cancel');
        assert.strictEqual(reply.text, 'Cancelling...');
    });
});
```

あとでキャンセルのための他の発話 ("quit"、"never mind"、および "stop it" など) を処理できる必要があることを考慮します。 それぞれの新しい発話用にさらに 3 つの反復テストを作成するのではなく、発話の一覧を使用して各テスト ケースのパラメーターを定義するようにテストをリファクタリングできます。

```javascript
describe('ShouldBeAbleToCancel', () => {
    const testCases = ['cancel', 'quit', 'never mind', 'stop it'];

    testCases.map(testData => {
        it(testData, async () => {
            const sut = new TestCancelAndHelpDialog();
            const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

            // Execute the test case
            let reply = await client.sendActivity('Hi');
            assert.strictEqual(reply.text, 'Hi there');
            assert.strictEqual(client.dialogTurnResult.status, 'waiting');

            reply = await client.sendActivity(testData);
            assert.strictEqual(reply.text, 'Cancelling...');
        });
    });
});
```

新しいテストは、異なるパラメーターを使用して 4 回実行され、各ケースは、[Mocha テスト エクスプローラー](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)で `ShouldBeAbleToCancel` テスト スイートの下に子項目として表示されます。 以下に示すように、それらのいずれかが失敗した場合は、テスト一式を再実行するのではなく、失敗したシナリオを実行してデバッグできます。

![SimpleCancelTestResults](media/how-to-unit-test/js/SimpleCancelTestResults.png)

### <a name="data-driven-tests-with-complex-types"></a>複合型を使用したデータ ドリブン テスト

単純な発話一覧を使用することは、単純な値の型のパラメーター (文字列、整数など) や小さなオブジェクトを受け取る小規模なデータ ドリブン テストに役立ちます。

`BookingDialog` は `BookingDetails` オブジェクトを受け取って、新しい `BookingDetails` オブジェクトを返します。 このダイアログのテストのパラメーター化されていないバージョンは、次のようになります。

```javascript
describe('BookingDialog', () => {
    it('Returns expected booking details', async () => {
        // Initial parameters
        const initialBookingDetails = {
            origin: 'Seattle',
            destination: undefined,
            travelDate: undefined
        };

        // Expected booking details
        const expectedBookingDetails = {
            origin: 'Seattle',
            destination: 'New York',
            travelDate: '2019-06-25'
        };

        const sut = new BookingDialog('bookingDialog');
        const client = new DialogTestClient('test', sut, initialBookingDetails, [new DialogTestLogger()]);

        // Execute the test case
        const reply = await client.sendActivity('Hi');
        ...

        // Check dialog results
        const result = client.dialogTurnResult.result;
        assert.strictEqual(result.destination, expectedBookingDetails.destination);
        assert.strictEqual(result.origin, expectedBookingDetails.origin);
        assert.strictEqual(result.travelDate, expectedBookingDetails.travelDate);
    });
});
```

このテストをパラメーター化するために、テスト ケース データを返す `bookingDialogTestCases` モジュールを作成しました。 各項目には、テスト ケース名、初期 "BookingDetails" オブジェクト、予想される "BookingDetails"、およびユーザーから送信された発話を含む文字列の配列と、各ターンでのダイアログからの予想される応答が含まれています。

```javascript
module.exports = [
    // Create the first test case object
    {
        name: 'Full flow',
        initialData: {},
        steps: [
            ['hi', 'To what city would you like to travel?'],
            ['Seattle', 'From what city will you be travelling?'],
            ['New York', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    },
    // Create the second test case object
    {
        name: 'Destination and Origin provided',
        initialData: {
            destination: 'Seattle',
            origin: 'New York'
        },
        steps: [
            ['hi', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedStatus: 'complete',
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    }
];
```

テスト データを含む一覧を作成したら、この一覧を個々のテスト ケースにマップするようにテストをリファクタリングできます。

```javascript
describe('DialogFlowUseCases', () => {
    const testCases = require('./testData/bookingDialogTestCases.js');

    testCases.map(testData => {
        it(testData.name, async () => {
            const sut = new BookingDialog('bookingDialog');
            const client = new DialogTestClient('test', sut, testData.initialData, [new DialogTestLogger()]);

            // Execute the test case
            for (let i = 0; i < testData.steps.length; i++) {
                const reply = await client.sendActivity(testData.steps[i][0]);
                assert.strictEqual((reply ? reply.text : null), testData.steps[i][1]);
            }

            // Check dialog results
            const actualResult = client.dialogTurnResult.result;
            assert.strictEqual(actualResult.destination, testData.expectedResult.destination);
            assert.strictEqual(actualResult.origin, testData.expectedResult.origin);
            assert.strictEqual(actualResult.travelDate, testData.expectedResult.travelDate);
        });
    });
});
```

`DialogFlowUseCases` テスト スイートが実行されたときの Mocha テスト エクスプローラーでのテスト スイートの結果の例を次に示します。

![BookingDialogTests](media/how-to-unit-test/js/BookingDialogTestsResults.png)

---

## <a name="using-mocks"></a>モックの使用

現在テストされていない部分に対しては、モック要素を使用できます。 なお、このレベルは、一般的に単体テストおよび統合テストとして考えることができます。

できるだけ多くの要素のモックを作成すると、テスト対象の各部分をより効果的に分離できます。 モック要素の候補としては、ストレージ、アダプター、ミドルウェア、アクティビティ パイプライン、チャネルなど、お使いのボットに直接含まれないものが挙げられます。 これには、テストしているボットの部分に関係ないミドルウェアなどの特定の側面を一時的に削除して、各部分を分離することが必要になる場合もあります。 ただし、ミドルウェアをテストする場合は、代わりにお使いのボットのモックを作成することもできます。

要素のモック作成では、要素を別の既知のオブジェクトで置き換えたり、最小限の Hello World 機能を実装したりといった、複数の形をとることができます。 その要素が不要な場合は単純に削除したり、単にその要素が何も行わないよう強制したりすることもできます。

モックを使用すると、ダイアログの依存関係を構成できます。また、テストの実行中に、データベース、LUIS モデル、またはその他のオブジェクトなどの外部リソースに依存せずに、それらが既知の状態であることを保証できます。

ダイアログのテストを容易にして、外部オブジェクトへの依存関係を減らすために、ダイアログ コンストラクターに外部依存関係を挿入する必要がある場合があります。

たとえば、`MainDialog` で `BookingDialog` をインスタンス化する代わりに、次のようにします。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog()
    : base(nameof(MainDialog))
{
    ...
    AddDialog(new BookingDialog());
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor() {
    super('MainDialog');
    ...
    this.addDialog(new BookingDialog('bookingDialog'));
    ...
}
```

---

`BookingDialog` のインスタンスをコンストラクター パラメーターとして渡します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog(BookingDialog bookingDialog)
    : base(nameof(MainDialog))
{
    ...
    AddDialog(bookingDialog);
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(bookingDialog) {
    super('MainDialog');
    ...
    this.addDialog(bookingDialog);
    ...
}
```

---

これにより、`BookingDialog` インスタンスをモック オブジェクトに置き換えて、実際の `BookingDialog` クラスを呼び出さずに `MainDialog` の単体テストを作成できます。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object
var mockDialog = new Mock<BookingDialog>();

// Use the mock object to instantiate MainDialog
var sut = new MainDialog(mockDialog.Object);

var testClient = new DialogTestClient(Channels.Test, sut);
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create the mock object
const mockDialog = new MockBookingDialog();

// Use the mock object to instantiate MainDialog
const sut = new MainDialog(mockDialog);

const testClient = new DialogTestClient('test', sut);
```

---

### <a name="mocking-dialogs"></a>ダイアログのモックの作成

前述のように、`MainDialog` は `BookingDialog` を呼び出して `BookingDetails` オブジェクトを取得します。 次のように、`BookingDialog` のモック インスタンスを実装して構成します。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object for BookingDialog.
var mockDialog = new Mock<BookingDialog>();
mockDialog
    .Setup(x => x.BeginDialogAsync(It.IsAny<DialogContext>(), It.IsAny<object>(), It.IsAny<CancellationToken>()))
    .Returns(async (DialogContext dialogContext, object options, CancellationToken cancellationToken) =>
    {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dialogContext.Context.SendActivityAsync($"{mockDialogNameTypeName} mock invoked", cancellationToken: cancellationToken);

        // Create the BookingDetails instance we want the mock object to return.
        var expectedBookingDialogResult = new BookingDetails()
        {
            Destination = "Seattle",
            Origin = "New York",
            TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dialogContext.EndDialogAsync(expectedBookingDialogResult, cancellationToken);
    });

// Create the sut (System Under Test) using the mock booking dialog.
var sut = new MainDialog(mockDialog.Object);
```

この例では、[Moq](https://github.com/moq/moq) を使用してモック ダイアログを作成し、`Setup` メソッドと `Returns` メソッドを使用してその動作を構成しました。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
class MockBookingDialog extends BookingDialog {
    constructor() {
        super('bookingDialog');
    }

    async beginDialog(dc, options) {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dc.context.sendActivity(`${ this.id } mock invoked`);

        // Create the BookingDetails instance we want the mock object to return.
        const bookingDetails = {
            origin: 'New York',
            destination: 'Seattle',
            travelDate: '2025-07-08'
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dc.endDialog(bookingDetails);
    }
}
...
// Create the sut (System Under Test) using the mock booking dialog.
const sut = new MainDialog(new MockBookingDialog());
...

```

この例では、`BookingDialog` から抽出し、`beginDialog` メソッドをオーバーライドして、基になるダイアログ ロジックをバイパスすることによってモック ダイアログを実装します。

---

### <a name="mocking-luis-results"></a>LUIS 結果のモックの作成

単純なシナリオでは、次のようにコードを使用してモック LUIS 結果を実装できます。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        var luisResult = new FlightBooking
        {
            Intents = new Dictionary<FlightBooking.Intent, IntentScore>
            {
                { FlightBooking.Intent.BookFlight, new IntentScore() { Score = 1 } },
            },
            Entities = new FlightBooking._Entities(),
        };
        return Task.FromResult(luisResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript

// Create a mock class for the recognizer that overrides executeLuisQuery.
class MockFlightBookingRecognizer extends FlightBookingRecognizer {
    constructor(mockResult) {
        super();
        this.mockResult = mockResult;
    }

    async executeLuisQuery(context) {
        return this.mockResult;
    }
}
...
// Create a mock result from a string
const mockLuisResult = JSON.parse(`{"intents": {"BookFlight": {"score": 1}}, "entities": {"$instance": {}}}`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
...
```

---

しかし、LUIS の結果は複雑な場合があります。その場合は、JSON ファイルに目的の結果をキャプチャし、それをリソースとしてプロジェクトに追加して、LUIS 結果に逆シリアル化した方が簡単です。 たとえば次のようになります。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        // Deserialize the LUIS result from embedded json file in the TestData folder.
        var bookingResult = GetEmbeddedTestData($"{GetType().Namespace}.TestData.FlightToMadrid.json");

        // Return the deserialized LUIS result.
        return Task.FromResult(bookingResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create a mock result from a json file
const mockLuisResult = require(`./testData/FlightToMadrid.json`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
```

---

## <a name="additional-information"></a>関連情報

- [CoreBot テスト サンプル (C#)](https://aka.ms/cs-core-test-sample)
- [CoreBot テスト サンプル (JavaScript)](https://aka.ms/js-core-test-sample)
- [Bot テスト](https://github.com/microsoft/botframework-sdk/blob/master/specs/testing/testing.md)
