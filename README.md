# Godot Dependency Injection
This package adds simple, Razor-like dependency injection support to Godot C#.

## Why does this package exist?
There is no native DI support in Godot C#, even though it would be very nice to have it for when you need it. I started as a web-dev with ASP.NET web APIs and Blazor projects and DI is bread and butter in those branches. When trying out Godot C# I was a little bit sad not to have some kind of 'dependency injection layer'.

There are two other projects that attempt to support DI in Godot C#, [Chickensoft.AutoInject](https://github.com/chickensoft-games/AutoInject) and [Godot.DependencyInjection](https://github.com/Filip-Drabinski/Godot.DependencyInjection) (unofficial repository, just like mine). However the first one is simply not my taste and the second has not been maintained for a while and I didnt get it to run. As a result I tried implementing basic dependency injection on my own...

While I have experience with C# .NET/ASP.NET, I am new to Godot. So I am open to suggestions for optimizing or expanding the projects featureset.

## Features
- Very simple and minimal to setup
- Full fledged DI using attributes
- Allows adding config files using standard .NET APIs
- Allows adding standard .NET logging
- Uses standard .NET interfaces and implementations under the hood
- Keyed dependency injection support

## Limitations
In Razor components, injected properties are obviously still null and not initialized in a components constructor. A similar, yet differnt limitation exists with Godot C# and this DI implementation. Injected propertiesa are not initialized before a nodes `_EnterTree()`-method is being invoked. However they are inialized before `_Ready()`, so that will probably be fine for the vast majority of use cases.

## Disclaimer
Currently the `IServiceProvider` used in the background is never used to create a service scope. As a result I dont think scoped services will behave as you would expect in other applications. I am also not sure what would qualify as a scope in Godot, so feel free to enlighten me if you do.

## Instructions
### Setup
1. Add the `Godot.CSharp.DependencyInjection` NuGet package to your Godot apps C# project and solution.
2. Create a script (no scene required) with a class that implements the abstract class `StartupNodeBase`.
3. Add the script to the autoload configuration of your Godot app in the Godot editor (top left in the editor: `Project/Project Settings/Globals/Autoload`).

### Use
#### Configurations
To register services, override the method `StartupNodeBase.ConfigureServices(IServiceCollection)` and register your services like you would in standard C# .NET applications.

There are configurable options for dependency injection available. As of right now there is just a small setting about logging and the `ServiceProviderOptions`, but this can easily be extended in the future if required.

```cs
internal partial class StartupNode : StartupNodeBase
{
	protected override void ConfigureDependencies(IServiceCollection services)
	{
		services.AddSingleton<IGameInstanceProvider, GameInstanceProvider>();
		services.AddSingleton<ICheckpointManager, CheckpointManager>();
	}

	protected override void ConfigureOptions(DependencyInjectionOptionsBuilder builder)
	{
		builder.WithEditorLoggingMode(EditorLoggingMode.Debug);
	}

	protected override void OnAfterEnterTree()
	{
		// For access to the _EnterTree() after the initial startup routine has been completed
	}
}
```

#### Injection
To inject services create a property in your node. This property can have any access modifier, but has to have a getter and a setter. Place the `InjectAttribute` over it.

If you want to use keyed injection you can specify a key in the constructor of the attribute.

```cs
public partial class Checkpoint : Area2D
{
	[Inject]
	private ICheckpointManager CheckpointManager { get; set; } = null!;

	[Inject("some-key")]
	private ISomeKeyedService SomeKeyedService { get; set; } = null!;
}
```

#### Accessing `DependencyInjectionOptions`
Accessing the configured `DependencyInjectionOptions` can be achieved using the `IDependencyInjectionOptionsProvider` interface.

```cs
internal partial class SomeNode : Node
{
	[Inject]
	private IDependencyInjectionOptionsProvider OptionsProvider { get; set; } = null!;

	public override void _Ready()
	{
		var options = OptionsProvider.GetOptions();
	}
}
```

### Editor Logging
This type of logging logs using `GD.Print(string)` and `Debug.WriteLine(string)`. It does not log into a file by default. This is meant to be able to comfortably track whether the injection is working as expected. Very useful for me when developing but also useful for anybody else who might wonder whether an error they might occurr is on them or this DI implementation.
