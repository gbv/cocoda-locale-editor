# cocoda-locale-editor
An editor for single file locale files in JSON format, e.g.

```json
{
  "en": {
    "hello": "Hello"
  },
  "de": {
    "hello": "Hallo"
  }
}
```

Currently hosted on GitHub Pages at https://gbv.github.io/cocoda-locale-editor/.

Work in progress.

## Table of Contents
- [Install](#install)
- [Usage](#usage)
  - [Starting from scratch](#starting-from-scratch)
  - [Using an existing file](#using-an-existing-file)
- [Maintainers](#maintainers)
- [Publish](#publish)
- [Contribute](#contribute)
- [License](#license)

## Install
```sh
npm ci
# Run dev server on port 8093
npm run dev
# Build the application (into folder dist/)
npm run build
```

## Usage

### Starting from scratch

First, click the "Reset" button to remove the example keys and languages. Add the languages you want to have in your file (e.g. "en" and "de"). When adding keys (at the bottom of the page), note that the chosen language in the dropdown will be the one where the key will be first created (with an empty string as a value). You can add nested keys by using the dot notation, for example "general.title". Note that you won't be able to add nested keys under an existing key (because it was already created as a string).

When you are finished, click the "Download" button at the top of the page to download the .json file with your locale.

### Using an existing file

There are two options for editing an existing file:
1. Uploading the file from disk.
2. Loading the file from an URL.

The latter will also add a URL parameter to the application so that the file can be loaded automatically. Example: https://gbv.github.io/cocoda-locale-editor/?fromUrl=https%3A%2F%2Fraw.githubusercontent.com%2Fgbv%2Fcocoda%2Fdev%2Fconfig%2Flocale.json

You can add languages with the field above the table, or add keys with the field below the table. When you are finished editing, save the file by clicking the "Download" button at the top. You can then commit the file into your repository.

## Maintainers
- [@stefandesu](https://github.com/stefandesu)

## Publish
Currently, the `master` branch is directly deployed to GitHub Pages.

## Contribute
Please use [GitHub issues](https://github.com/gbv/cocoda-locale-editor/issues) for bug reports, feature requests or questions.

PRs accepted.

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License
MIT © 2018 Verbundzentrale des GBV (VZG)
