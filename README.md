# Editable Layout for Directus

A table layout that allows you to edit item fields directly inline.

![](https://raw.githubusercontent.com/formfcw/directus-extension-editable-layout/main/docs/preview-240611.png)

## Installation & Setup

-   [Official Guide](https://docs.directus.io/extensions/installing-extensions.html)
-   [NPM Package](https://www.npmjs.com/package/directus-extension-editable-layout)

Once installed, go to your collection page, on the `Layout Options` tab in the sidebar, select `Editable` from the `Layout` drop-down list.

### Layout Options

`Save`: If the `Automatic` checkbox is `checked`, your changes will be saved automatically. If it is `unchecked` you will have to manually press the Save button that appears on the top right corner (or use the keyboard shortcut `[cmd] + [s]`) to save your changes.

`Spacing`: Change the spacing of the table rows as you would in the default table layout.

## Usage

Click a cell within the collection table and press `[Enter]` or `Double-click` to enter the field and make your change. When you exit the cell by clicking outside it or pressing `[Escape]`, the field value is updated if you have autosave enabled. Otherwise, the cell appears as edited. You can use the `[Tab]` and `arrow keys` to navigate through the cells. You can add fields/columns using the `+`-button in the upper right corner of the collection table, just like in the default table layout.

## Supported Interfaces

All interfaces except relational ones are supported — non-inline/non-trivial interfaces open in a popup.

**Relational Interfaces:** Coming soon …

## Contribution

Contributions are welcome. Make sure to open an issue for bugs or start a discussion for feature requests, before you start writing code!

## License

This project is licensed under [GPL-3.0](./LICENSE-GPL-3.0) and is based on [Spreadsheet Layout](https://github.com/directus-labs/extensions/tree/main/packages/spreadsheet-layout), which I developed for Directus Labs and which is licensed under [MIT](./LICENSE-MIT).
