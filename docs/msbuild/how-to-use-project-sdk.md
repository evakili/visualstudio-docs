---
title: "How to: Reference an MSBuild Project SDK | Microsoft Docs"
ms.custom: ""
ms.date: "01/25/2018"
ms.reviewer: ""
ms.suite: ""
ms.technology: msbuild
ms.tgt_pltfrm: ""
ms.topic: "article"
helpviewer_keywords: 
  - "MSBuild, SDKs, SDK"
author: "jeffkl"
ms.author: jeffkl
manager: angerlic
ms.workload: 
  - "multiple"
---
# How to: Use MSBuild Project SDKs
[!INCLUDE[vstecmsbuild](../extensibility/internals/includes/vstecmsbuild_md.md)] 15.0 introduced the concept of the "project SDK", which simplifies using software development kits that require properties and targets to be imported.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>net46</TargetFramework>
    </PropertyGroup>
</Project>
```  
  
During evaluation of the project, [!INCLUDE[vstecmsbuild](../extensibility/internals/includes/vstecmsbuild_md.md)] adds implicit imports at the top and bottom of your project:

```xml
<Project>
    <!-- Implicit top import -->
    <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" />

    <PropertyGroup>
        <TargetFramework>net46</TargetFramework>
    </PropertyGroup>

    <!-- Implicit bottom import -->
    <Import Project="Sdk.targets" Sdk="Microsoft.NET.Sdk" />
</Project>  
```  

## Referencing a Project SDK
 There are three ways to reference a project SDK

1. Use the `Sdk` attribute on the `<Project/>` element:
    ```xml
    <Project Sdk="My.Custom.Sdk">
        ...
    </Project>
    ```
    An implicit import is added to the top and bottom of the project as discussed above.  The format of the `Sdk` attribute is `Name[/Version]` where Version is optional.  For example, you can specify `My.Custom.Sdk/1.2.3`.

2. Use the top-level `<Sdk/>` element:
    ```xml
    <Project>
        <Sdk Name="My.Custom.Sdk" Version="1.2.3" />
        ...
    </Project>
   ```
   An implicit import is added to the top and bottom of the project as discussed above.  The `Version` attribute is not required.

3. Use the `<Import/>` element anywhere in your project:
    ```xml
    <Project>
        <PropertyGroup>
            <MyProperty>Value</MyProperty>
        </PropertyGroup>
        <Import Project="Sdk.props" Sdk="My.Custom.Sdk" />
        ...
        <Import Project="Sdk.targets" Sdk="My.Custom.Sdk" />
    </Project>
   ```
   Explicitly including the imports in your project allows you full control over the order.

   When using the `<Import/>` element, you can specify an optional `Version` attribute as well.  For example, you can specify `<Import Project="Sdk.props" Sdk="My.Custom.Sdk" Version="1.2.3" />`.

## How Project SDKs are Resolved
When evaluating the import, [!INCLUDE[vstecmsbuild](../extensibility/internals/includes/vstecmsbuild_md.md)] dynamically resolves the path to the project SDK based on the name and version you specified.  [!INCLUDE[vstecmsbuild](../extensibility/internals/includes/vstecmsbuild_md.md)] also has a list of registered SDK resolvers which are plug-ins that locate project SDKs on your machine.  These plug-ins include:

1. A NuGet-based resolver that queries your configured package feeds for NuGet packages that match the ID and version of the SDK you specified.<br/>
   This resolver is only active if you specified an optional version and it can be used for any custom project SDK.  
2. A .NET CLI resolver that resolves SDKs that are installed with .NET CLI.<br/>
   This resolver locates project SDKs such as `Microsoft.NET.Sdk` and `Microsoft.NET.Sdk.Web` which are part of the product.
3. A default resolver that resolves SDKs that were installed with MSBuild.

The NuGet-based SDK resolver supports specifying a version in your [global.json](https://docs.microsoft.com/en-us/dotnet/core/tools/global-json) that allows you to control the project SDK version in one place rather than in each individual project:

```json
{
    "msbuild-sdks": {
        "My.Custom.Sdk": "5.0.0",
        "My.Other.Sdk": "1.0.0-beta"
    }
}
```
Only one version of each project SDK can be used during a build.  If you are referencing two different versions of the same project SDK, MSBuild will emit a warning.  It is recommended to **not** specify a version in your projects if a version is specified in your `global.json`.  

## See Also  
 [MSBuild Concepts](../msbuild/msbuild-concepts.md)   
 [Customize Your Build](../msbuild/customize-your-build.md)   
