````markdown
---
layout: post
title: "Creating Shell Extensions in .NET 8 with SharpShell"
date: 2024-06-03 11:36:34 +0100
categories: [.NET, SharpShell, Shell Extensions]
---

In this blog, I'll walk you through creating a Windows shell extension using the SharpShell library in .NET 8. SharpShell is a popular library for creating shell extensions, but it was traditionally used with the .NET Framework. Thanks to the Microsoft Windows Compatibility Pack, it's now possible to create shell extensions in .NET 8.

## Prerequisites

- Visual Studio 2022 or later
- .NET 8 SDK
- Basic knowledge of C# and Windows shell extensions

## Step 1: Create a .NET 8 Class Library

First, we need to create a .NET 8 class library. Open Visual Studio and create a new project:

1. Select **Class Library**.
2. Name the project `SharpShellNet8Demo`.
3. Choose **.NET 8.0** as the target framework.

## Step 2: Modify the .csproj File

Modify the project file (`.csproj`) to target only Windows and include the necessary packages:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UseWindowsForms>true</UseWindowsForms>
    <EnableComHosting>true</EnableComHosting>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.Compatibility" Version="8.0.4" />
    <PackageReference Include="SharpShell" Version="2.7.2" />
  </ItemGroup>

</Project>
```
````

## Step 3: Create a Context Menu Extension

Next, we'll create a simple context menu extension. Add a new class to your project named `SimpleContextMenu.cs` and implement the following code:

```csharp
using SharpShell.Attributes;
using SharpShell.SharpContextMenu;
using System;
using System.Linq;
using System.Runtime.InteropServices;
using System.Windows.Forms;

namespace SharpShellNet8Demo
{
    [ComVisible(true)]
    [ClassInterface(ClassInterfaceType.None)]
    [COMServerAssociation(AssociationType.AllFilesAndFolders)]
    [COMServerAssociation(AssociationType.Directory)]
    [COMServerAssociation(AssociationType.DirectoryBackground)]
    [Guid("B3B9C5B6-5C92-4D1B-85E6-2F80B72F6E28")]
    public class SimpleContextMenu : SharpContextMenu
    {
        protected override bool CanShowMenu() => true;

        protected override ContextMenuStrip CreateMenu()
        {
            var menu = new ContextMenuStrip();

            var mainItem = new ToolStripMenuItem
            {
                Text = "Simple Extension Action"
            };

            var actionItem = new ToolStripMenuItem
            {
                Text = "Perform Action"
            };
            actionItem.Click += (sender, e) => MessageBox.Show("Action performed!");

            mainItem.DropDownItems.Add(actionItem);
            menu.Items.Add(mainItem);

            return menu;
        }

