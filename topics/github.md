# GitHub

- [GitHub](#github)
  - [Searches](#searches)
  - [Access tokens](#access-tokens)
  - [Coauthorship (multiple authors)](#coauthorship-multiple-authors)
  - [README.md](#readmemd)
    - [Images](#images)
    - [Link references](#link-references)
  - [Releases](#releases)
  - [Pages](#pages)

## Searches

Sample search using multiple topics and a language:

- GUI: `language:rust topic:hacktoberfest topic:virtualization`
- HTTP: https://github.com/search?l=Rust&q=topic%3Ahacktoberfest+topic%3Avirtualization&type=Repositories

## Access tokens

Reference: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

- Account settings
- Developer settings
- Personal access tokens

## Coauthorship (multiple authors)

Github's specification for coauthorship is to add each additional author in the commit message, in the format `Co-authored-by: Name <email>`. This is supported also by Gitlab.

## README.md

### Images

Display an image (WATCH OUT! This won't be rendered from external sites):

```md
![Highlighted block rendering](/readme_images/hightlighted_block_rendering.png?raw=true)
```

Image (raw) path: `https://github.com/64kramsystem/vscode-markdown-code-blocks-asm-syntax-highlighting/blob/master/pizza/test.png?raw=true`

### Link references

- Project root: prefix the path with `/../../` (see [Stack Overflow](https://stackoverflow.com/a/40440270/210029))
  - commit SHAs don't autoreference; it's not clear if there is any way (besides using a rel/abs URL)
  - don't forget that naive paths (`.../$repo/$filename`) won't work, as the file type and branch are required!
- Current path: use the bare filename, without prefixes; the (dynamically generated) prefix `/blob/$current_branch/` will be automatically added

## Releases

Download the latest release of a repo (filtered by a certain file extension):

```sh
curl -sSL https://api.github.com/repos/rust-lang/mdBook/releases/latest | jq --raw-output '.assets[] | .browser_download_url' | grep 'linux-gnu.tar.gz$'
```

## Pages

It's possible to have only one subdomain per account, however, it's possible to have unlimited page sites in the path (e.g. `myaccount.github.io/mypages`):

- create a repository and go to `settings -> pages`
- the site root can be either the project root or `docs/`, but nothing else
