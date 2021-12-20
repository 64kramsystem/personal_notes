# Visual Studio Code

- [Visual Studio Code](#visual-studio-code)
  - [Publishing extensions](#publishing-extensions)
  - [Associating Markdown code blocks to grammars](#associating-markdown-code-blocks-to-grammars)
  - [Configuration](#configuration)
  - [C/C++](#cc)
    - [Setting up a project](#setting-up-a-project)
    - [Define macros (symbols)](#define-macros-symbols)

## Publishing extensions

```sh
# Prerequisite; requires NodeJS v14.
#
npm install -g vsce

# Prerequisite: create organization and token (see https://code.visualstudio.com/api/working-with-extensions/publishing-extension#publishing-extensions).

# Run from the extension repository.

# Package into a vsix file.
#
vsce package

# Publish!
#
vsce login $organization
vsce publish
```

## Associating Markdown code blocks to grammars

See PRs in the [repository](https://github.com/64kramsystem/vscode-fenced-code-block-grammar-injections).

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

Setup the basic options via UI (`C/C++: Edit Configurations (UI)`); if necessary, tweak `.vscode/c_cpp_properties.json`.

Some options can be found by observing `./configure`, and/or via bear (see below).

When a project is opened, the extension will perform a build dry run; it's better to run build (make) the project before, since some operations may be performed (e.g. generating files), that the dry run doesn't.

For desperate cases, an option is to use the `bear` tool, which generates the compile options for each file.

Run `bear make`, then add `"compileCommands": "${workspaceFolder}/compile_commands.json"` to `.vscode/c_cpp_properties.json`.

### Define macros (symbols)

Not clear:

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
