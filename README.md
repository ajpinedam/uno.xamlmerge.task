# Uno.XamlMerge.Task
An msbuild task which enables the merging of WinUI XAML resource dictionaries into a single flat ResourceDictionary.

This code is derived from WinUI's [`BatchMergeXaml`](https://github.com/microsoft/microsoft-ui-xaml/blob/a1ace7957abc19bce86141203560e15a737ccac7/tools/CustomTasks/BatchMergeXaml.cs) class, to be reused in any WinUI and Uno Platform Controls library.

## Why merge ResourceDictionary instances ?
This task is generally used to allow for separate resource dictionary authoring (which makes them easier to load and read) to avoid impacting resources lookup runtime performance. In WinUI, ResourceDictionary resolution is performed through a graph traversal of MergedDictionaries, which generally implies that a worse case resolution can require as many lookups as there are dictionaries.

To limit the impact of the traversal, this task task takes all resource dictionaries found in a specific MSBuild item group, and merges them into a single file. This file is then generally either named `Themes\Generic.xaml` or referenced as a merged dictionary from another `Themes\Generic.xaml` file.

## Using the task

Include the following block:

```xml
<PropertyGroup>
    <_Uno_XamlMerge_Task_Version>1.0.0</_Uno_XamlMerge_Task_Version>
</PropertyGroup>
<ItemGroup>
    <PackageReference Include="Uno.XamlMerge.Task" Version="$(_Uno_XamlMerge_Task_Version)" />
</ItemGroup>
```

Then select the resource dictionaries to be merged:
```xml
<ItemGroup>
    <XamlMergeInput Include="Styles\**\*.xaml" Exclude="Styles\Generic.xaml" />
</ItemGroup>
```

> Note: The `Exclude` portion is only needed if some of your dictionaries still need to be referenced explicitly as some libraries require valid full path, such as [Uno.Themes](https://github.com/unoplatform/uno.themes).

Then add the following block at the end your project library or app:

```xml
<Import Project="$(NuGetPackageRoot)uno.xamlmerge.task\$(_Uno_XamlMerge_Task_Version)\build\Uno.XamlMerge.Task.targets"
        Condition="'$(TargetFramework)' == '' and '$(TargetFrameworks)'!='' and exists('$(NuGetPackageRoot)\uno.xamlmerge.task\$(_Uno_XamlMerge_Task_Version)')" />
```

The generated file is called `Generated\mergedpages.xaml` by default, but can be overriden as follows:

```xml
<PropertyGroup>
    <XamlMergeOutputFile>Themes\Generic.xaml</XamlMergeOutputFile>
</PropertyGroup>
```
Otherwise, the generated file can be referenced as follows:
```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:TestLibrary">

	<ResourceDictionary.MergedDictionaries>
		<ResourceDictionary Source="ms-appx:///REPLACE_ME/Generated/mergedpages.xaml" />
	</ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```

## Troubleshooting
- Make sure that all namespaces definitions across files are of the same values. (e.g. `xlmns:ns1="http://site1"` and `xlmns:ns1="http://site2"` in two different files will fail)
- If you include the same resource dictionary (e.g. Colors.xaml) file in multiple merged files (generally used to have a file for custom brushes and colors), you can remove those multiple inclusions.
