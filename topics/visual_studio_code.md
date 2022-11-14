# Visual Studio Code

- [Visual Studio Code](#visual-studio-code)
  - [Extensions](#extensions)
    - [Building/Packaging/Publishing](#buildingpackagingpublishing)
    - [Debug](#debug)
    - [Get launch configurations](#get-launch-configurations)
  - [Associating Markdown code blocks to grammars](#associating-markdown-code-blocks-to-grammars)
  - [Configuration](#configuration)
  - [C/C++](#cc)
    - [Setting up a project](#setting-up-a-project)
      - [Makefile-only projects](#makefile-only-projects)
    - [CMake](#cmake)
    - [Manually set Includes/Macros](#manually-set-includesmacros)

## Extensions

### Building/Packaging/Publishing

Run from the extension repository.

Build:

```sh
# Install the node modules in the project (under `node_modules`), and transpile; interrupt after compilation.
#
npm install
```

Package (requires NodeJS v14):

```sh
npm install -g vsce

# Package into a vsix file.
#
vsce package
```

Publish; requires organization and token (see https://code.visualstudio.com/api/working-with-extensions/publishing-extension#publishing-extensions).

WATCH OUT!!! vsce seems to associate a given extension to the local path, so when an extension is renamed (or similar), rename the project directory!

```sh
vsce login $organization

vsce publish

vsce unpublish
```

### Debug

Output (an object) to the console, server side:

```js
console.log(">>> " + JSON.stringify(object, null, 4));
```

### Get launch configurations

See https://code.visualstudio.com/api/references/vscode-api#WorkspaceConfiguration:

```js
const launch_config = vscode.workspace.getConfiguration('launch', vscode.workspace.workspaceFolders[0].uri);
const configurations = launch_config.get("configurations");
```

## Associating Markdown code blocks to grammars

See PRs in the [repository](https://github.com/64kramsystem/vscode-fenced-code-block-grammar-injections).

When integrating into an existing project, a new language id entry needs to be added; if an existing one is recycled, the grammar will apply only to the new scope.

## Configuration

Configuring file type:

```json
// In settings.json
{
  // ...
  "[c]": {
    "editor.tabSize": 4,
    "editor.detectIndentation": false
  }
}
```

See https://code.visualstudio.com/docs/editor/variables-reference for variables; some:

- `userHome`
- `workspaceFolder`

## C/C++

### Setting up a project

_READ THE WHOLE CHAPTER, AS THE MAKE TOOLS ARE A CLUSTERF*CK_

The `Makefile tools` extension takes care of the project configuration; make sure to launch `C/C++: Edit Configurations (UI)`, which creates `.vscode/c_cpp_properties.json`.

The error "Makefile entry point not found" can be ignored; it can be worked around by symlinking the Makefile into the workspace root dir (`settings.json#makefilePath` won't fix the error).

#### Makefile-only projects

WATCH OUT!! Configuration errors may not be visible!!

```json
  // Sample configuration for a project that is built via `make all`; becomes the default.
  //
  "makefile.configurations": [
    {
      "name": "Default",
      "makeArgs": [ "all" ],
      // Add if the Makefile is not in the root directory.
      //
      "makeDirectory": "${workspaceFolder}/sdl_port_project/src"
    }
  ]
```

It seems that the extension own build result is the only way to configure defines/includes: settings configured in `settings.json`/`c_cpp_properties.json` (including `compileCommands`) don't have any effect.

In order to test the configuration (some changes may *not* be applied):

- there may be an internal confguration db, so run "Makefile: Clean configure" command
- reset via "Reset the Makefile Tools Extension" command, and close/reopen the given file
- errors can sometimes be detected by running the task `Makefile: Build the current target`, however, it may not display anything even in case of error

### CMake

Make sure to configure the right cmake binary (e.g. if provided via python).

When a project is opened, the extension will perform a build dry run; it's better to run build (make) the project before, since some operations may be performed (e.g. generating files), that the dry run doesn't.

NOT VERIFIED: the option `c_cpp_properties.json#compileCommands` may apply to this build type.

### Manually set Includes/Macros

WATCH OUT! Remember that in Makefile-only projects, this section may not apply.

In order to solve specific include problems, click on the lighbulb, and select the appropriate option (for unclear reasons, this is not always available).

Add include paths:

```json
// In settings.json
"C_Cpp.default.includePath": [
  "/path/to/lib"
],

// In c_cpp_properties.json

"includePath": [
  "${workspaceFolder}/**", // default
  "/usr/include/SDL2",
  "/usr/include/x86_64-linux-gnu/sys"
],

// For unclear reasons, some entries must be manually added to `browse.path`, otherwise, a warning is raised:

"browse": {
  "path": [
    "/usr/include/SDL2"
  ]
},
```

Define macros:

```json
// In settings.json

"C_Cpp.default.defines": [
  "__UNIX__",
]

// or in c_cpp_properties.json

"defines": [
  "THREAD_SYSTEM_DEPENDENT_IMPLEMENTATION"
]
```
