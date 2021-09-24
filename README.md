# klepto
Main repository of klepto toolchain.

This does not contain any actual code (the toolchain itself is split in multiple repositories) but has the user guide and releases.

# How to use klepto
First, you must have in your `PATH` the host Swift toolchain of the same version as klepto. This is necessary to run SwiftPM (it needs host Swift shared libs).

Download the toolchain from the releases tab of this repository and extract it somewhere. Add that to your `PATH` and you're all set.

Like the regular Swift toolchain, everything is done using one command: `klepto`. It is a subset of SwiftPM so you can use klepto very easily by swapping `swift` by `klepto` in your usual Swift commands.

Use `klepto --help` to see all the available commands and their options.

There are a few things to know:
- SwiftPM (`klepto build`) is the only supported way to build Swift code (don't try to use `swiftc` directly)
- romfs, image, homebrew title and author are not implemented yet but will eventually be
- deko3d shaders compilation is not implemented yet
- Tests are not supported
- Switch sysmodules are not supported for now, but may eventually be
- Switch build files are placed in `.build_nx`
- Foundation and Dispatch are not available for now, and may never be
- Image introspection has not been reimplemented (it normally uses `dlopen`), so stack traces have been disabled
- The clang `blocks` extension has been forcefully disabled (due to bad luck in newlib's headers, but who cares)
- You can use the `__SWITCH__` define to tell if the code is being compiled for Switch
    - This applies to both the package manifest (`Package.swift`) and the actual code, allowing you to make hybrid packages
    - You can also use `os(libnx)` if you want but it will emit warnings when compiling with a regular Swift toolchain (does not apply to the manifest)
- The Switch platform counts as Linux so:
    - `os(linux)` is true
    - `__linux__` and `__unix__` are both defined

All features of the language are otherwise available.

# Package manifest differences
Any Swift package won't work on Switch out of the box. There are a few differences in how you build the manifest:

- Executable products have been replaced by nx application products
    - Use `.nxApplication(...)` instead of `.executable(...)`
    - This will make an NRO file in the working directory, along with an ELF file for debugging

If your package is hybrid, don't forget to surround your Switch products by `#if __SWITCH__ (...) #endif`.

# Creating a new package
Creating a new package is done the same way as with the regular Swift toolchain: `klepto package init`. You can use `--type` to select the package type to create.

The types are however different:
- `nx-application`: creates a normal Switch application with the standard libnx hello world in `main.swift`
- `nx-hybrid-application`: creates a PC / Switch hybrid application, which is the same as the above type but with all the Switch specific stuff surrounded by `#if __SWITCH__ (...) #endif` in both the package manifest and the code

The default type is now `nx-application`.

# Building your package
Use `klepto build` to build your package. This accepts the same parameters as the regular Swift build command.

The configuration is `debug` by default, which is unoptimized. Use `-c release` to have an optimized release build (recommended).

# Running your package
If you have one nx application product, you can run it directly by using `klepto run` instead of `klepto build`. Both commands accept the same set of parameters.

Using the run command will first build your application then run it using nxlink directly. As such, the run command accepts all standard nxlink parameters in addition to the build parameters.

For instance, this build the application using the release configuration, then sends the NRO to the Switch at 10.0.0.14 before starting the nxlink server: `klepto run -c release -a 10.0.0.14 -s`.

# Cleaning your package
You can clean build artifacts by using `klepto clean`, which is simply a shortcut to `klepto package clean`.

This is not a standard Swift feature.
