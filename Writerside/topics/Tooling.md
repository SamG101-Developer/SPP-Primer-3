# Tooling

<primary-label ref="header-label"/>

<secondary-label ref="doc-complete"/>

## Commands

<secondary-label ref="doc-sect-subj-update"/>

<secondary-label ref="feature-wip"/>

The `spp` tool allows several commands to be run, to create, manage and run S++ projects. The commands are:
- [`init`](#project-initialization) - Create a new S++ project in the current folder. The project structure is created
  with the folders and `main.spp` file.
- [`build`](#project-building) - Compile the current project. The compiler will compile all `.spp` files in the `src`
  folder, and output the binary to the `bin` folder. All `vcs/ext/ffi` dependencies will be linked automatically.
- [`run`](#project-running) - Compile and run the current project. The steps are the same as `build`, but the binary is
  executed immediately after compilation.
- [`clean`](#project-cleaning) - Clean the project. The folder containing the binary is emptied, and the project is
  ready to be built again. The binary folder is not deleted.
- [`update`](#project-vcs-updating) - Update the vcs dependencies of the project. The compiler will check the `vcs`
  folder for newer versions of the dependencies, and will offer to update them individually, or all at once.
- [`version`](#compiler-version) - Display the version of the S++ compiler that is currently installed. This can also be
  used to
  list versions of the `vcs` and `ext` dependencies.
- [`help`](#compiler-help) - Display the help message for the S++ compiler. This message contains a list of all
  commands, and a
  brief description of each command.

## Project Initialization

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

The `spp init` command creates the folders and files in the current directory to create a new S++ project. This will
enable the project to be built and run without structural errors. See the project structure section for more information
on the folders and files created.

If the folder is not empty, a confirmation prompt will be shown. If there is a `main.spp` file in the current directory,
and the overwrite prompt is confirmed, the `main.spp` file not be overwritten, as it assumed that a non-complete project
is being converted into a valid S++ project.

No arguments can be passed to the `init` command. The `spp.toml` project configuration that is generated holds all the
metadata that can be updated.

## Project Building

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-wip"/>

The `spp build` command is used to compile the current project. It will recursively search the `src` folder for all
`.spp` code files. The `vcs` and `ext` folders are then searched for code files too, and added to the module tree.
Finally, the stub files inside the `ffi` modules are also linked, for symbol information.

Once all the code has been compiled into llvm code, the `ffi` libraries are linked into the binary, and the binary is
output to the `bin` folder. The binary is placed in a subfolder of the build type (debug or release), and the
dependencies are placed in the same folder.

If the project is already built, and only a subset of the files have been modified, then only those files are
re-compiled. If part of the code changed is exportable (ie function signatures), then files that directly use these
exported symbols are also recompiled. If only a function body is changed, then only that file is recompiled.

Each stage of the compiler can be parallelized, as the order of files does not matter. However, a stage of the compiler
cannot begin until all modules have completed the previous stage. If a module contains an error, the build process
stops, and the error is displayed.

| Flag              | Description                                          |
|-------------------|------------------------------------------------------|
| `--debug`         | Compiles the project in debug mode (default).        |
| `--release`       | Compiles the project in release mode.                |
| `--clean`         | Cleans the project before building.                  |
| `--executable`    | Compiles the project as an executable (default).     |
| `--shared`        | Compiles the project as a shared library.            |
| `--static`        | Compiles the project as a static library.            |
| `--no-vcs-update` | Prevents the compiler from checking for vcs updates. |

The default command is `spp build --debug --executable --no-vcs-update`.

## Project Running

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-not-impl-yet"/>

The `spp run` command can only be used if the `--executable` flag was used in the `build` step. It runs the built binary
from the `bin` folder. The binary is recompiled if there are any changes to the code, and is run immediately after
compilation. The build mode flag is required, to rebuild/run the correct binary.

| Flag        | Description                                   |
|-------------|-----------------------------------------------|
| `--debug`   | Compiles the project in debug mode (default). |
| `--release` | Compiles the project in release mode.         |
| `--clean`   | Cleans the project before building.           |

The default command is `spp run --debug`.

## Project Cleaning

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-not-impl-yet"/>

The `spp clean` command is used to delete the build mode folder if a flag is passed, or all build mode folders if no
flag is passed. This will force the entire project to be re-built, as the binary is deleted. The `bin` folder is not
deleted, as it is part of the project structure; only emptied.

| Flag        | Description              |
|-------------|--------------------------|
| `--debug`   | Cleans the debug build.  |
| `--release` | Cleans the release build |

## Project VCS Updating

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-wip"/>

The `spp update` command is used to update the vcs dependencies of the project. The compiler will check the `vcs` folder
for all the dependencies. If there is a newer version of the repository, a prompt offering to update the repository (or
all repositories) will be shown. The prompt will also show the current version and the new version of the repository.

An argument can be specified to select a specific repository to update. If no argument is passed, then all repositories
will be checked for updates, using `--update-repo *`.

The `spp build` command will run the `spp update` command each time there is a repository update, unless the
`--no-vcs-update` flag is used. If the user has specified that they don't want to update a repository, the compiler will
subsequently add `--ignore-repo <repo_name>` to the `spp update` call from `spp build`. This will skip checks for the
specified repository.

| Flag                         | Description                                      |
|------------------------------|--------------------------------------------------|
| `--ignore-repo <repo_names>` | Ignores the update for the specified repository. |
| `--update-repo <repo_names>` | Updates the specified repositories (can be `*`). |

## Compiler Version

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-not-impl-yet"/>

The `spp version` command is used to display the version of the S++ compiler that is currently installed. This will
output the version number, and nothing else. The `--libraries` flag can be used to list the versions of the `vcs` and
`ext` libraries. This information is pulled from their respective `spp.toml` files, but is included as a convenience
tool.

| Flag          | Description                                          |
|---------------|------------------------------------------------------|
| `--libraries` | Lists the versions of the `vcs` and `ext` libraries. |

## Compiler Help

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-not-impl-yet"/>

The `spp help` command is used to display a list of all commands and flags that can be passed to the commands.
