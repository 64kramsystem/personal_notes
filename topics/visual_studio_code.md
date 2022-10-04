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
    - [Multi-root and makefile warning](#multi-root-and-makefile-warning)
    - [Find compile commands](#find-compile-commands)
    - [Includes/Macros](#includesmacros)
      - [Configuration/errors handling](#configurationerrors-handling)

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

The `Makefile tools` extension takes care of the project configuration.

Setup the basic options via UI (`C/C++: Edit Configurations (UI)`), which will create `.vscode/c_cpp_properties.json`, then tweak it, if necessary.

Some options can be found by observing `./configure`, and/or via bear (see below).

When a project is opened, the extension will perform a build dry run; it's better to run build (make) the project before, since some operations may be performed (e.g. generating files), that the dry run doesn't.

### Multi-root and makefile warning

In a multi-root setup, typically, a makefile warning is raised when the project is opened.

If the projects are not actually executed, just touch a phony Makefile in the root directory.

### Find compile commands

Follow [C notebook strategies](c.md#find-compilation-commands); then add `"compileCommands": "${workspaceFolder}/compile_commands.json"` to `.vscode/c_cpp_properties.json`.

### Includes/Macros

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

#### Configuration/errors handling

- if the project is managed by CMake, and CMake Tools doesn't gather the configuration:
  - make sure that the `c_cpp_properties.json` exists
  - make sure to configure the right cmake binary (e.g. if provided via python)
  - generate the compile commands, and extract the defines/includes from there
- in order to solve include problems, just click on the lighbulb, and select the appropriate option
  - for unclear reasons, this is not always available