        [ComRegisterFunction]
        public static void Register(Type t)
        {
            RegisterContextMenu(t, @"*\shellex\ContextMenuHandlers\");
            RegisterContextMenu(t, @"Directory\shellex\ContextMenuHandlers\");
            RegisterContextMenu(t, @"Directory\Background\shellex\ContextMenuHandlers\");
        }

        private static void RegisterContextMenu(Type t, string basePath)
        {
            string keyPath = basePath + "SimpleContextMenu";
            using (var key = Microsoft.Win32.Registry.ClassesRoot.CreateSubKey(keyPath))
            {
                if (key == null)
                {
                    Console.WriteLine($"Failed to create registry key: {keyPath}");
                }
                else
                {
                    key.SetValue(null, t.GUID.ToString("B"));
                }
            }
        }

        [ComUnregisterFunction]
        public static void Unregister(Type t)
        {
            UnregisterContextMenu(@"*\shellex\ContextMenuHandlers\");
            UnregisterContextMenu(@"Directory\shellex\ContextMenuHandlers\");
            UnregisterContextMenu(@"Directory\Background\shellex\ContextMenuHandlers\");
        }

        private static void UnregisterContextMenu(string basePath)
        {
            string keyPath = basePath + "SimpleContextMenu";
            try
            {
                Microsoft.Win32.Registry.ClassesRoot.DeleteSubKeyTree(keyPath, false);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error unregistering COM server from {keyPath}: {ex.Message}");
            }
        }
    }
}
```

### Explanation of Attributes

- **`[ComVisible(true)]`**: This attribute makes the class visible to COM components.
- **`[ClassInterface(ClassInterfaceType.None)]`**: This attribute specifies that no class interface is generated for the class.
- **`[COMServerAssociation(AssociationType.AllFilesAndFolders)]`**: This attribute registers the shell extension for all files and folders.
- **`[COMServerAssociation(AssociationType.Directory)]`**: This attribute registers the shell extension for directories.
- **`[COMServerAssociation(AssociationType.DirectoryBackground)]`**: This attribute registers the shell extension for the background of directories.
- **`[Guid("B3B9C5B6-5C92-4D1B-85E6-2F80B72F6E28")]`**: This attribute assigns a unique identifier to the class.

## Step 4: Register and Unregister the Extension

To test our shell extension, we need to register and unregister it. Create two batch files, `Register.bat` and `Unregister.bat`, to handle this.

### Register.bat

```batch
@echo off
:: Check for administrative privileges
net session >nul 2>&1
if %errorlevel% == 0 (
    echo Running with administrative privileges
) else (
    echo Requesting administrative privileges...
    goto UACPrompt
)

goto Start

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
    "%temp%\getadmin.vbs"
    exit /B

:Start
cd /d "%~dp0"

:: Find DLL files ending with comhost.dll
for /f "delims=" %%a in ('dir /b *comhost.dll') do set "DLLFile=%%a"
if defined DLLFile goto Found

echo No DLL file ending with comhost.dll found, please enter the DLL name:
SET /P DLLName=Enter the DLL name to register (include .dll extension):
set DLLFile=%DLLName%

:Found
if not exist "%DLLFile%" (
    echo The specified DLL file does not exist.
    pause
    exit /b
)

regsvr32 /s "%DLLFile%"
taskkill /f /im explorer.exe
start explorer.exe
echo %DLLFile% registered and Explorer restarted.
pause
```

### Unregister.bat

```batch
@echo off
:: Check for administrative privileges
net session >nul 2>&1
if %errorlevel% == 0 (
    echo Running with administrative privileges
) else (
    echo Requesting administrative privileges...
    goto UACPrompt
)

goto Start

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
    "%temp%\getadmin.vbs"
    exit /B

:Start
cd /d "%~dp0"

:: Find DLL files ending with comhost.dll
for /f "delims=" %%a in ('dir /b *comhost.dll') do set "DLLFile=%%a"
if defined DLLFile goto Found

echo No DLL file ending with comhost.dll found, please enter the DLL name:
SET /P DLLName=Enter the DLL name to unregister (include .dll extension):
set DLLFile=%DLLName%

:Found
if not exist "%DLLFile%" (
    echo The specified DLL file does not exist.
    pause
    exit /b
)

regsvr32 /u /s "%DLLFile%"
taskkill /f /im explorer.exe
start explorer.exe
echo %DLLFile% unregistered and Explorer restarted.
pause
```

## Step 5: Build and Register the Extension

1. Build your project in Visual Studio.
2. Navigate to the output directory (usually `bin\Debug\net8.0-windows`).
3. Run `Register.bat` as an administrator to register the shell extension.
4. Right-click on any file or folder to see the new context menu option "Simple Extension Action".

## Step 6: Unregister the Extension

When you want to unregister the shell extension, run `Unregister.bat` as an administrator.

## Conclusion

Creating a shell extension in .NET 8 using SharpShell and the Microsoft Windows Compatibility Pack is straightforward. This tutorial covered setting up your project, writing the shell

extension code, and handling registration and unregistration. Experiment with different types of extensions to enhance your Windows Explorer experience!

Feel free to reach out if you have any questions or run into issues. Happy coding!

```

```
