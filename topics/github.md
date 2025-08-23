# GitHub

- [GitHub](#github)
  - [Searches](#searches)
  - [Access tokens](#access-tokens)
  - [Coauthorship (multiple authors)](#coauthorship-multiple-authors)
  - [README.md/Docs](#readmemddocs)
    - [Relative link references](#relative-link-references)
      - [Images](#images)
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

## README.md/Docs

Collapsible text:

```md
<details><summary>TITLE</summary>
MARKDOWN_BODY
</details>
```

### Relative link references

There are two base formats: file (README)-relative (`path/to/file`) and root-relative (`/path/to/file`).

- the link decodes to  `$repo_path/  blob/$branch  /$rel_path/path/to/file`
- the image decodes to `$repo_path/  raw/$branch   /$rel_path/path/to/file`

Notes:

- relative paths may not be supported by external sites; `crates.io` supports relative paths
- symlinks are not followed (via interface, or HTTP links), with the exception of the `README` rendering in each directory (direct opening is not followed)
- don't forget that naive paths (`$repo_path/$filename`) won't work, as git/hub information (tree, etc.) is required!
- commit SHAs don't autoreference; it's not clear if there is any way (besides using a rel/abs URL)
- if one wants to manually set the git/hub information, use `/../../$location/path/` (see [Stack Overflow](https://stackoverflow.com/a/40440270/210029))

Relative links needs support in 3rd party services; e.g. `crates.io` supports them, but watch out the details (see [Cargo manifest publication info](rust_tooling.md#cargo-manifest-for-publication)).

#### Images

```md
![Logo](/images/serdine.jpg?raw=true)
```

## Releases

Download the latest release of a repo (filtered by a certain file extension):

```sh
curl -sSL https://api.github.com/repos/rust-lang/mdBook/releases/latest | jq --raw-output '.assets[] | .browser_download_url' | grep 'linux-gnu.tar.gz$'
```

## Pages

It's possible to have only one subdomain per account, however, it's possible to have unlimited page sites in the path (e.g. `myaccount.github.io/mypages`):

- create a repository and go to `settings -> pages`
- the site root can be either the project root or `docs/`, but nothing else
