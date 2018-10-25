---
title: .NET Core を使用したアクティビティの認証 | Microsoft Docs
description: .NET Core を使用してボットのアクティビティを認証する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f3aa350cbada77bd9e423a1910f93440a7a1682d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996989"
---
# <a name="authenticating-activities-using-net-core"></a>.NET Core を使用したアクティビティの認証

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[.NET Core](/dotnet/core/index) を使ってボットを開発する場合は、[Bot Framework Connector](bot-builder-dotnet-connector.md) を使ってボットとの間で[アクティビティ](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) メッセージを送受信できます。 Connector サービスを使うには、対象にしているフレームワークのバージョンに適した認証モデルを設定する必要があります。

Bot Framework Connector.AspNetCore では、次のバージョンの ASP.NET がサポートされています。
* .NET Core v1.1/AspNetCore1.x の場合、**Microsoft.Bot.Connector.AspNetCore 1.x** を使用します
* .NET Core v2.0/AspNetCore2.x の場合、**Microsoft.Bot.Connector.AspNetCore 2.x** を使用します

この記事では、ボットの対象である特定のフレームワークに対して認証モデルを設定する方法を説明します。

## <a name="prerequisites"></a>前提条件

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows) 対象とするバージョンの .NET Core をインストールします (例: .NET Core 1.1 または .NET Core v2.0)。
* [ボットを登録します](~/bot-service-quickstart-registration.md)。 ボットを登録して、認証プロセスに必要なアプリ ID とパスワードを取得します。

## <a name="create-a-net-core-project"></a>.NET Core プロジェクトを作成する

.NET Core プロジェクトを作成するには、次のようにします。

1. Visual Studio 2017 を開き、**[ファイル] > [新規作成] > [プロジェクト...]** の順にクリックします。
2. **[Visual C#]** ノードを展開し、**[.NET Core]** をクリックします。
3. **[ASP.NET Core Web アプリケーション]** プロジェクトの種類を選択し、プロジェクト情報を入力します (例: 名前、場所、ソリューション名などのフィールド)。
4. Click **OK**.
5. プロジェクトの対象が *.NET Core* および *ASP.NET Core* の希望のバージョンであることを確認します。 たとえば、次のスクリーンショットでは、プロジェクトの対象は **.NET Core** と **ASP.NET Core 2.0** です。

![ASP.NET Core v2.0 プロジェクトを作成する](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > **ASP.NET Core 2.0** を対象にする場合は、コンピューターにインストールされているバージョンが 2.x 以降であることを確認します。

6. **[Web API]** のプロジェクトの種類を選択します。
7. **[OK]** をクリックしてプロジェクトを作成します。

## <a name="download-the-nuget-package"></a>NuGet パッケージをダウンロードする

**Bot Framework Connector** を使うには、.NET Core のバージョンに対して適切な NuGet パッケージをインストールします。 NuGet パッケージをインストールするには、次のようにします。

1. **ソリューション エクスプローラー**でプロジェクト名を右クリックし、**[NuGet パッケージの管理...]** をクリックします
2. **[参照]** をクリックして、**Connector.ASPNetCore** を探します。 
3. 対象にするバージョンを選択します。 たとえば、次のスクリーンショットでは、バージョン **2.0.0.3** を選択しています。 バージョン 1.1.3.2 をインストールするには、**[バージョン]** ドロップダウンをクリックし、代わりにバージョン **1.1.3.2** を選択します。
![ASP Net Core バージョン 2.0.0.3 の NuGet パッケージ](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. **[インストール]** をクリックします。

## <a name="update-the-appsettingsjson"></a>appsettings.json を更新する

Bot Framework Connector では、ボットを認証するためにアプリ ID とパスワードが必要です。 これらの値は、Web アプリ プロジェクトの **appsettings.json** で設定できます。

> [!NOTE]
> ボットの **AppID** と **AppPassword** を見つけるには、「[MicrosoftAppID と MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」を参照してください。

### <a name="appsettingsjson-for-net-core-v11"></a>.NET Core v1.1 の appsettings.json:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>.NET Core v2.0 の appsettings.json:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>startup.cs クラスを更新する

**startup.cs** クラスを更新します。 プロジェクトのバージョンに応じて、認証が動作するように適切なコードを更新します。

### <a name="startupcs-for-net-core-v11"></a>.NET Core v1.1 の startup.cs

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>.NET Core v2.0 の startup.cs:

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>MessagesController.cs クラスを作成する

**ソリューション エクスプローラー**で、**MessagesController.cs** という名前の新しい空のクラスを追加します。 **MessagesController.cs** クラスで、**Post** メソッドを次のコードで更新します。 これにより、ボットはユーザーのメッセージを送受信できるようになります。

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
