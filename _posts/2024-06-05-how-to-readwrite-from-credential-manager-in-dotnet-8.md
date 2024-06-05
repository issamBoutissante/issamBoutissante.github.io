---
layout: post
title: "How to Read/Write from Credential Manager in .NET 8"
date: 2024-06-05 11:20:30 +0100
categories: [.NET, Credential Manager, Secure Key Storage]
comments: true
---

## How to Read/Write from Credential Manager in .NET 8

Credential Manager is a secure storage solution for sensitive information, such as usernames and passwords. It provides a way to manage credentials for various applications securely. In this article, we will walk through the steps to read, write, update, and delete credentials in .NET 8 using the Meziantou.Framework.Win32 library, which also works in .NET 6 and 7.

### Setting Up the Project

Before we dive into the code, ensure you have the following NuGet package installed in your .NET project:

```shell
dotnet add package Meziantou.Framework.Win32
```

### Writing Credentials to Credential Manager

To securely save credentials, we can use the `CredentialManager.WriteCredential` method. Here is a step-by-step guide:

1. **Define the Credential Information:**
   - `applicationName`: A unique identifier for the application.
   - `userName`: The username associated with the credential.
   - `secret`: The password or sensitive information.
   - `comment`: A description for the credential.
   - `persistence`: Determines how the credential is stored (Session, LocalMachine, Enterprise).

```csharp
using Meziantou.Framework.Win32;
using System;

public class CredentialManagerHelper
{
    public static void SaveCredential(string applicationName, string userName, string secret, string comment, CredentialPersistence persistence)
    {
        CredentialManager.WriteCredential(
            applicationName: applicationName,
            userName: userName,
            secret: secret,
            comment: comment,
            persistence: persistence);
    }
}
```

2. **Usage Example:**

```csharp
class Program
{
    static void Main(string[] args)
    {
        string appName = "MyApp";
        string userName = "user123";
        string password = "P@ssw0rd!";
        string comment = "User login information";

        CredentialManagerHelper.SaveCredential(appName, userName, password, comment, CredentialPersistence.LocalMachine);

        Console.WriteLine("Credential saved successfully.");
    }
}
```

### Reading Credentials from Credential Manager

To retrieve the stored credentials, use the `CredentialManager.ReadCredential` method:

```csharp
public class CredentialManagerHelper
{
    public static Credential ReadCredential(string applicationName)
    {
        var credential = CredentialManager.ReadCredential(applicationName);
        if (credential == null)
        {
            Console.WriteLine("No credential found.");
            return null;
        }

        Console.WriteLine($"UserName: {credential.UserName}");
        Console.WriteLine($"Secret: {credential.Password}");
        Console.WriteLine($"Comment: {credential.Comment}");

        return credential;
    }
}
```

### Updating Credentials

Updating a credential is straightforward. Simply call the `SaveCredential` method again with the same `applicationName`:

```csharp
public static void UpdateCredential(string applicationName, string newUserName, string newSecret, string newComment)
{
    SaveCredential(applicationName, newUserName, newSecret, newComment, CredentialPersistence.LocalMachine);
}
```

### Deleting Credentials

To remove a credential, use the `CredentialManager.DeleteCredential` method:

```csharp
public class CredentialManagerHelper
{
    public static void DeleteCredential(string applicationName)
    {
        try
        {
            CredentialManager.DeleteCredential(applicationName);
            Console.WriteLine("Credential deleted successfully.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error deleting credential: {ex.Message}");
        }
    }
}
```

### Understanding Credential Persistence

The `CredentialPersistence` enumeration defines how credentials are stored:

- **Session**: Credentials are only available for the current session.
- **LocalMachine**: Credentials are saved for the current user on the local machine and are not accessible by other users.
- **Enterprise**: Credentials are available to all authenticated users on the domain.

```csharp
public enum CredentialPersistence : uint
{
    Session = 1,
    LocalMachine,
    Enterprise,
}
```

### Full Example

Here is a complete example that demonstrates creating, reading, updating, and deleting credentials:

```csharp
using Meziantou.Framework.Win32;
using System;

public class CredentialManagerHelper
{
    public static void SaveCredential(string applicationName, string userName, string secret, string comment, CredentialPersistence persistence)
    {
        CredentialManager.WriteCredential(
            applicationName: applicationName,
            userName: userName,
            secret: secret,
            comment: comment,
            persistence: persistence);
    }

    public static Credential ReadCredential(string applicationName)
    {
        var credential = CredentialManager.ReadCredential(applicationName);
        if (credential == null)
        {
            Console.WriteLine("No credential found.");
            return null;
        }

        Console.WriteLine($"UserName: {credential.UserName}");
        Console.WriteLine($"Secret: {credential.Password}");
        Console.WriteLine($"Comment: {credential.Comment}");

        return credential;
    }

    public static void UpdateCredential(string applicationName, string newUserName, string newSecret, string newComment)
    {
        SaveCredential(applicationName, newUserName, newSecret, newComment, CredentialPersistence.LocalMachine);
    }

    public static void DeleteCredential(string applicationName)
    {
        try
        {
            CredentialManager.DeleteCredential(applicationName);
            Console.WriteLine("Credential deleted successfully.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error deleting credential: {ex.Message}");
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        string appName = "MyApp";
        string userName = "user123";
        string password = "P@ssw0rd!";
        string comment = "User login information";

        // Save credential
        CredentialManagerHelper.SaveCredential(appName, userName, password, comment, CredentialPersistence.LocalMachine);
        Console.WriteLine("Credential saved successfully.");

        // Read credential
        CredentialManagerHelper.ReadCredential(appName);

        // Update credential
        CredentialManagerHelper.UpdateCredential(appName, "newUser", "NewP@ssw0rd!", "Updated user login information");

        // Read updated credential
        CredentialManagerHelper.ReadCredential(appName);

        // Delete credential
        CredentialManagerHelper.DeleteCredential(appName);
    }
}
```

### Conclusion

In this article, we've covered how to read, write, update, and delete credentials using the Credential Manager in .NET 8. This approach ensures that sensitive information is stored securely, leveraging the built-in capabilities of the Windows operating system. By following these steps, you can manage your application's credentials securely and efficiently.

Feel free to reach out with any questions or feedback in the comments below!
