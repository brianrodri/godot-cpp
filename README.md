# godot-cpp

> **Warning**
>
> This repository's `master` branch is only usable with
> [GDExtension](https://godotengine.org/article/introducing-gd-extensions)
> from Godot's `master` branch.
>
> For users of stable branches, switch to the branch matching your target Godot version:
> - [`4.0`](https://github.com/godotengine/godot-cpp/tree/4.0)
> - [`4.1`](https://github.com/godotengine/godot-cpp/tree/4.1)
>
> Or check out the Git tag matching your Godot version (e.g. `godot-4.1.1-stable`).
>
> For GDNative users (Godot 3.x), switch to the [`3.x`](https://github.com/godotengine/godot-cpp/tree/3.x)
> or the [`3.5`](https://github.com/godotengine/godot-cpp/tree/3.5) branch.

This repository contains the  *C++ bindings* for the [**Godot Engine**](https://github.com/godotengine/godot)'s GDExtensions API.

- [**Versioning**](#versioning)
- [**Compatibility**](#compatibility)
- [**Contributing**](#contributing)
- [**Getting started**](#getting-started)
- [**Included example**](#included-example)

## Versioning

This repositories follows the same branch versioning as the main [Godot Engine
repository](https://github.com/godotengine/godot):

- `master` tracks the current GDExtension development branch for the next Godot
  4.x minor release.
- `3.x` tracks the development of the GDNative plugin for the next 3.x minor
  release.
- Other versioned branches (e.g. `4.0`, `3.5`) track the latest stable release
  in the corresponding branch.

Stable releases are also tagged on this repository:
[**Tags**](https://github.com/godotengine/godot-cpp/tags).

**For any project built against a stable release of Godot, we recommend using
this repository as a Git submodule, checking out the specific tag matching your
Godot version.**

> As the `master` branch of Godot is constantly getting updated, if you are
> using `godot-cpp` against a more current version of Godot, see the instructions
> in the `gdextension` folder to update the relevant files.

## Compatibility

**Warning:** The GDExtension API is brand new in Godot 4.0, and is still
considered in **beta** stage, despite Godot 4.0 itself being released.

This applies to both the GDExtension interface header, the API JSON, and this
first-party `godot-cpp` extension.

Some compatibility breakage is to be expected as GDExtension and `godot-cpp`
get more used, documented, and critical issues get resolved. See the
[Godot issue tracker](https://github.com/godotengine/godot/issues?q=is%3Aissue+is%3Aopen+label%3Atopic%3Agdextension)
and the [godot-cpp issue tracker](https://github.com/godotengine/godot/issues)
for a list of known issues, and be sure to provide feedback on issues and PRs
which affect your use of this extension.

## Contributing

We greatly appreciate help in maintaining and extending this project. If you
wish to help out, ensure you have an account on GitHub and create a "fork" of
this repository. See [Pull request workflow](https://docs.godotengine.org/en/stable/community/contributing/pr_workflow.html)
for instructions.

Please install clang-format and copy the files in `misc/hooks` into `.git/hooks`
so formatting is done before your changes are submitted.

## Getting started

It's a bit similar to what it was for 3.x but also a bit different.
This new approach is much more akin to how core Godot modules are structured.

Compiling this repository generates a static library to be linked with your shared lib,
just like before.

To use the shared lib in your Godot project you'll need a `.gdextension`
file, which replaces what was the `.gdnlib` before.
See [example.gdextension](test/project/example.gdextension) used in the test project:

```ini
[configuration]

entry_symbol = "example_library_init"
compatibility_minimum = 4.1

[libraries]

macos.debug = "res://bin/libgdexample.macos.debug.framework"
macos.release = "res://bin/libgdexample.macos.release.framework"
windows.debug.x86_64 = "res://bin/libgdexample.windows.debug.x86_64.dll"
windows.release.x86_64 = "res://bin/libgdexample.windows.release.x86_64.dll"
linux.debug.x86_64 = "res://bin/libgdexample.linux.debug.x86_64.so"
linux.release.x86_64 = "res://bin/libgdexample.linux.release.x86_64.so"
# Repeat for other architectures to support arm64, rv64, etc.
```

The `entry_symbol` is the name of the function that initializes
your library. It should be similar to following layout:

```cpp
extern "C" {

// Initialization.

GDExtensionBool GDE_EXPORT example_library_init(GDExtensionInterfaceGetProcAddress p_get_proc_address, GDExtensionClassLibraryPtr p_library, GDExtensionInitialization *r_initialization) {
	godot::GDExtensionBinding::InitObject init_obj(p_get_proc_address, p_library, r_initialization);

	init_obj.register_initializer(initialize_example_module);
	init_obj.register_terminator(uninitialize_example_module);
	init_obj.set_minimum_library_initialization_level(MODULE_INITIALIZATION_LEVEL_SCENE);

	return init_obj.init();
}
}
```

The `initialize_example_module()` should register the classes in ClassDB, very like a Godot module would do.

```cpp
using namespace godot;
void initialize_example_module(ModuleInitializationLevel p_level) {
	if (p_level != MODULE_INITIALIZATION_LEVEL_SCENE) {
		return;
	}
	ClassDB::register_class<Example>();
}
```

Any node and resource you register will be available in the corresponding `Create...` dialog. Any class will be available to scripting as well.

## Examples and templates

See the [godot-cpp-template](https://github.com/godotengine/godot-cpp-template) project for a
generic reusable template.

Or checkout the code for the [Summator example](https://github.com/paddy-exe/GDExtensionSummator)
as shown in the [official documentation](https://docs.godotengine.org/en/latest/tutorials/scripting/gdextension/gdextension_cpp_example.html).
