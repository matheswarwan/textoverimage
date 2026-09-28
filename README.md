# textoverimage: text-over-image block for SFMC Content Builder (work in progress)

A custom content block for Salesforce Marketing Cloud (SFMC) Content Builder. The goal is to place rich text on top of an image, optionally in a custom font. This is an unfinished prototype: several parts are marked "WIP" in the UI and some code paths are broken (see Known limitations).

## What it does

The block editor has these sections:

- **Text over image**: a CKEditor 5 rich text editor (loaded from the CKEditor CDN) for the overlay text.
- **Additional Settings / Upload Font**: pick a `.woff`, `.ttf` or `.otf` file. The font is read in the browser and kept as a base64 data URL.
- **Image**: either an external image URL (marked WIP) or an uploaded `.jpg`, `.gif` or `.png`. Uploaded images are also read as base64 data URLs. Filling one input disables the other.
- **Image Position (WIP)**: number fields for width, height and left/right/top/bottom margins (defaults 600, 400, 10, 10, 10, 10).
- A **submit** button that saves the current state.

## How it works

`index.html` creates the Block SDK with the standard `htmlblock` and `stylingblock` tabs.

`saveData()` runs when the editor text changes (500 ms debounce), when a position field changes, or when you click submit. It:

1. Builds a data object: `textOverImage` (editor HTML), `fontBase64`, `imageUrl`, `imageBase64`, and `imagePosition` (`width`, `height`, `margin`, and a fixed `padding` of 5).
2. Saves it with `sdk.setData(data)`.
3. Builds the email HTML with `generateHTMLContent()` and sets it with `sdk.setContent(html)`.

The generated HTML is a `position: relative` wrapper that contains:

- a `<style>` tag with an `@font-face` rule named `customFont`, pointing at the base64 font,
- an absolutely positioned `<div>` centered over the image (`top/left: 50%`, `translate(-50%, -50%)`), with the overlay text and the margins from the form,
- the `<img>` at `width: 100%`, using the uploaded image if there is one, otherwise the external URL.

## Hosting

The block is a static page. It must be served over HTTPS so SFMC can load it in an iframe. `index.php` only includes `index.html`, so it can run on a PHP host (for example Heroku with the PHP buildpack) or on any static web server. jQuery, Bootstrap 3 and CKEditor are loaded from public CDNs. There are no hardcoded hosting URLs for the block itself.

## Register it in SFMC

1. Host the files on an HTTPS URL.
2. In SFMC go to **Setup → Apps → Installed Packages**, create a package (or open an existing one) and add a component of type **Custom Content Block**.
3. Set the endpoint URL to your hosted `index.html` (or the site root).
4. The block then shows up in Content Builder under custom blocks.

## Project structure

- `index.html`: the current block (editor UI, `saveData()`, `generateHTMLContent()`).
- `index-old.html`: an earlier version with a simple form (text, image URL, position) based on SLDS styling. Not used.
- `imageoverlay.html`: an early CKEditor experiment page. Not used.
- `font.html`, `font and image delete.html`: scratch HTML and JS snippets. Not used.
- `TODO`: notes, including an idea to browse Content Builder images through the Content Builder asset query endpoint.
- `The Californication.ttf`: a sample font file, apparently for testing the font upload.
- `blocksdk.js`: vendored copy of the SFMC Block SDK.
- `index.php`: PHP entry point that includes `index.html`.
- `css/`, `svg/`, `js/`: vendored SLDS CSS, an SLDS icon sprite, jQuery and RequireJS. Used by `index-old.html`, not by `index.html`.
- `icon.png`, `dragIcon.png`: block icons.

## Known limitations

- Reopening a saved block is broken. The load code in `sdk.getData()` reads `data.textOverImage.margin.*` (but `textOverImage` is a string) and `data.imagePosition.margin.width` (width is not inside `margin`). This throws an error, so saved values are not restored in the form.
- The margin values are saved without units (for example `10`, not `10px`), so the inline CSS margins have no effect.
- The overlay is always centered. The position fields do not move the text in a meaningful way yet.
- Fonts and uploaded images are embedded as base64 in the email HTML. This makes emails large, and many email clients ignore `@font-face` and absolute positioning, or block data URL images.
- The image `alt` text is hardcoded to "SNOW".
- Browsing images from Content Builder is only an idea in `TODO` and a commented-out function.
- The page mixes Bootstrap 3 (loaded) with Bootstrap 4 class names such as `col-sm` and `d-flex`, so parts of the layout are unstyled.
- Lots of `console.log` output, including full base64 files.
- No tests.

## Ideas

- Fix the load path in `sdk.getData()` so saved blocks reopen correctly.
- Upload the image (and font) to Content Builder and reference it by URL instead of embedding base64.
- Use a table and background-image approach (with a VML fallback for Outlook) for better email client support.
- Remove the unused old pages and vendored files.
