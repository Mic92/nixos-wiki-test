---
title: NixPkgs Standard Environment
permalink: /NixPkgs_Standard_Environment/
---

Overview
--------

The purpose of the "standard environment", known as `stdenv` in the NixPkgs codebase, is to set up an environment in which to run a Nix package's builder. The Nix installer will call the builder, providing it a completely empty environment. The standard environment adds onto this empty environment by defining a framework around a typical build process. The intention is to help package maintainers by moving code for packages with similar builders into a single place.

The standard environment is defined in the [`stdenv/generic/default.nix`](https://github.com/NixOS/nixpkgs/blob/544475cb45d3dc38ff91f2f7cd28fd826d00a1a0/pkgs/stdenv/generic/default.nix) file and implemented in the [`stdenv/generic/setup.sh`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh) file. The `setup.sh` script contains the definition of the standard environment's build framework, which consists of shell functions for the build phases and for library functions.

Standard Environment
--------------------

The standard environment is implemented in two parts: a Nix expression and a script to set up and run the package's builder.

### Nix Expression

The [standard environment's Nix expression](https://github.com/NixOS/nixpkgs/blob/master/pkgs/stdenv/generic/default.nix) provides a function, named `mkDerivation`, which accepts attributes of packages. Effectively, the `mkDerivation` function enables package maintainers to note only a package's deviation from a normal package which uses the standard environment's build framework. Attributes of a package go into the `mkDerivation` function and a `derivation` object is returned. A package's derivation object contains all information required to build that package, including its package dependencies, its builder program and options, and the builder script.

Most package variation lies in its build script, so it should be well documented and well organized. Let's look at how to organize a package's build variations.

### Default Builder

To be useable by every package, the standard environment must be able to build every package. Many build tools exist, so the standard environment delegates this responsibility to each package by calling the `genericBuild` shell function, which is expected to be implemented by the package.

The standard environment provides a default implementation of the `genericBuild` function in the `setup.sh` script, which is called by the \[<https://github.com/NixOS/nixpkgs/blob/master/pkgs/stdenv/generic/default-builder.sh>"&gt;`default-builder.sh`\] shell script.

If a package uses AutoConf and Make as build tools, the default implementation of the `genericBuild` function will work great, as it was built around these tools. It provides a layer of abstraction around the build process used by AutoConf and Make, in the form of several shell functions, called "phases". Each phase corresponds to a step of the build process used by these tools, such as configure, make, and install. Having defined these phases, the `genericBuild` function sequentially executes them to build the packaged application.

#### Generic Build

[`genericBuild()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L815)

-   Indicate the build is beginning by showing a message to the user and starting a new level of nesting by using the `header` function.
-   If the `buildCommand` environment variable has a value, execute the custom build command it specifies and exit the build.
-   Set the `$phases` environment variable to `$prePhases unpackPhase patchPhase $preConfigurePhases configurePhase $preBuildPhases buildPhase checkPhase $preInstallPhases installPhase fixupPhase installCheckPhase $preDistPhases distPhase $postPhases`, unless it already has a value.
-   Execute each phase
    -   Skip the corresponding phase if any of the following environment variables are appropriately set or not set: `$dontBuild`, `$doCheck`, `$dontInstall`, `$dontFixup`, `$doInstallCheck`, `$doDist`
    -   Indicate a phase's beginning by calling the `showPhaseHeader` function.
    -   Log the environment variables for the current phase by calling the `dumpVars` function. This information is useful when debugging a failed build.
    -   Enter the directory specified by the `$sourceRoot` environment variable after the `unpackPhase` function completes.
    -   Leave the current phase's nesting level by calling the `stopNest` function.

### Build Phases

[`unpackPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L463)

-   Run the `preUnpack` hook.
-   Unpack the files specified by the `$srcs` environment variable by using the `unpackFile` function. If it has no value, the file specified by the `$src` environment variable is used.
-   Detect and store the name of the directory produced by the unpacking in the `sourceRoot` environment variable. The build fails if zero or more than one directory is produced.
-   Run the `postUnpack` hook.

[`patchPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L530)

-   Run the `prePatch` hook.
-   Uncompress, where appropriate, and apply each patch specified in the `patches` environment variable by using the `patch` program. Options can be passed to the `patch` program by storing them in the `patchFlags` environment variable.
-   Run the `postPatch` hook.

[`configurePhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L560)

-   Run the `preConfigure` hook.
-   Run the `fixLibTool` function on any file named "ltmain.sh", unless the `$dontFixLibtool` environment variable has a value.
-   Add default flags to the configure program.
    -   Prepend `$prefixKey=$prefix $configureFlags`, unless the `$dontAddPrefix` environment variable has a value. The `$prefixKey` default value is `-prefix=`.
    -   Prepend `--disable-dependency-tracking $configureFlags`, unless the `$dontAddDisableDepTrack` environment variable has a value, or the `$configureScript` file does not contain the word "dependency-tracking".
    -   Prepend `--disable-static $configureFlags`, unless the `$dontDisableStatic` environment variable has a value, or the `$configureScript` file does not contain the word "enable-static".
-   Configure the software by executing the program specified by the `$configureScript` environment variable, using `./configure` by default, with the options specified by the `$configureFlagsArray` environment variable.
-   Run the `postConfigure` hook.

[`buildPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L603)

-   Run the `preBuild` hook.
-   Run the `make` program on the `makefile` file, using the options specified by the `$makeFlags`, `$makeFlagsArray`, `$buildFlags`, and `$buildFlagsArray` environment variables. If the file doesn't exist, the file specified by the `$makefile` environment variable is used. If no file is found, the build fails.
-   Run the `postBuild` hook.

[`checkPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L621)

-   Run the `preCheck` hook.
-   Run the `make` program on the `makefile` file, using the options specified by the `$makeFlags`, `$makeFlagsArray`, `$checkFlags`, `$checkFlagsArray`, and `$checkTarget` environment variables. The `$checkFlags` default value is `VERBOSE=y` and the `$checkTarget` default value is `check`.
-   Run the `postCheck` hook.

[`installPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L669)

-   Run the `preInstall` hook.
-   Make the build's output directory, which is specified by the `$prefix` environment variable.
-   Run the `make` program on the `makefile` file, using the options specified by the `$installTargets`, `$makeFlags`, `$makeFlagsArray`, `$installFlags`, and `$installFlagsArray` environment variables. The `$installTargets` default value is `install`.
-   Run the `preInstall` hook.

[`fixupPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L687)

-   Run the `preFixup` hook.
-   Share each directory specified by the `$forceShare` environment variable by moving them to the `share` directory and then placing links in the root of the build directory. The default value of `$forceShare` is `man doc info`.
-   Compress all files in the `share/man` directory, unless the `$dontGzipMan` environment variable has a value.
-   Strip all executable files of debug information in the directories specified by the `$stripDebugList` environment variable. This list will always include the `lib`, `lib32,` `lib64`, `libexec`, `bin`, and `sbin` directories. The files will be stripped by calling the `stripDirs` function with the `-S` option, in addition to any options specified by the `$stripDebugFlags` environment variable. This operation will be skipped if the `$dontStrip` environment variable has a value.
-   Strip all executable files of <em>all</em> information in the directories specified by the `$stripAllList` environment variable. The files will be stripped by calling the `stripDirs` function with the `-s` option, in addition to any options specified by the `$stripAllFlags` environment variable. This operation will be skipped if the `$dontStrip` environment variable has a value.
-   Run the `patchELF` function on the build directory, unless the `$havePatchELF` environment variable is not 1 and the `$dontPatchELF` environment variable has a value.
-   Run the `patchShebangs` function on the build directory, unless the `$dontPatchShebangs` environment variable has a value.
-   Save the value of the `$propagatedBuildInputs` environment variable to the `$out/nix-support/propagated-build-inputs` file.
-   Save the value of the `$propagatedNativeBuildInput` environment variable to the `$out/nix-support/propagated-native-build-inputs` file.
-   Save the value of the `$propagatedUserEnvPkgs` environment variable to the `$out/nix-support/propagated-user-env-packages` file.
-   Save the value of the `$setupHook` environment variable to the `$out/nix-support/setup-hook` file. This operation uses the `substituteAll` function, which replaces all lowercase environment variables with their values in the file.
-   Run the `postFixup` hook.

[`installCheckPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L768)

-   Run the `preInstallCheck` hook.
-   Run the `make` program on the `makefile` file, using the options specified by the `$makeFlags`, `$makeFlagsArray`, `$installCheckFlags`, `$installCheckFlagsArray`, and `$installCheckTarget` environment variables. The `$installCheckTarget` default value is `installcheck`.
-   Run the `postInstallCheck` hook.

[`distPhase()`](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/generic/setup.sh#L781)

-   Run the `preDist` hook.
-   Run the `make` program on the `makefile` file, using the options specified by the `$distFlags`, `$distFlagsArray`, and `$distTarget` environment variables. The `$distTarget` default value is `dist`.
-   Copy all files specified by the `$tarballs` environment variable to the `$out/tarballs` directory. The `$tarballs` default value is `*.tar.gz`.
-   Run the `postDist` hook.

### Hooks

`preHook()` - Arbitrary code to execute before the "standard environment" setup script executes. This is executed after the initial `PATH` definition and before the `addInputsHook` hook is called.

Examples of use: [All's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L14), [Darwin's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L20), [FreeBSD's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L29), \[<https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L42> OpenBSD's prehook, [NetBSD's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L59), [Cygwin's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/native/default.nix#L72), [Nix's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/nix/default.nix#L6), [Linux's prehook](https://github.com/NixOS/nixpkgs/blob/bee4c41e13a9276db4f4a582234c670d74edfa1a/pkgs/stdenv/linux/default.nix#L25)

`postHook()` - Arbitrary code to execute after the "standard environment" setup script finishes loading.

`userHook()` - Arbitrary code to execute after the "standard environment" setup script finishes loading. This is executed after the `postHook` hook is called.

`addInputsHook()` - Arbitrary code to execute before executing the setup scripts defined in the packages specified by the `$buildInputs`, `$propagatedBuildInputs`, `$nativeBuildInputs`, and `$propagatedNativeBuildInputs` environment variables.

### Environment Variables

-   `PATH_DELIMITER=':'`
-   `NIX_GCC=@gcc@`
-   `PATH=@initialPath@:@gcc@`
-   `SHELL=@shell@`
-   `shell=@shell@`
-   `NIX_LDFLAGS="-rpath $out/lib $NIX_LDFLAGS"`, unless `$NIX_NO_SELF_RPATH` == 1
-   `NIX_LDFLAGS="-rpath $out/lib64 $NIX_LDFLAGS"`, unless the `$NIX_LIB64_IN_SELF_RPATH` environment variable has a value.
-   `NIX_LDFLAGS="-rpath $out/lib32 $NIX_LDFLAGS"`, unless the `$NIX_LIB32_IN_SELF_RPATH` environment variable exists.
-   `TZ=UTC`
-   `prefix="$out"`, overridable, unless `$useTempPrefix` == 1, then prefix="$NIX<em>BUILD</em>TOP/tmp_prefix"
-   `PATH=$_PATH${_PATH:+:}$PATH`
-   `NIX_INDENT_MAKE=1`
-   `NIX_BUILD_CORES=(autoCalculated)`, overridable
-   `nestingLevel=0`

### Library Functions

`runHook(function or fileName or envVarName)` - Run the named hook, either by calling the function with that name or by evaluating the variable with that name.

**`exitHandler()`** - Defines commands to execute when the build shell exits.

-   Clear nesting level
-   Save build performance timings to the `$NIX_BUILD_TOP/.times` file
-   Run the `failureHook` hook if the build failed
    -   If the `$succeedOnFailure` value is true, log the failure to the `$out/nix-support/failed` failure and change failure to success
-   Run the `exitHook` if the build did not fail

**`addToSearchPathWithCustomDelimiter(String delimiter, String envVarName, String path)`** - Convenience function. Prepend the specified path to the specified environment variable, followed by the specified delimiter.

**`addToSearchPath(String envVarName, String path)`** -Convenience function. Prepend the specified path to the specified environment variable, followed by the default delimiter, ":". The default delimiter can be changed by changing the value of the `PATH_DELIMITER` environment variable.

**`ensureDir(List<String> paths)`** - Ensure that the specified paths exist.

**`installBin(List<String> paths)`** - Copy the specified paths to `$out/bin`.

**`findInputs(String pkg, String var, String propogatedBuildInputs)`** - Recursively find all build inputs. Executes the specified package's `nix-support/setup-hook` file and the `setup-hook` file of all packages specified in its `nix-support/$propagatedBuildInputsFile` file, if it exists. The expected behavior of the `setup-hook` file is not defined. This seems to be used primarily with `crossPkgs` and `nativePkgs`.

**`addToNativeEnv(String pkg)`** - Add the `/bin` directory of the specified package to the `_PATH` environment variable, then executes each script defined in the `envHooks` environment variable with the specified package as an argument.

**`addToCrossEnv(String pkg)`** - Add the `/bin` directory of the specified package to the `_PATH` environment variable, then executes each script defined in the `crossEnvHook` environment variable with the specified package as an argument.

**`stripDirs(List<String> dirs, String stripFlags)`** - Execute the `strip` program with the specified flags on all files in the specified directories, recursively, in the root directory, specified by the `$prefix` environment variable. In Unix systems, the `strip` program is used to reduce an executable file's size by removing debug symbols and/or symbol tables. [Examples of `strip`](http://www.thegeekstuff.com/2012/09/strip-command-examples/)

**`substitute(String input, String output, String operation, String pattern, String replacement)`**

<em>Operations</em>:

-   <em>`--replace`</em>: Copy the specified input file to the specified output file while modifying its contents to replace the specified symbol with the specified value.
-   <em>`--subst-var`</em>: Copy the specified input file to the specified output file while modifying its contents to replace the specified environment variable with its value.
-   <em>`--subst-var-by`</em>: Copy the specified input file to the specified output file while modifying its contents to replace the specified environment variable with the specified value.

**`substituteInPlace(String file, String operation, String pattern, String replacement)`**

<em>Operations</em>:

-   <em>`--replace`</em>: Modify the specified file by replacing the specified symbol with the specified value.
-   <em>`--subst-var`</em>: Modify the specified file by replacing the specified environment variable with its value.
-   <em>`--subst-var-by`</em>: Modify the specified file by replacing the specified environment variable with the specified value.

**`substituteAll(String input, String output)`** - Copy the specified input file to the specified output file while modifying its contents to replace all lowercase environment variables with their values.

**`substituteAllInPlace(String input, String output)`** - Modify the specified file by replacing all lowercase environment variables with their values.

**`startNest()`** - Increment the `nestingLevel`.

**`stopNest()`** - Decrement the `nestingLevel`.

**`header(String msg)`** - Print the specified `msg` after starting a new nesting level.

**`closeNest()`** - Stop every nesting level. Used when abnormally exiting a build.

**`dumpVars()`** - Log all environment variables to the `$NIX_BUILD_TOP/env-vars` file, unless the `$noDumpEnvVars` value is 1. This environment variable can be controlled from Nix's command line programs by using the `-K` option.

**`stripHash(String path)`** - Strips the path of all parent directories and the base file name's hash, if it has one.

**`unpackFile(String filePath)`** - Unpacks the specified file by choosing and using the appropriate unpacking program with default options. If the file's extension is not "tar" or "zip", this function simply copies the specified file to the build directory. If there is no specified file, this function will run the `unpackCmd` hook, which is expected to place the software's source code in the build directory.

**`fixLibtool(String filePath)`** - Modifies the specified file by running the `sed` program with the `'s^eval sys_lib_.*search_path=.*^^'` regular expression as an operand.

**`patchELF()`** - Execute the `patchelf` program on all executable files in the build directory, or have an ".so\*" extension, to removes unused paths from the file's "RPATH".

**`patchShebangs(String dir)`** - Modify all executable files in the specified directory by replacing the `#!`-prefixed path on the first line with the equivalent path in the Nix store. The program's path in the Nix store is stored in an environment variable, whose name is the program's name. These environment variables are defined as a package's dependent packages are added to the environment.

**`showPhaseHeader(String phaseName)`** - Show the current phase to the user and start a new level of nesting by passing the specified phase to the `header` function.

### Following a Nix Package Build

When is the default builder called when installing a package? Let's follow the build process.

When we execute the `nix-env -i myPackage` command, the Nix package installer does many things, such as parsing Nix expressions to build a derivation for the package. We will skip most of the steps and look only at how Nix starts a package builder.

In this context, Nix has distilled the entire package into a single "derivation" object. This [derivation object](https://github.com/NixOS/nix/blob/7cf539c728b8f2a4478c0384d3174842e1e0cced/src/libstore/derivations.hh#L43) contains every dependency of the software, most notably the application's source files and dependent packages, its builder application and arguments, and the initial state of environment variables.

The Nix package installer uses this derivation object as the sole input to the build process. Nix starts by [creating an empty shell](https://github.com/NixOS/nix/blob/a478e8a7bb8c24da0ac91b7100bd0e422035c62f/src/libstore/build.cc#L1621) and [setting the environment's initial state](https://github.com/NixOS/nix/blob/a478e8a7bb8c24da0ac91b7100bd0e422035c62f/src/libstore/build.cc#L1657). Then, with the build environment prepared, Nix [executes the package's build script](https://github.com/NixOS/nix/blob/a478e8a7bb8c24da0ac91b7100bd0e422035c62f/src/libstore/build.cc#L2142), defined by the "builder" attribute of the derivation.

The standard environment provides a default build process, which is an abstraction layer onto , consisting of several shell functions, called phases, which correspond to each step in the AutoConf & Make build process. These phases can be customized to support variations in an application's unique build process by adding this custom shell code in shell functions called "hooks". Each phase in the standard environment's build process has a "pre" hook and a "post" hook, which allow a package to customize the phase by defining code to execute before and after each phase. Alternatively, if an application's build process is very unusual, a package can completely override a phase by redefining the phase's function.