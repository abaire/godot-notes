---
title: "SCons integration"
subcategory: "CLion"
---

As of March 2021 [CLion](https://www.jetbrains.com/clion/) does not support [SCons](https://scons.org/) natively.
It is possible to get things more or less working with a few workarounds.

## Loading an SCons-based project in CLion

1. Have SCons generate a compilation database

   This snippet leverages SCon's `compilation_db` tool, registering it as a target called `compiledb` and adding that
   target to the default build action.
   
   ```python
   from SCons import __version__ as scons_raw_version
   scons_ver = env._get_major_minor_revision(scons_raw_version)
   if scons_ver >= (4, 0, 0):
     env.Tool("compilation_db")
     cdb = env.CompilationDatabase()
     env.Alias("compiledb", cdb)
     Default(cdb)
   ```
1. Perform a build (optionally just use the `compiledb` target)
1. Open the generated `compile_commands.json` with CLion.
    1. Select `Open as project`
1. If the compilation database changes (e.g., due to adding/removing files) it can be reloaded via
   `Tools > Compilation database > Reload compilation database project`

## Building from within CLion

`scons` can be invoked from CLion using [Custom Build Targets](https://www.jetbrains.com/help/clion/custom-build-targets.html)

1. Preferences > Build, Execution, Deployment > Custom Build Targets
1. Use the `+` button to add a new target
1. Set the Toolchain to default (or a custom toolchain)
1. Edit the Build command

    1. Add a new External Tool (note that these are not the same as CLion's external tools in Preferences)

        1. Set the path to `scons` and appropriate build options

## Running from within CLion

Once the Custom build target is set up, a `Custom build application` run configuration can be set up to build and
launch. This is particularly useful when debugging Godot itself, where a test project can be executed directly using
Godot's `-e` option to provide the project's `project.godot` file.
