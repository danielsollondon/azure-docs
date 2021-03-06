---
title: ActiveDirectoryInteractive connects to SQL | Microsoft Docs
description: "C# Code example, with explanations, for connecting to Azure SQL Database by using SqlAuthenticationMethod.ActiveDirectoryInteractive mode."
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: active directory
ms.devlang:
ms.topic: conceptual
author: GithubMirek
ms.author: MirekS
ms.reviewer: GeneMi
ms.date: 01/25/2019
manager: craigg
---
# Connect to Azure SQL Database with Active Directory MFA

This article provides a C# program that connects to your Microsoft Azure SQL Database. The program uses interactive mode authentication, which supports [Azure Active Directory (Azure AD) multi-factor authentication (MFA)](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks).

For more information about MFA support for SQL tools, see [Azure Active Directory support in SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/azure-active-directory).

## Multi-factor authentication for Azure SQL Database

Starting in .NET Framework version 4.7.2, the enum [`SqlAuthenticationMethod`](https://docs.microsoft.com/dotnet/api/system.data.sqlclient.sqlauthenticationmethod) has a new value - `ActiveDirectoryInteractive`. In a client C# program, the enum value directs the system to use the Azure AD interactive mode supporting MFA to connect to an Azure SQL Database. The user who runs the program sees the following dialog boxes:

* A dialog box that displays an Azure AD user name and asks for the user's password.

   If the user's domain is federated with Azure AD, then this dialog box doesn't appear as no password is needed.

   If the Azure AD policy imposes MFA on the user, then the next two dialog boxes are displayed.

* The first time a user goes through MFA, the system displays a dialog box asking for a mobile phone number to send text messages to. Each message provides the *verification code* that the user must enter in the next dialog box.

* A dialog box asking for an MFA verification code, which the system has sent to a mobile phone.

For information about how to configure Azure AD to require MFA, see [Getting started with Azure Multi-Factor Authentication in the cloud](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-cloud).

For screenshots of these dialogs, see [Configure multi-factor authentication for SQL Server Management Studio and Azure AD](sql-database-ssms-mfa-authentication-configure.md).

> [!TIP]
> You can search .NET Framework APIs with our **.NET API Browser** tool page:
>
> [https://docs.microsoft.com/dotnet/api/](https://docs.microsoft.com/dotnet/api/)
>
> You can also search directly with the optional **?term=&lt;search value&gt;** parameter:
>
> [https://docs.microsoft.com/dotnet/api/?term=SqlAuthenticationMethod](https://docs.microsoft.com/dotnet/api/?term=SqlAuthenticationMethod)

## Configure your C# application in the Azure portal

Before you begin, you should have an [Azure SQL Database server](sql-database-get-started-portal.md) created and available.

### Register your app and set permissions


To use Azure AD authentication, your C# program has to register as an Azure AD application. To register an app, you need to be either an Azure AD admin or a user assigned the Azure AD *Application Developer* role. For more information about assigning roles, see [Assign administrator and non-administrator roles to users with Azure Active Directory.](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal)

 Completing an app registration generates and displays an **Application ID**. Your program has to include this ID to connect.


To register and set necessary permissions for your application:

1. Azure portal > **Azure Active Directory** > **App registrations** > **New application registration**

    ![App registration](media/active-directory-interactive-connect-azure-sql-db/image1.png)

    Once the app registration is created, the **Application ID** value is generated and displayed.

    ![App ID displayed](media/active-directory-interactive-connect-azure-sql-db/image2.png)

2. **Registered app** > **Settings** > **Required permissions** > **Add**

    ![Permissions settings for registered app](media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-c32.png)

3. **Required permissions** > **Add** > **Select an API** > **Azure SQL Database**

    ![Add access to API for Azure SQL Database](media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-Azure-sql-db-d11.png)

4. **API access** > **Select permissions** > **Delegated permissions**

    ![Delegate permissions to API for Azure SQL Database](media/active-directory-interactive-connect-azure-sql-db/sshot-add-api-access-azure-sql-db-delegated-permissions-checkbox-e14.png)

### Set an Azure AD admin for your SQL Database server


For your C# program to run, an Azure SQL server admin needs to assign an Azure AD admin for your Azure SQL server. 

 * **SQL Server** > **Active Directory admin** > **Set admin**

For more information about Azure AD admins and users for Azure SQL Database, see the screenshots in [Configure and manage Azure Active Directory authentication with SQL Database](sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server), in its section **Provision an Azure Active Directory administrator for your Azure SQL Database server**.

### Add a non-admin user to a specific database (optional)

An Azure AD admin for a SQL Database server can run the C# example program. An Azure AD user can run the program if they are in the database. An Azure AD SQL admin or an Azure AD user who exists already in the database and has the `ALTER ANY USER` permission on the database can add a user.

You can add a user to the database with the SQL [`Create User`](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql?view=sql-server-2017) command. For example, `CREATE USER [<username>] FROM EXTERNAL PROVIDER`.

For more information, see [Use Azure Active Directory Authentication for authentication with SQL Database, Managed Instance, or SQL Data Warehouse](sql-database-aad-authentication.md).

## New authentication enum value

The C# example relies on the [`System.Data.SqlClient`](https://docs.microsoft.com/dotnet/api/system.data.sqlclient) namespace. Of special interest for multi-factor authentication is the enum `SqlAuthenticationMethod`, which has the following values:

- `SqlAuthenticationMethod.ActiveDirectoryInteractive`

   Use this value with an Azure AD user name to implement MFA. This value is the focus of the present article. It produces an interactive experience by displaying dialogs for the user password, and then for MFA validation if MFA is imposed on this user. This value is available starting with .NET Framework version 4.7.2.

- `SqlAuthenticationMethod.ActiveDirectoryIntegrated`

  Use this value for a *federated* account. For a federated account, the user name is known to the Windows domain. This authentication method doesn't support MFA.

- `SqlAuthenticationMethod.ActiveDirectoryPassword`

  Use this value for authentication that requires an Azure AD user name and password. Azure SQL Database does the authentication. This method doesn't support MFA.

## Set C# parameter values from the Azure portal

For the C# program to successfully run, you need to assign proper values to static fields. Shown here are fields with example values. Also shown are the Azure portal locations where you can obtain the needed values:

| Static field name | Example value | Where in Azure portal |
| :---------------- | :------------ | :-------------------- |
| Az_SQLDB_svrName | "my-sqldb-svr.database.windows.net" | **SQL servers** > **Filter by name** |
| AzureAD_UserID | "auser@abc.onmicrosoft.com" | **Azure Active Directory** > **User** > **New guest user** |
| Initial_DatabaseName | "myDatabase" | **SQL servers** > **SQL databases** |
| ClientApplicationID | "a94f9c62-97fe-4d19-b06d-111111111111" | **Azure Active Directory** > **App registrations** > **Search by name** > **Application ID** |
| RedirectUri | new Uri("https://mywebserver.com/") | **Azure Active Directory** > **App registrations** > **Search by name** > *[Your-App-regis]* > **Settings** > **RedirectURIs**<br /><br />For this article, any valid value is fine for RedirectUri, as it isn't used here. |
| &nbsp; | &nbsp; | &nbsp; |

## Verify with SQL Server Management Studio (SSMS)

Before you run the C# program, it's a good idea to check that your setup and configurations are correct in SSMS. Any C# program failure can then be narrowed to source code.

### Verify SQL Database firewall IP addresses

Run SSMS from the same computer, in the same building, where you plan to run the C# program. For this test, any **Authentication** mode is okay. If there's any indication that the database server firewall isn't accepting your IP address, see [Azure SQL Database server-level and database-level firewall rules](sql-database-firewall-configure.md) for help.

### Verify Azure Active Directory MFA

Run SSMS again, this time with **Authentication** set to **Active Directory - Universal with MFA support**. This option requires SSMS version 17.5 or later.

For more information, see [Configure multi-factor authentication for SSMS and Azure AD](sql-database-ssms-mfa-authentication-configure.md).

> [!NOTE]
> If you are a guest user in the database, you also need to provide the Azure AD  domain name for the database - **Options** > **AD domain name or tenant ID**. To find the domain name in the Azure portal, select **Azure Active Directory** > **Custom domain names**. In the C# example program, providing a domain name is not necessary.

## C# code example

The example C# program relies on the [*Microsoft.IdentityModel.Clients.ActiveDirectory*](https://docs.microsoft.com/dotnet/api/microsoft.identitymodel.clients.activedirectory) DLL assembly.

To install this package, in Visual Studio, select **Project** > **Manage NuGet Packages**. Search for and install **Microsoft.IdentityModel.Clients.ActiveDirectory**.

### C# source code

```csharp

using System;

// Reference to Azure AD authentication assembly
using Microsoft.IdentityModel.Clients.ActiveDirectory;

using DA = System.Data;
using SC = System.Data.SqlClient;
using AD = Microsoft.IdentityModel.Clients.ActiveDirectory;
using TX = System.Text;
using TT = System.Threading.Tasks;

namespace ADInteractive5
{
    class Program
    {
        // ASSIGN YOUR VALUES TO THESE STATIC FIELDS !!
        static public string Az_SQLDB_svrName = "<Your SQL DB server>";
        static public string AzureAD_UserID = "<Your User ID>";
        static public string Initial_DatabaseName = "<Your Database>";
        // Some scenarios do not need values for the following two fields:
        static public readonly string ClientApplicationID = "<Your App ID>";
        static public readonly Uri RedirectUri = new Uri("<Your URI>");

        public static void Main(string[] args)
        {
            var provider = new ActiveDirectoryAuthProvider();

            SC.SqlAuthenticationProvider.SetProvider(
                SC.SqlAuthenticationMethod.ActiveDirectoryInteractive,
                //SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated,  // Alternatives.
                //SC.SqlAuthenticationMethod.ActiveDirectoryPassword,
                provider);

            Program.Connection();
        }

        public static void Connection()
        {
            SC.SqlConnectionStringBuilder builder = new SC.SqlConnectionStringBuilder();

            // Program._  static values that you set earlier.
            builder["Data Source"] = Program.Az_SQLDB_svrName;
            builder.UserID = Program.AzureAD_UserID;
            builder["Initial Catalog"] = Program.Initial_DatabaseName;

            // This "Password" is not used with .ActiveDirectoryInteractive.
            //builder["Password"] = "<YOUR PASSWORD HERE>";

            builder["Connect Timeout"] = 15;
            builder["TrustServerCertificate"] = true;
            builder.Pooling = false;

            // Assigned enum value must match the enum given to .SetProvider().
            builder.Authentication = SC.SqlAuthenticationMethod.ActiveDirectoryInteractive;
            SC.SqlConnection sqlConnection = new SC.SqlConnection(builder.ConnectionString);

            SC.SqlCommand cmd = new SC.SqlCommand(
                "SELECT '******** MY QUERY RAN SUCCESSFULLY!! ********';",
                sqlConnection);

            try
            {
                sqlConnection.Open();
                if (sqlConnection.State == DA.ConnectionState.Open)
                {
                    var rdr = cmd.ExecuteReader();
                    var msg = new TX.StringBuilder();
                    while (rdr.Read())
                    {
                        msg.AppendLine(rdr.GetString(0));
                    }
                    Console.WriteLine(msg.ToString());
                    Console.WriteLine(":Success");
                }
                else
                {
                    Console.WriteLine(":Failed");
                }
                sqlConnection.Close();
            }
            catch (Exception ex)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Connection failed with the following exception...");
                Console.WriteLine(ex.ToString());
                Console.ResetColor();
            }
        }
    } // EOClass Program .

    /// <summary>
    /// SqlAuthenticationProvider - Is a public class that defines 3 different Azure AD
    /// authentication methods.  The methods are supported in the new .NET 4.7.2 .
    ///  . 
    /// 1. Interactive,  2. Integrated,  3. Password
    ///  . 
    /// All 3 authentication methods are based on the Azure
    /// Active Directory Authentication Library (ADAL) managed library.
    /// </summary>
    public class ActiveDirectoryAuthProvider : SC.SqlAuthenticationProvider
    {
        // Program._ more static values that you set!
        private readonly string _clientId = Program.ClientApplicationID;
        private readonly Uri _redirectUri = Program.RedirectUri;

        public override async TT.Task<SC.SqlAuthenticationToken>
            AcquireTokenAsync(SC.SqlAuthenticationParameters parameters)
        {
            AD.AuthenticationContext authContext =
                new AD.AuthenticationContext(parameters.Authority);
            authContext.CorrelationId = parameters.ConnectionId;
            AD.AuthenticationResult result;

            switch (parameters.AuthenticationMethod)
            {
                case SC.SqlAuthenticationMethod.ActiveDirectoryInteractive:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,  // "https://database.windows.net/"
                        _clientId,
                        _redirectUri,
                        new AD.PlatformParameters(AD.PromptBehavior.Auto),
                        new AD.UserIdentifier(
                            parameters.UserId,
                            AD.UserIdentifierType.RequiredDisplayableId));
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_1 == '.ActiveDirectoryIntegrated'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserCredential());
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryPassword:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_2 == '.ActiveDirectoryPassword'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserPasswordCredential(
                            parameters.UserId,
                            parameters.Password));
                    break;

                default: throw new InvalidOperationException();
            }
            return new SC.SqlAuthenticationToken(result.AccessToken, result.ExpiresOn);
        }

        public override bool IsSupported(SC.SqlAuthenticationMethod authenticationMethod)
        {
            return authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryInteractive
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryPassword;
        }
    } // EOClass ActiveDirectoryAuthProvider .
} // EONamespace.  End of entire program source code.

```

&nbsp;

#### Actual test output from C#

```
[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>> ADInteractive5.exe
In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.
******** MY QUERY RAN SUCCESSFULLY!! ********

:Success

[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>>
```

## Next steps

- [Get information about an Azure AD administrator for SQL Server](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqlserveractivedirectoryadministrator)

