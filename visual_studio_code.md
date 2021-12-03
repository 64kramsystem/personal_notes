# Visual Studio Code

- [Visual Studio Code](#visual-studio-code)
  - [Publishing extensions](#publishing-extensions)
  - [Associating Markdown code blocks to grammars](#associating-markdown-code-blocks-to-grammars)
  - [C/C++](#cc)
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

## C/C++

### Define macros (symbols)

In settings:

```json
"C_Cpp.default.defines": [
  "__UNIX__",
]
```