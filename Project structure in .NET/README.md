## Project structure

Порядок действий для создания решения и проектов.

```sh
$CA_SlnPath = "super-solution"
$CA_SlnName = "SuperSolution"
$CA_Framework = "net9.0"
$CA_SDKVersion = "9.0.0"

mkdir $CA_SlnPath
dotnet new editorconfig -o $CA_SlnPath
```

Содержимое .editorconfig между `# All files` и `#### .NET Coding Conventions ####`

```Properties
# All files
[*]
charset = utf-8
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

# Visual Studio Solution Files
[*.sln]
indent_style = tab

# Visual Studio XML Project Files
[*.{csproj,vbproj,vcxproj.filters,proj,projitems,shproj}]
indent_size = 2

# XML Configuration Files
[*.{xml,config,props,targets,nuspec,resx,ruleset,vsixmanifest,vsct}]
indent_size = 2

# JSON Files
[*.{json,json5,webmanifest}]
indent_size = 2

# YAML Files
[*.{yml,yaml}]
indent_size = 2

# Markdown Files
[*.{md,mdx}]
trim_trailing_whitespace = false

# Web Files
[*.{htm,html,js,jsm,ts,tsx,cjs,cts,ctsx,mjs,mts,mtsx,css,sass,scss,less,pcss,svg,vue}]
indent_size = 2

# Batch Files
[*.{cmd,bat}]
end_of_line = crlf

# Bash Files
[*.sh]
end_of_line = lf

# Makefiles
[Makefile]
indent_style = tab

# C# files
[*.cs]

#### Core EditorConfig Options ####

# Indentation and spacing
indent_size = 4

# New line preferences
insert_final_newline = true
trim_trailing_whitespace = true

#### .NET Coding Conventions ####
```

```sh
dotnet new globaljson --sdk-version $CA_SDKVersion --roll-forward latestFeature -o $CA_SlnPath
dotnet new gitignore -o $CA_SlnPath
dotnet new buildprops -o $CA_SlnPath
dotnet new packagesprops -o $CA_SlnPath
dotnet new buildtargets -o $CA_SlnPath
dotnet new sln -o $CA_SlnPath -n $CA_SlnName
```

Создание NuGet.config

```XML
<?xml version="1.0" encoding="utf-8"?>
<configuration>

  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  </packageSources>

  <packageSourceMapping>
    <packageSource key="nuget.org">
      <package pattern="*" />
    </packageSource>
  </packageSourceMapping>

</configuration>
```

Создание .gitattributes

```Properties
# Auto-detect text files, ensure they use LF.
* text=auto eol=lf working-tree-encoding=UTF-8

# Solution files
*.sln text=auto eol=crlf working-tree-encoding=UTF-8

# Bash scripts
*.sh text eol=lf
```

Содержимое для Directory.Build.props

```XML
<Project>
  <!-- See https://aka.ms/dotnet/msbuild/customize for more details on customizing your build -->
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
    <!-- <ArtifactsPath>$(MSBuildThisFileDirectory)artifacts</ArtifactsPath> -->
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

Содержимое для Directory.Packages.props

```XML
<!-- For more info on central package management go to https://devblogs.microsoft.com/nuget/introducing-central-package-management/ -->
<Project>
  <PropertyGroup>
    <!-- Enable central package management, https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management -->
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>

  <ItemGroup Label="Package versions for .NET 9.0"
    Condition=" '$(TargetFrameworkIdentifier)' == '.NETCoreApp' And $([MSBuild]::VersionEquals($(TargetFrameworkVersion), '9.0')) ">
  </ItemGroup>

</Project>
```

Создание проектов решения

```sh
dotnet new web -o $CA_SlnPath/src/Web --framework $CA_Framework
dotnet new classlib -o $CA_SlnPath/src/Application --framework $CA_Framework
dotnet new classlib -o $CA_SlnPath/src/Infrastructure --framework $CA_Framework
dotnet new classlib -o $CA_SlnPath/src/Domain --framework $CA_Framework
```

P.S. В .csproj файлах описаны свойства для текущего проекта, они перезаписывают свойства из Directory.Build.props \
Пример:

```XML
<PropertyGroup>
  <TargetFramework>net9.0</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

Настройки `RootNamespace` и `AssemblyName`

```XML
<PropertyGroup>
  <RootNamespace>SuperSolution.Web</RootNamespace>
  <AssemblyName>SuperSolution.Web</AssemblyName>
</PropertyGroup>
```

Включение проектов в решение

```sh
dotnet sln $CA_SlnPath/$CA_SlnName.sln add (ls -r $CA_SlnPath/src/*.csproj)
```

Связывание проектов ссылками

```sh
dotnet add $CA_SlnPath/src/Web reference $CA_SlnPath/src/Application
dotnet add $CA_SlnPath/src/Web reference $CA_SlnPath/src/Infrastructure
dotnet add $CA_SlnPath/src/Infrastructure reference $CA_SlnPath/src/Application
dotnet add $CA_SlnPath/src/Application reference $CA_SlnPath/src/Domain
```

## TODO

Добавить проекты для тестов (см. clean-architecture):

* Application.FunctionTests
* Application.UnitTests
* Infrastructure.IntegrationTests
* Domain.UnitTests

Добавить референсы для тестов: ?

#### Domain
Не использовать Data Annotations в domain сущностях, использовать Fluent API\
Использовать Value Objects там где уместно\
Использовать пользовательские исключения\
Инициализировать все коллекции

#### Application
CQRS + MediatR\
Fluent Validation\
AutoMapper

#### Infrastructure
Независимость от базы данных\
Fluent API (data annotations)\
Там где возможно использовать conventions вместо configuration (меньше кода)\
Никакой из слоев не зависит от Infrastructure

#### Api (Web, UI и т.д.)
В контроллерах не должно быть никакой логики\
ViewModels\
Open API (swagger)
