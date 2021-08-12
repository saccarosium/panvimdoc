# panvimdoc

Write documentation in [pandoc markdown](https://pandoc.org/MANUAL.html).
Generate documentation in vimdoc.

# Motivation

Writing documentation is hard.
Writing documentation for vim plugins in vimdoc is an additional hassle.
Making writing vim plugin documentation frictionless is important.

Writing documentation in markdown and converting it to vimdoc can help toward that goal.
This way, plugin authors will have to write documentation just once ( for example, as part of the README of the project ), and the vim documentation can be autogenerated.

Writing vim documentation requires conforming to a few simple rules.
Although `vimdoc` is not a well defined spec, it does have some nice syntax highlighting and features like tags and links when the text file is in `vimdoc` compatible format and when `filetype=help` in vim.
Also, typically, while vim documentation are just plain text files, they are usually formatted well using whitespace.
Preserving these features and characteristics is important.

See [References](#references) for more information.

Writing documentation in Markdown and convert to vimdoc is not a novel idea.
[@mjlbach](https://github.com/mjlbach) has already implemented a neovim treesitter based markdown to vimdoc converter that works fairly well.
This approach is close to ideal. There are no dependencies ( except for the Markdown treesitter parser ). While it appears that the markdown parser may cause crashes, I have not experienced any in my use. It is neovim only but you can use this on github actions even for a vim plugin documentation.

I found two other projects that do something similar, again linked in the references.
As far as I can tell, these projects are all in use and actively maintained and these projects may suit your need.

However, none of these projects use Pandoc.
Pandoc Markdown supports a wide number of features: See <https://pandoc.org/MANUAL.html> for more information.
Most importantly, it supports a range of Markdown formats and flavors.
And, Pandoc has lua filters and a custom output writer that can be configured in lua.
Pandoc filters can extend the capability of Pandoc with minimal lua scripting.
Pandoc filters and the custom output writer in lua are very easy to write and maintain too.

This project aims to write a specification in Pandoc Markdown, and to take advantage of Pandoc filters and the custom output writer capability, to convert a Markdown file to a vim documentation help file.
This project provides a reference implementation of the specification as well.

# Goals:

- Markdown file must be readable when the file is presented as the README on GitHub / GitLab.
- Markdown file converted to HTML using Pandoc must be web friendly and render appropriately (if the user chooses to do so).
- Vim documentation generated must support links and tags.
- Vim documentation generated should be aesthetically pleasing to view, in vim and as a plain text file.
  - This means using columns and spacing appropriately.
- Format of built in Vim documentation is used as guidelines but not as rules.

# Features

- Autogenerate title for vim documentation
- Generate links and tags
- Support markdown syntax for tables
- Support raw vimdoc syntax where ever needed for manual control.
- Support including multiple Markdown files

# Specification

The specification is described in [panvimdoc.md](./doc/panvimdoc.md) along with examples.
The generated output is in [panvimdoc.txt](./doc/panvimdoc.txt).
The reference implementation of the Pandoc lua filter is in [panvimdoc.lua](./scripts/panvimdoc.lua).
See [entrypoint.sh](./entrypoint.sh) for how to use this script, or check the [Usage](#usage) section.

If you would like to contribute to the specification please feel free to comment on this issue: <https://github.com/kdheepak/panvimdoc/issues/1>.

# Usage

```bash
pandoc --lua-filter scripts/include-files.lua -t scripts/panvimdoc.lua ${INPUT} ${OUTPUT}
```

The following are the metadata fields that the custom writer uses:

- `project` (String) _required_: This is typically the plugin name. This is prefixed to all generated tags
- `vimdoctitle` (String) _required_: This is the name of the documentation file that you want to generate
- `vimversion` (String) _optional_: The version vim / neovim that the plugin is targeting. If not present, the version of vim in the available environment is used.
- `toc` (Boolean) _optional_: Whether to generate table of contents or not

Example:

```markdown
---
project: panvimdoc
vimdoctitle: panvimdoc.txt
vimversion: Neovim v0.5.0
toc: true
---
```

## Using Github Actions

Add the following to `./.github/workflows/pandocvim.yml`:

```yaml
name: panvimdoc

on: [push]

jobs:
  custom_test:
    runs-on: ubuntu-latest
    name: pandoc to vimdoc
    steps:
      - uses: actions/checkout@v2
      - name: panvimdoc
        uses: kdheepak/panvimdoc@v1
        with:
          pandoc: INPUT_FILENAME.md
          vimdoc: doc/OUTPUT_FILENAME.txt
      - uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: "Auto generate docs"
          branch: ${{ github.head_ref }}
```

Choose `INPUT_FILENAME` and `OUTPUT_FILENAME` appropriately.

# References

- <https://learnvimscriptthehardway.stevelosh.com/chapters/54.html>
- <https://github.com/nanotee/vimdoc-notes>
- <https://github.com/mjlbach/babelfish.nvim>
- <https://foosoft.net/projects/md2vim/>
- <https://github.com/wincent/docvim>
