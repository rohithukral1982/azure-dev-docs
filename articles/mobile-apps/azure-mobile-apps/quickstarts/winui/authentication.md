---
title: Add authentication to your Windows (WinUI3) app
description: Add authentication to your Windows (WinUI3) app using Azure Mobile Apps with our tutorial.
author: adrianhall
ms.service: mobile-services
ms.topic: article
ms.date: 06/11/2022
ms.author: adhal
---

# Add authentication to your Windows (WinUI3) app

In this tutorial, you add Microsoft authentication to the TodoApp project using Azure Active Directory. Before completing this tutorial, ensure you've [created the project and deployed the backend](./index.md).

> [!TIP]
> Although we use Azure Active Directory for authentication, you can use any authentication library you wish with Azure Mobile Apps.  

[!INCLUDE [Register with AAD for the backend](~/mobile-apps/azure-mobile-apps/includes/quickstart/common/register-aad-backend.md)]

[!INCLUDE [Configure the service for authentication](~/mobile-apps/azure-mobile-apps/includes/quickstart/windows/configure-auth-backend.md)]

## Add authentication to the app

The Microsoft Datasync Framework has built-in support for any authentication provider that uses a Json Web Token (JWT) within a header of the HTTP transaction.  This application will use the [Microsoft Authentication Library (MSAL)](/azure/active-directory/develop/msal-overview) to request such a token and authorize the signed in user to the backend service.

[!INCLUDE [Configure a native app for authentication](~/mobile-apps/azure-mobile-apps/includes/quickstart/common/register-aad-client.md)]

Open the `TodoApp.sln` solution in Visual Studio and set the `TodoApp.WPF`project as the startup project.  Add the [Microsoft Identity Library (MSAL)](/azure/active-directory/develop/msal-overview) to the `TodoApp.WinUI` project:

[!INCLUDE [Set up MSAL in Windows](~/mobile-apps/azure-mobile-apps/includes/quickstart/windows/add-msal-library.md)]

Open the `MainWindow.xaml.cs` file in the `TodoApp.WinUI` project.  

Add the following `using` statements to the top of the file:

``` csharp
using Microsoft.Datasync.Client;
using Microsoft.Identity.Client;
using System.Diagnostics;
using System.Linq;
```

Adjust the constructor and fields to add a reference to the identity client as follows:

``` csharp
private readonly TodoListViewModel _viewModel;
private readonly ITodoService _service;
private IPublicClientApplication _identityClient;

public MainWindow()
{
    this.InitializeComponent();
    ResizeWindow(this, 480, 800);

    _service = new RemoteTodoService(GetAuthenticationToken);
    _viewModel = new TodoListViewModel(this, _service);
    mainContainer.DataContext = _viewModel;
}

public async Task<AuthenticationToken> GetAuthenticationToken()
{
    if (_identityClient == null) 
    {
        _identityClient = PublicClientApplicationBuilder.Create(Constants.ApplicationId)
            .WithAuthority(AzureCloudInstance.AzurePublic, "common")
            .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
            .Build();
    }
    var accounts = await _identityClient.GetAccountsAsync();
    AuthenticationResult? result = null;
    try
    {
        result = await _identityClient
            .AcquireTokenSilent(Constants.Scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        result = await _identityClient
            .AcquireTokenInteractive(Constants.Scopes)
            .ExecuteAsync();
    }
    catch (Exception ex)
    {
        // Display the error text - probably as a pop-up
        Debug.WriteLine($"Error: Authentication failed: {ex.Message}");
    }

    return new AuthenticationToken
    {
        DisplayName = result?.Account?.Username ?? "",
        ExpiresOn = result?.ExpiresOn ?? DateTimeOffset.MinValue,
        Token = result?.AccessToken ?? "",
        UserId = result?.Account?.Username ?? ""
    };
}
```

The `GetAuthenticationToken()` method works with the Microsoft Identity Library (MSAL) to get an access token suitable for authorizing the signed-in user to the backend service.  This function is then passed to the `RemoteTodoService` for creating the client.  If the authentication is successful, the `AuthenticationToken` is produced with data necessary to authorize each request.  If not, then an expired bad token is produced instead.

## Test the app

You should be able to press **F5** to run the app.  When the app runs, a browser will be opened to ask you for authentication.  The first time the app runs, you'll be asked to consent to the access:

![Screenshot of the AAD consent request.](./media/authentication-consent.png)

Press **Yes** to continue to your app.  The app will then run as before.

## Next steps

Next, configure your application to operate offline by [implementing an offline store](./offline.md).

## Further reading

* [Quickstart: Protect a web API with the Microsoft identity platform](/azure/active-directory/develop/web-api-quickstart?pivots=devlang-aspnet-core)
* [Quickstart: Acquire a token and call Microsoft Graph API from a desktop application](/azure/active-directory/develop/desktop-app-quickstart?pivots=devlang-windows-desktop)
* [Microsoft identity platform: Windows Presentation Foundation tutorial](/azure/active-directory/develop/tutorial-v2-windows-desktop)   