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
    - [Find compile commands](#find-compile-commands)
    - [Includes/Macros](#includesmacros)

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

## C/C++

### Setting up a project

The `Makefile tools` extension takes care of the project configuration.

Setup the basic options via UI (`C/C++: Edit Configurations (UI)`), which will create `.vscode/c_cpp_properties.json`, then tweak it, if necessary.

Some options can be found by observing `./configure`, and/or via bear (see below).

When a project is opened, the extension will perform a build dry run; it's better to run build (make) the project before, since some operations may be performed (e.g. generating files), that the dry run doesn't.

### Find compile commands

CMake can generate the compile commands: `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1`, which outputs to `compile_commands.json`.

For desperate cases, an option is to use the `bear` tool, which generates the compile options for each file.

Run `bear make`, then add `"compileCommands": "${workspaceFolder}/compile_commands.json"` to `.vscode/c_cpp_properties.json`.

### Includes/Macros

Add include paths; for unclear reasons, some defines raise an error, even if go to definition works.

```json
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

Define macros (not clear):

```json
// In settings.json

"C_Cpp.default.defines": [
  "__UNIX__",
]

// or in c_cpp_properties.json (in a configuration)?

"defines": [
    "THREAD_SYSTEM_DEPENDENT_IMPLEMENTATION"
]
```
