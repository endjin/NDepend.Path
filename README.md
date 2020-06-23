# NDepend.Path

NDepend.Path is framework to handle all sorts of paths operations: File, Directory, Absolute, Drive Letter, UNC, Relative, prefixed with an Environment Variable, that contain a Variable etc... 

This framework is used in the product [NDepend](https://ndepend.com) to handle all paths operations.

## CI / CD

The project is [hosted on Azure DevOps](https://dev.azure.com/endjin-labs/NDepend.Path) under the `endjin-labs` org.

## Packages

The NuGet packages for the project, hosted on NuGet.org are:

- [NDepend.Path](https://www.nuget.org/packages/NDepend.Path)

## History

- Jan 30, 2014 - Initial Release from [Patrick Smacchia](https://github.com/psmacchia/NDepend.Path), author of [NDepend](https://ndepend.com)
- April 14 2015 - Werner Putschögl creates & publishes a nuget package 
- June 23 2020 - [endjin](https://endjin.com) updates project to .NET Standard 2.1 and publishes new nuget package.

## Licenses

[![GitHub license](https://img.shields.io/badge/MIT-blue.svg)](https://github.com/endjin/NDepend.Path/blob/master/LICENSE)

NDepend.Path is available under the [MIT open source license](https://github.com/endjin/NDepend.Path/blob/master/LICENSE).

## Code of conduct

This project has adopted a code of conduct adapted from the [Contributor Covenant](http://contributor-covenant.org/) to clarify expected behavior in our community. This code of conduct has been [adopted by many other projects](http://contributor-covenant.org/adopters/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [&#104;&#101;&#108;&#108;&#111;&#064;&#101;&#110;&#100;&#106;&#105;&#110;&#046;&#099;&#111;&#109;](&#109;&#097;&#105;&#108;&#116;&#111;:&#104;&#101;&#108;&#108;&#111;&#064;&#101;&#110;&#100;&#106;&#105;&#110;&#046;&#099;&#111;&#109;) with any additional questions or comments.

## Examples

### Support for absolute file and directory path, drive letter and absolute

```csharp
var absoluteFilePath = @"C:\Dir\File.txt".ToAbsoluteFilePath();

Assert.IsTrue(absoluteFilePath.FileName == "File.txt");
Assert.IsTrue(absoluteFilePath.Kind == AbsolutePathKind.DriveLetter);
Assert.IsTrue(absoluteFilePath.DriveLetter.Letter == 'C');

var absoluteUNCDirectoryPath = @"\\Server\Share\Dir".ToAbsoluteDirectoryPath();

Assert.IsTrue(absoluteUNCDirectoryPath.DirectoryName == "Dir");
Assert.IsTrue(absoluteUNCDirectoryPath.Kind == AbsolutePathKind.UNC);
Assert.IsTrue(absoluteUNCDirectoryPath.UNCServer == "Server");
Assert.IsTrue(absoluteUNCDirectoryPath.UNCShare == "Share");
```

### Support for path browsing

```csharp
Assert.IsTrue(absoluteFilePath.ParentDirectoryPath.ToString() == @"C:\Dir");
Assert.IsTrue(absoluteUNCDirectoryPath.GetChildFileWithName("File.txt").ToString() == @"\\Server\Share\Dir\File.txt");
Assert.IsTrue(absoluteUNCDirectoryPath.GetBrotherDirectoryWithName("Dir2").ToString() == @"\\Server\Share\Dir2");

if (absoluteUNCDirectoryPath.Exists) {
   var filePaths = absoluteUNCDirectoryPath.ChildrenFilesPath;
   var dirPaths = absoluteUNCDirectoryPath.ChildrenDirectoriesPath;
}
```

### Support for relative path

```csharp
var absoluteDirectoryPath = @"C:\DirA\DirB".ToAbsoluteDirectoryPath();
var relativeFilePath = @"..\DirC\File.txt".ToRelativeFilePath();
var absoluteFilePath2 = relativeFilePath.GetAbsolutePathFrom(absoluteDirectoryPath);

Assert.IsTrue(absoluteFilePath2.ToString() == @"C:\DirA\DirC\File.txt");

var relativeFilePath2 = absoluteFilePath2.GetRelativePathFrom(absoluteDirectoryPath);

Assert.IsTrue(relativeFilePath2.Equals(relativeFilePath)); // Use Equals() for path comparison, dont use ==
```

### Support for path prefixed with an environment variable

```csharp
var envVarFilePath = @"%ENVVAR%\DirB\File.txt".ToEnvVarFilePath();
Assert.IsTrue(envVarFilePath.EnvVar == "%ENVVAR%");

IAbsoluteFilePath absoluteFilePath3;
Assert.IsTrue(envVarFilePath.TryResolve(out absoluteFilePath3) == EnvVarPathResolvingStatus.ErrorUnresolvedEnvVar);

Environment.SetEnvironmentVariable("ENVVAR", @"NotAValidPath");
Assert.IsTrue(envVarFilePath.TryResolve(out absoluteFilePath3) == EnvVarPathResolvingStatus.ErrorEnvVarResolvedButCannotConvertToAbsolutePath);

Environment.SetEnvironmentVariable("ENVVAR", @"C:\DirA");

Assert.IsTrue(envVarFilePath.TryResolve(out absoluteFilePath3) == EnvVarPathResolvingStatus.Success);
Assert.IsTrue(absoluteFilePath3.ToString() == @"C:\DirA\DirB\File.txt");

Environment.SetEnvironmentVariable("ENVVAR", "");
```

### Support for path that contain variable(s)

```csharp
var variableFilePath = @"$(VarA)\DirB\$(VarC)\File.txt".ToVariableFilePath();

Assert.IsTrue(variableFilePath.PrefixVariable == "VarA");
Assert.IsTrue(variableFilePath.AllVariables.First() == "VarA");
Assert.IsTrue(variableFilePath.AllVariables.ElementAt(1) == "VarC");

IAbsoluteFilePath absoluteFilePath4;

Assert.IsTrue(variableFilePath.TryResolve(
   new KeyValuePair<string, string>[0],
   out absoluteFilePath4) == VariablePathResolvingStatus.ErrorUnresolvedVariable);

Assert.IsTrue(variableFilePath.TryResolve(
   new[] {new KeyValuePair<string, string>("VarA", "NotAValidPath"),
            new KeyValuePair<string, string>("VarC", "DirC")},
   out absoluteFilePath4) == VariablePathResolvingStatus.ErrorVariableResolvedButCannotConvertToAbsolutePath);

Assert.IsTrue(variableFilePath.TryResolve(
   new[] {new KeyValuePair<string, string>("VarA", @"C:\DirA"),
            new KeyValuePair<string, string>("VarC", "DirC")},
   out absoluteFilePath4) == VariablePathResolvingStatus.Success);

Assert.IsTrue(absoluteFilePath4.ToString() == @"C:\DirA\DirB\DirC\File.txt");
```

### Support for path normalisation

```csharp
Assert.IsTrue(@"C://DirA/\DirB//".ToDirectoryPath().ToString() == @"C:\DirA\DirB");
Assert.IsTrue(@"%ENVVAR%\DirA\..\DirB".ToDirectoryPath().ToString() == @"%ENVVAR%\DirB");
Assert.IsTrue(@".\..".ToDirectoryPath().ToString() == "..");
Assert.IsTrue(@".\..\.\Dir".ToDirectoryPath().ToString() == @"..\Dir");
```

### Support for paths collection

```csharp
IAbsoluteDirectoryPath commonRootDir;

Assert.IsTrue(new[] {
   @"C:\Dir".ToAbsoluteDirectoryPath(),
   @"C:\Dir\Dir1".ToAbsoluteDirectoryPath(),
   @"C:\Dir\Dir2\Dir3".ToAbsoluteDirectoryPath(),
}.TryGetCommonRootDirectory(out commonRootDir));

Assert.IsTrue(commonRootDir.ToString() == @"C:\Dir");
```

### Possibility to work with IFilePath IDirectoryPath and be abstracted from the underlying kind (Absolute/Relative/EnvVar/Variable)

```csharp
foreach (var s in new[] {
   @"C:\Dir\File.txt",
   @"\\Server\Share\Dir\File.txt",
   @"..\..\Dir\File.txt",
   @"%ENVVAR%\Dir\File.txt",
   @"$(Var)\Dir\File.txt",
}) {
   var filePath = s.ToFilePath();

   Assert.IsTrue(filePath.FileName == @"File.txt");
   Assert.IsTrue(filePath.HasParentDirectory);
   Assert.IsTrue(filePath.ParentDirectoryPath.DirectoryName == @"Dir");
}
```

### Support for Try... syntax to avoid exception

```csharp
IDirectoryPath directoryPath;

Assert.IsFalse(@"NotAValidPath".TryGetDirectoryPath(out directoryPath));

string failureReason;

Assert.IsFalse(@"NotAValidPath".IsValidDirectoryPath(out failureReason));
Assert.IsTrue(failureReason == @"The string ""NotAValidPath"" is not a valid directory path.");
```

### Smart enough to distinguish between File and Directory path when possible

```csharp
Assert.IsFalse(@"C:".IsValidFilePath());
Assert.IsFalse(@"\\Server\Share".IsValidFilePath());
Assert.IsTrue(@"C:".IsValidDirectoryPath());
Assert.IsTrue(@"\\Server\Share".IsValidDirectoryPath());

Assert.IsTrue(@"C:\Dir".IsValidFilePath());
Assert.IsTrue(@"\\Server\Share\Dir".IsValidFilePath());
Assert.IsTrue(@"C:\Dir".IsValidDirectoryPath());
Assert.IsTrue(@"\\Server\Share\Dir".IsValidDirectoryPath());
```