# TIA Add-In Build Package

The TIA Add-In Build Package creates and manages all necessary files to develop and debug a TIA Add-In. After installing the build package, the project is ready for development and no further setup is required.

> This project serves as documentation and [issue tracker](CONTRIBUTING.md) only.

## Features

- Automatic referencing of the `Siemens.Engineering.AddIn` assembly from the TIA Portal installation
- Template files
  - view provider and shortcut menu template files are created
  - improved support for unhandled exceptions
- Creating and dynamically updating of the Add-In publisher configuration
  - configuration is fully preconfigured
  - automatic detection of dependencies (AdditionalAssemblies)
  - automatic detection of certificates in the project structure
- Post build event for converting (\*.dll -> \*.addin) of the TIA Add-In
  - *.addin is automatically deployed into the TIA Portal installation (requires admin privileges)
- Improved error handling for unhandled exceptions
  - Errors are displayed as a message box before the add-in is terminated
- Extension methods for
  - recursive finding of all devices in the project
  - get all PLCs, Drives, ...
  - iterate through all parents
  - and more ...
- Deployment of [TIA Add-In Tester](https://support.industry.siemens.com/cs/ww/en/view/109783096)
  - Debug configuration is preset
  - Add-In is immediately ready for testing/debugging without copying and installing it in the TIA Portal

## Installation

Create a new class library project or install the package into an existing project.

1. right click the add-in project and select 'Manage NuGet Packages...'
2. install package `Siemens.Collaboration.Net.TiaPortal.AddIn.Build` and select a matching version (16.\* = TIA Portal V16, 17.\* = TIA Portal V17)
3. build your project

## References

A reference to the assembly `Siemens.Engineering.AddIn.dll` must be added to the project. This establishes the connection to the Openness API. The TIA Add-In Build Package automatically detects the TIA Portal installation and references the required references into the project.

> In case the TIA Portal location can not be detected, define the property `TiaPortalLocation` the add-in project file (*.csproj). See [Example](#example) for reference.

## Build Tasks

The TIA Add-In Build Package executes multiple build tasks before and after building the add-in project.

### Ensure Environment

The `EnsureEnvironment` task verifies that the correct target framework is configured for the Add-In project. TIA Add-Ins build for TIA Portal V16 should be built for .Net Framework 4.6.2, TIA Portal V17 for .Net Framework 4.8. If the wrong target framework configuration is detected the correct version will be set automatically.
> This task can be disabled by setting `<EnsureEnvironmentTaskEnabled>false</EnsureEnvironmentTaskEnabled>` in the project file (*.csproj).

### Template Task

A simple add-in consists of at least the following two classes:

- View Provider: In this class, you define the areas in which the shortcut menu is to be displayed. For each area you have to program a separate provider, i.e. a separate class:
  - `ProjectTreeAddInProvider`: Project tree
  - `GlobalLibraryTreeAddInProvider`: Global libraries
  - `ProjectLibraryTreeAddInProvider`: Project library
  - `DevicesAndNetworksAddInProvider`: Hardware and network editor
  - ~~`VciEditorAddInProvider`: Version Control Interface workspace~~ (not supported in this template)
- Shortcut menu: This class contains the program code and defines the shortcut menu with its entries.

One or more view providers need to be inherited in order to be able to load the Add-In. Each view provider then provides one or more shortcut menu instances. Shortcut menu items are implemented in classes by inheriting the class `ContextMenuAddIn`.

The TIA Add-In Build Package will deploy template implementations of the required classes if no implementation is detected.
> This task can be disabled by setting `<TemplateTaskEnabled>false</TemplateTaskEnabled>` in the project file (*.csproj).

### Publisher Task

For the add-in to be recognized by the TIA Portal and displayed, the compiled assembly file (*.dll) must be converted to a file with the file extension ".addin". To do this the command-line program `Siemens.Engineering.AddIn.Publisher.exe` that is available in the installation directory of the TIA Portal in the "PublicAPI" folder must be used.  
To convert an assembly a configuration file is needed which provides all required information. This file is either created and preconfigured by the TIA Add-In Build Package (`AddInPublisherConfiguration.xml`) or automatically detected in the project directory. The following settings of the configuration file will be maintained by the TIA Add-In Build Package:

- path of the assembly to be converted (FeatureAssembly)
- automatic detection of dependencies (AdditionalAssemblies)
- automatic detection of certificates in the project structure
- file version update

A post build event is automatically created to convert the assembly to an add-in. The converted add-in will be deployed alongside the assembly.
If the build process is executed with elevated privileges the add-in is deployed in the TIA Portal Add-Ins directory as well.

> Open the IDE (e.g. Microsoft Visual Studio) with admin privileges for automatic add-in deployment.

> This task can be disabled by setting `<PublisherTaskEnabled>false</PublisherTaskEnabled>` in the project file (*.csproj).

### Deploy Launch Settings Task

TIA Add-Ins can easily be tested and debugged using the [TIA Add-In Tester](https://support.industry.siemens.com/cs/ww/en/view/109783096). The [add-in templates](#template-task) already implement the required `GetSelection` method to create a `MenuSelectionProvider` selection. The deployment and debug configuration of the TIA Add-In Tester will be setup by this task. See [TIA Add-In Tester](https://support.industry.siemens.com/cs/ww/en/view/109783096) for further information on how to debug and test a TIA Add-In.

> This task can be disabled by setting `<DeployLaunchSettingsTaskEnabled>false</DeployLaunchSettingsTaskEnabled>` in the project file (*.csproj).

### Write Changes Task

After all task have been completed the necessary changes are written to the files.
> This task can be disabled by setting `<WriteChangesTaskEnabled>false</WriteChangesTaskEnabled>` in the project file (*.csproj).

### Disabling a task

All tasks can be disabled. Open the add-in project file (*.csproj) and set one of the following properties in a property group to disable it:

- `<EnsureEnvironmentTaskEnabled>false</EnsureEnvironmentTaskEnabled>`
- `<TemplateTaskEnabled>false</TemplateTaskEnabled>`
- `<PublisherTaskEnabled>false</PublisherTaskEnabled>`
- `<DeployLaunchSettingsTaskEnabled>false</DeployLaunchSettingsTaskEnabled>`
- `<WriteChangesTaskEnabled>false</WriteChangesTaskEnabled>`

#### Example

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <EnsureEnvironmentTaskEnabled>false</EnsureEnvironmentTaskEnabled>
    <TiaPortalLocation>F:\Software\TIA Portal V17</TiaPortalLocation>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Siemens.Collaboration.Net.TiaPortal.AddIn.Build" Version="17.*" />
  </ItemGroup>
</Project>
```

## Extensions

The TIA Add-In Build Package adds several extension methods.

### Hardware

Get all devices in a project (iterates through devices, device groups and ungrouped devices recursively)

```csharp
IEnumerable<Device> devices = tia.Projects.First().AllDevices()
```

Recursively get all device items of a device

```csharp
IEnumerable<DeviceItem> deviceItems = myDevice.AllDeviceItems()
```

Get all device items with classification CPU in this project

```csharp
IEnumerable<DeviceItem> cpus = project.AllCpus();
```

Get all PlcSoftware in the project

```csharp
IEnumerable<PlcSoftware> plcSoftwares = project.AllPlcSoftware();
```

Get all SINAMICS Startdrive devices in the project

```csharp
IEnumerable<Device> drives = project.AllStartdriveDevices();
```

### Parents

Get all parents of an IEngineeringObject

```csharp
IEnumerable<IEngineeringObject> parents = engineeringObject.Parents();
```

Get first parent of type `Device` of an IEngineeringObject

```csharp
Device device = engineeringObject.Parent<Device>();
```

### Messages and Dialogs

Open a message box in foreground

```csharp
var result = AddInMessageBox.Show("Successfully imported all files", "Import done", MessageBoxButton.OK, MessageBoxImage.Information);
```

Open a dialog in the foreground of the TIA Portal

```csharp
var result = openFileDialog.ShowDialogInForeground();
```

## License

Code and documentation copyright 2022 Siemens AG.

See [LICENSE.md](LICENSE.md).
